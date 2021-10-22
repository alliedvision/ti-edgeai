# ti-edgeai
This repository contains a Texas Instruments EdgeAI demo with an Alvium USB camera from Allied Vision.   
The demo code shows how to quickly prototype new system designs with cameras for industrial applications and covers:

* Object detection on a scene with multiple objects
* Semantic segmentation model 


# Prerequisites
To run this demo application, you need:

Hardware:

* [EdgeAI Starter Kit EVM](https://www.ti.com/tool/SK-TDA4VM)
* [Camera Kit](https://www.alliedvision.com/en/alvium-camera-kit-1800-u-ti-edge-ai)


Software:
* [EdgeAI SDK](https://www.ti.com/tool/PROCESSOR-SDK-J721E)
* [Vimba SDK for ARM64](https://www.alliedvision.com/en/products/vimba-sdk/#c1497)



### 1. Select your Camera Kit from Allied Vision
Allied Vision offers two Camera Kits for TI Edge AI processors from Texas Instruments, which include everything needed to make a prototype. The picture below shows different components in the Camera Kit.

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
Vimba for Linux is a comprehensive SDK for all Allied Vision cameras with GigE and USB interface. You can download the ARM64 SDK <a href="https://www.alliedvision.com/en/products/vimba-sdk/#c1497">from Allied Vision's website</a>.  You can follow the instructions in the Allied Vision documentation to install the SDK. The result should look similar to the example below, where Vimba was installed in the /opt/Vimba_5_0 directory.

```
cd /opt/Vimba_5_0
```
```
cd /opt/Vimba_5_0/VimbaUSBTL
su -c ./Install.sh

Registering GENICAM_GENTL64_PATH
Registering VimbaUSBTL device types
Done
Please reboot before using the USB transport layer
Successfully installed VimbaPython-1.1.0
```

Please restart the system before proceeding with the next steps. The Vimba SDK has many pre-compiled examples in C and Python that you can use to test the camera and different configuration options. The C or Python APIs can be used to configure the camera properties such as gain, exposure, image size and also to capture images.

Install Vimba Python (you can find a description in the Readme in the VimbaPython repository):
```
root@j7-evm:/opt/Vimba_5_0/VimbaPython/Source# python3 -m pip install .
Processing /opt/Vimba_5_0/VimbaPython/Source
Installing collected packages: VimbaPython
    Running setup.py install for VimbaPython ... done
Successfully installed VimbaPython-1.1.0
```
 Please connect the camera to USB3.0 port of the TI StarterKit before the next steps.
To ensure the installation succeeded, you can use the Python examples included in the Vimba SDK. Using list_cameras.py in the Python Examples directory, you can see the cameras detected by the Vimba SDK. Each camera has a unique ID that will be used in the demos.

```
cd /opt/Vimba_5_0/VimbaPython/Examples
```
TI's edge AI SDK makes application development very easy with integrating gStreamer for efficient handling of video pipeline common in vision analytics applications. Allied Vision provides gStreamer support for Vimba at: https://github.com/alliedvision/gst-vimbasrc.

You can download the gst-vimbasrc code to a local directory, such as /opt/gst-vimbasrc-master. 

```
mkdir  /opt/gst-vimbasrc-master
mv  /opt/gst-vimbasrc-master
```

We recommend you to make a separate copy for the code changes in a different directory, for example:

```
cd /opt/edge_ai_apps
cp -pr app_python allied
cd /opt/edge_ai_apps/allied
```
In this directory, execute the following script which installs the gst plug-in for Allied Vision cameras. Please note that this script sets the VIMBA_HOME and the GST_PLUGIN_SYSTEM_PATH so that these are available for the applications. You can use gst-inspect-1.0 to ensure both vimba plugin and also kmssink plugin (part of edgeAI SDK) are visible as shown below.

```
#!/usr/bin/env bash
export VIMBA_HOME="/opt/Vimba_5_0"
cd /opt/gst-vimbasrc-master
source ./build.sh
cd /opt/edge_ai_apps/allied
export GST_PLUGIN_SYSTEM_PATH=/opt/gst-vimbasrc-master/build:/usr/lib/gstreamer-1.0
# Verify the gst pipeline with AlliedVision camera and KMSSINK
#gst-launch-1.0 vimbasrc camera=DEV_1AB22C00C604 ! videoscale ! videoconvert ! queue ! kmssink
gst-inspect-1.0 | grep vimba
gst-inspect-1.0 | grep kms
```

A successful build looks like this:
```
# ============================================================================
cmake -S . -B build -DVIMBA_HOME=$VIMBA_HOME
-- Configuring done
-- Generating done
-- Build files have been written to: /opt/gst-vimbasrc-master/build
# ============================================================================
cmake --build build
make[1]: Entering directory '/opt/gst-vimbasrc-master/build'
make[2]: Entering directory '/opt/gst-vimbasrc-master/build'
Scanning dependencies of target gstvimbasrc
make[2]: Leaving directory '/opt/gst-vimbasrc-master/build'
make[2]: Entering directory '/opt/gst-vimbasrc-master/build'
[ 25%] Building C object CMakeFiles/gstvimbasrc.dir/src/gstvimbasrc.c.o
[ 50%] Building C object CMakeFiles/gstvimbasrc.dir/src/vimba_helpers.c.o
[ 75%] Building C object CMakeFiles/gstvimbasrc.dir/src/pixelformats.c.o
[100%] Linking C shared library libgstvimbasrc.so
make[2]: Leaving directory '/opt/gst-vimbasrc-master/build'
[100%] Built target gstvimbasrc
make[1]: Leaving directory '/opt/gst-vimbasrc-master/build'
# ============================================================================
```

Now you can use Allied Vision cameras with Vimba and the gStreamer plugin. As a sanity check, use gst-launch-1.0 to capture data from the camera and send to the display. Below command can be used. As you notice in the command, camera configuration can also be done in the same command or optionally, you can pass an XML file with camera configuration parameters. More details are available at https://github.com/alliedvision/gst-vimbasrc.

```
gst-launch-1.0 vimbasrc camera=DEV_1AB22C00C604  exposureauto=2 height=720 width=1280 ! video/x-raw ,format=RGB ! videoscale ! videoconvert ! queue ! kmssink
```
Once you verified this, you can run the edgeAi demos with these cameras using edge AI SDK.


# Running Demo code

Once you verified this, now is the time to replace the input image portion in the EdgeAI demo framework. EdgeAI SDK has a modular architecture for input data, inferencing and output data. You can use the same software architecture as-is with only replacing how the input is captured.

There are two places where you need to make code changes. First one is setting the pipeline for the input video flow in the .yaml file used by the edge ai application. Below code snippet shows the changes where two additional input sources are added with the correct camera ID from the previous install. One camera is 'input_allied' and the other one is 'input_allied_mono'. In the flow for the inference, you can use the camera you want to use in the demo. The output can be display or a file.

```
title: "Object Detection Demo Testing with Allied Vision USB camera Sep 2021"
log_level: 0
inputs:
    input0:
        source: /dev/video1
        format: jpeg
        width: 1280
        height: 720
        framerate: 30
    input1:
        source: /opt/edge_ai_apps/data/videos/video_0000.mp4
        width: 1280
        height: 720
        framerate: 25
        loop: False
    input2:
        source: /opt/edge_ai_apps/data/images/%04d.jpg
        width: 1280
        height: 720
        index: 0
        framerate: 1
        loop: True
    input_allied:
        source: DEV_1AB22C00C604
        width: 1280
        height: 720
        index: 0
        framerate: 1
        loop: True
    input_allied_mono:
        source: DEV_1AB22C00C73D
        width: 1280
        height: 720
        index: 0
        framerate: 1
        loop: True
models:
    model0:
        model_path: /opt/model_zoo/TVM-OD-5020-yolov3-mobv1-gluon-mxnet-416x416
        viz_threshold: 0.6
    model1:
        model_path: /opt/model_zoo/TFL-OD-2020-ssdLite-mobDet-DSP-coco-320x320
        viz_threshold: 0.6
    model2:
        model_path: /opt/model_zoo/ONR-OD-8050-ssd-lite-regNetX-800mf-fpn-bgr-coco-512x512
        viz_threshold: 0.7
outputs:
    output0:
        sink: kmssink
        width: 1920
        height: 1080
    output1:
        sink: /opt/edge_ai_apps/data/output/videos/output_video.mov
        width: 1920
        height: 1080
    output2:
        sink: /opt/edge_ai_apps/data/output/images/output_image_%04d.jpg
        width: 1920
        height: 1080

flows:
    flow0:
        input: input_allied_mono
        models: [model1]
        outputs: [output0]
        mosaic:
            mosaic0:
                width:  1280
                height: 720
                pos_x:  320
                pos_y:  180

```

Second change is in the get_input_str function of gst_wrapper.py module where the corresponding gst pipeline is set according to the instructions in the previous setup. You can see the modularity of the code when we add additional input sources to make it easy for customers add different types of cameras.

```
def get_input_str(input):
    """
    Construct the gst input string
    Args:
        input: input configuration
    """

    elif ('%' in input.source):
        if (not os.path.exists(input.source % input.index)):
            status = 'no file'
            input.source = input.source % input.index
        elif (not (source_ext in image_dec)):
            status = 'fmt err'
        else:
            source = 'image'
    elif (input.source == 'videotestsrc'):
        source = 'videotestsrc'
    # Allied Vision Camera
    elif (input.source.startswith('DEV_')):
      vimbaCameraId = input.source
      source = 'vimba'
    else:
        status = 'no file'


    if (source == 'camera'):
        source_cmd = 'v4l2src device=' + input.source + ' io-mode=2 ! '
        if input.format == 'jpeg':
            source_cmd += 'image/jpeg, width=%d, height=%d ! ' % \
                                                     (input.width, input.height)
            source_cmd += 'jpegdec !'
        else:
            source_cmd += 'video/x-raw, width=%d, height=%d !' % \
                                                     (input.width, input.height)
    # Allied Vision Camera
    elif (source == 'vimba'):
        #gst-launch-1.0 vimbasrc camera=DEV_1AB22C00C604  exposureauto=2 ! video/x-raw ,format=RGB ! videoscale ! videoconvert ! queue ! kmssink
        #source_cmd = 'vimbasrc camera=DEV_1AB22C00C604 exposureauto=2 '
        source_cmd = 'vimbasrc camera='+vimbaCameraId+' exposureauto=2 '
        source_cmd += '! videoscale ! videoconvert ! queue ! '
	
```
Once you make the changes, you can run the demo just like the standard demos in the edgeAI SDK. Below commands are couple of examples for the object detection.

```
root@j7-evm:/opt/edge_ai_apps# ./app_edgeai.py ../configs/allied_od.yaml
```

Below is a sample output from the object detection demo on a scene with multiple objects being detected along with the bounding boxes.

![Object detection demo](https://cdn.alliedvision.com/fileadmin/content/images/landing_pages/Alvium_Camera_Kit_TI/7_av_color_od.gif)

Now, to change the input source from Alvium 1800 U-500c color camera to Alvium 1800 U-158m mono camera, all we need to do is change the .yaml file and run the demo.

Below is a sample output with the mono camera from the object detection demo on a scene with multiple objects being detected along with the bounding boxes.

![Object detection demo](https://cdn.alliedvision.com/fileadmin/content/images/landing_pages/Alvium_Camera_Kit_TI/7_av_mono_od.gif)

Below is a sample output from the semantic segmentation model running on the same scene.

![Semantic segmentation demo](https://cdn.alliedvision.com/fileadmin/content/images/landing_pages/Alvium_Camera_Kit_TI/7_av_mono_ss.gif)

That is about it.
There are only **two places where the code needs to be changed** if you are bringing a new camera to be able to run different demos on the edgeAI platform. With extensive model selection and modular software architecture, it is indeed very easy for developers to make such changes quickly and prototype new system designs with different camera modules needed for industrial applications.


