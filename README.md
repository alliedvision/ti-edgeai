# ti-edgeai
This repository contains a Texas Instruments EdgeAI demo with an Alvium USB camera from Allied Vision.   
The demo code shows how to quickly prototype new system designs with cameras for industrial applications and covers:

* Object detection on a scene with multiple objects
* Semantic segmentation model 


# Prerequisites
To run this demo application, you need:

Hardware:

* [EdgeAI Starter Kit EVM](https://software-dl.ti.com/jacinto7/esd/edgeai-devkit)
* [Camera Module](https://www.alliedvision.com/en/alvium-camera-kit/1800-U-TI-Edge-AI)


Software:
* [EdgeAI SDK](https://software-dl.ti.com/jacinto7/esd/edgeai-devkit)
* [Vimba SDK for ARM64](https://www.alliedvision.com/en/products/vimba-sdk/#c1497)



### 1. Select your Camera Starter Kit from Allied Vision
Allied Vision offers two Camera Starter Kits for TI Edge AI processors from Texas Instruments, which include everything needed to make a prototype. The picture below shows different components in the Starter Kit.

<img src="https://cdn.alliedvision.com/fileadmin/content/images/landing_pages/Alvium_Camera_Kit_TI/Camera_Kit_TI_Edge_AI_Image.png" width="700">

The Camera Kit contains:
* **Alvium 1800 U-500c (color)**: 5 Megapixel USB 3.0 camera with S-Mount and closed housing, equipped with ON Semi's Rolling Shutter AR0521 CMOS sensor
* Allied Vision S-mount lens (8mm focal length)
* Camera tripod adapter
* USB Cable cables (1m length)


Or
	
* **Alvium 1800 U-158m (mono)**: 1,58 Megapixel USB 3.0 camera with S-Mount and closed housing, equipped with SONY IMX 273 Global Shutter CMOS sensor
* Allied Vision S-mount lens (12mm focal length)
* Camera tripod adapter
* USB Cable cables (1m length)


### 2. Install Vimba
Vimba for Linux is a comprehensive SDK for all Allied Vision cameras with GigE and USB interface. You can download the ARM64 SDK <a href="https://www.alliedvision.com/en/products/vimba-sdk/#c1497">from Allied Vision's website</a>.  You can follow the instructions in the Allied Vision documentation to install the SDK. The result should look similar to below.

<img src="https://cdn.alliedvision.com/fileadmin/content/images/landing_pages/Alvium_Camera_Kit_TI/screenshot_starterkit_Vimba-install.PNG" width="700">

Please restart the system before proceeding with the next steps. The Vimba SDK has many pre-compiled examples in C and Python that you can use to test the camera and different configuration options. The C or Python APIs can be used to configure the camera properties such as gain, exposure, image size and also to capture the images as shown in the next step.

# Demo code

Please connect the camera module to the USB3.0 port of your host system.

As a first step, make sure your system is working properly. The Python code below can be used to configure the camera for a given image size, exposure, gain and then save the image to a JPG file.

```c
from time import time
import cv2

# Import vimba libraries
from vimba import *

print('Starting Vimba')

#List the interface features
print('\nListing all the features of the interfaces\n')

with Vimba.get_instance() as vimba:
  ifs = vimba.get_all_interfaces()
  print('number of interfaces:', len(ifs))

  #Capture an image
  print('\nCapture and print an image\n')

  av_cams = vimba.get_all_cameras()
  print('number of cameras:', len(av_cams))
  #av_cam = av_cams[0]
  #av_cam.Width.set(64)
  #av_cam.Height.set(64)
  with av_cams[0] as av_cam:
    # Acquire 10 frame with a custom timeout (default is 2000ms) per frame acquisition.
    # Max_width: 2592, Max_Height: 1944
    # Multiple of 8
    av_cam.Width.set(2592)
    av_cam.Height.set(1944)
    
    # Set gain to the max, 24dB
    print('cur gain is is',  av_cam.Gain.get())
    av_cam.Gain.set(24)     
    print('new gain is',  av_cam.Gain.get()) 

    #inc=exposure_time.get_increment()
    av_cam.ExposureTime.set(40000) # 500msec
    exposure_time = av_cam.ExposureTime
    print('Default exposure time is', exposure_time)
    time=exposure_time.get()
    print('Default exposure time is (get)', time)

    for frame in av_cam.get_frame_generator(limit=1):    
      print('Got {}'.format(frame), flush=True)
      #frame = av_cam.get_frame()
      #frame.convert_pixel_format(PixelFormat.Mono8)
      frame.convert_pixel_format(PixelFormat.Bgr8)
      cv2.imwrite('av_frame_test.jpg',frame.as_opencv_image())

```
Once you have verified the code is working, replace the input image portion in the EdgeAI demo framework. 
EdgeAI SDK has a modular architecture for input data, inferencing and output data. 
You can use that as-is with only replacing how the input is captured.

There are two places where you need to make code changes:

First, set the pipeline for the video flow. The following code snippet shows the changes where the 
standard USB camera at /dev/video2 is replaced with the Allied Vision camera and the APIs provided by Vimba.

```c
    def setup_pipeline(self, gst_src_cmd, gst_sink_cmd, disp_size):
        """
        Create the Gstreamer pipelines for camera and display.
        Starts the threads for running the pipeline and inference algorithm.

        Args:
            gst_src_cmd (string): A Gstreamer pipeline which starts with either
                file source or camera element and ends with an appsrc element
            gst_sink_cmd (string): A Gstreamer pipeline which starts with an
                appsink element and connects to some display element or filesink
                element
            disp_size (tuple): Width and Height of the display
        """
  
              # Use Vimba camera
        print("Initialize the Allied Vision Camera")
        with Vimba.get_instance() as vimba:
          ifs = vimba.get_all_interfaces()
          print('number of interfaces:', len(ifs))
          
          self.av_cams = vimba.get_all_cameras()
          print('number of cameras:', len(self.av_cams))
          #av_cam = vimba.get_all_cameras()[0]
          #av_cam = self.av_cams[0]
          with self.av_cams[0] as self.av_cam:
            #Max: 2592x1944
            #1280x720
            self.av_cam.Width.set(1280)
            self.av_cam.Height.set(720)            
          #Set gain to the max, 24dB
            print('cur gain is is',  self.av_cam.Gain.get())
            self.av_cam.Gain.set(12)     
            print('new gain is',  self.av_cam.Gain.get())             
            self.av_cam.ExposureTime.set(40000) # 500msec
            exposure_time = self.av_cam.ExposureTime
            print('Default exposure time is', exposure_time)
            time=exposure_time.get()
            print('Default exposure time is (get)', time)
        
            print("Complete: Initialized the Allied Vision Camera.. test by reading one frame")
 
            for frame in self.av_cam.get_frame_generator(limit=1):    
              print('Got {}'.format(frame), flush=True)
              frame.convert_pixel_format(PixelFormat.Bgr8)        
   ```

The second change is in the run_pipeline module where the standard camera read is replaced with reading the frame from the camera as shown below.

```c
    def run_pipeline(self):
        """
        Callback function to run the capture display pipeline in a separate
        thread. This gets the buffer from Gstreamer appsink via OpenCV camera,
        calls the postprocess hook and sends the output to Gstreamer appsrc via
        OpenCV display
        """
        print("[UTILS] Starting pipeline thread")
        while (not utils.stop):
            start_frame = time()

            # Srik
            #status, self.frame = self.camera.read()
            
            # Use an image
            
            #print('reading the next image from Allied VIsion camera')
            #self.frame = cv2.imread("save1.jpg") #From Allied, does all 3 but display is bad
            #self.frame = cv2.imread("save2.jpg") # Works
                    
            with Vimba.get_instance() as vimba:
                av_cams = vimba.get_all_cameras()
                with av_cams[0] as av_cam:
                    for frame in av_cam.get_frame_generator(limit=1):    
                        #print('Got {}'.format(frame), flush=True)
                        frame.convert_pixel_format(PixelFormat.Bgr8)
                        self.frame = frame.as_opencv_image()
                        cv2.imwrite('av_frame_debug2.jpg',self.frame)
 
            #self.frame = cv2.imread("av_frame_debug2.jpg")    
   ```

Once you make the changes, you can run the demo just like the standard demos in the edgeAI SDK. Below commands are couple of examples for the object detection.

```c
./object_detection.py -m ../models/detection/TFL-OD-203-ssdLite-mobDet-EdgeTPU-coco-320x320 -o test_det.avi
   ```
Below is a sample output from the object detection demo on a scene with multiple objects being detected along with the bounding boxes.

![Object detection demo](https://cdn.alliedvision.com/fileadmin/content/images/landing_pages/Alvium_Camera_Kit_TI/screenshot_starterkit-TI-edge-analysis.png)


```c
./semantic_segmentation.py -m ../models/segmentation/TVM-SS-565-fpnlite-aspp-mobv2-ade20k32-512x512 -o test_semantic.avi
   ```

Below is a sample output from the semantic segmentation model running on the same scene.
![Semantic segmentation model](https://cdn.alliedvision.com/fileadmin/content/images/landing_pages/Alvium_Camera_Kit_TI/screenshot_starterkit_semantic-segmentation.png)

That is about it.
There are only two places where the code needs to be changed if you are bringing a new camera to be able to run different demos on the edgeAI platform. With extensive model selection and modular software architecture, it is indeed very easy for developers to make such changes quickly and prototype new system designs with different camera modules needed for industrial applications.


