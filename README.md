# YOLOv4 ROS

## Overview

This is a ROS package for YOLOv4. It is modified from the repository https://github.com/leggedrobotics/darknet_ros/

## Installation

In order to install darknet_ros, clone the latest version from this repository into your catkin workspace and compile the package using ROS.

    cd yolov4-ws/src
    git clone --recursive https://github.com/t1mkhuan9/yolov4-ros-noetic darknet_ros

Then compile the package by

    cd ../
    catkin_make 

### Common Fixes
If you are compiling with CUDA, you might receive the following build error:

    nvcc fatal : Unsupported gpu architecture 'compute_61'.

This means that you need to check the compute capability (version) of your GPU. You can find a list of supported GPUs in CUDA here: [CUDA - WIKIPEDIA](https://en.wikipedia.org/wiki/CUDA#Supported_GPUs). Simply find the compute capability of your GPU and add it into darknet_ros/CMakeLists.txt. Simply add a similar line like

    -O3 -gencode arch=compute_62,code=sm_62

This build error may also occur sometimes:

    image.c:16:10: fatal error: stb_image.h: No such file or directory
        #include "stb_image.h"

This is because it is not correctly referenced. Change the include statement in `darknet/src/image.c` line 16 and line 20 to

    #include "../3rdparty/stb/include/stb_image.h"
    #include "../3rdparty/stb/include/stb_image_write.h"

Notice: DO NOT change if the problem does not occur.

### Download weights

The yolo-voc.weights and tiny-yolo-voc.weights are downloaded automatically in the CMakeLists.txt file. If you need to download them again, go into the weights folder and download the two pre-trained weights from the COCO data set:

    cd catkin_workspace/src/darknet_ros/darknet_ros/yolo_network_config/weights/
    wget http://pjreddie.com/media/files/yolov2.weights
    wget http://pjreddie.com/media/files/yolov2-tiny.weights

And weights from the VOC data set can be found here:

    wget http://pjreddie.com/media/files/yolov2-voc.weights
    wget http://pjreddie.com/media/files/yolov2-tiny-voc.weights

And the pre-trained weight from YOLO v3 can be found here:

    wget http://pjreddie.com/media/files/yolov3-tiny.weights
    wget http://pjreddie.com/media/files/yolov3.weights

There are more pre-trained weights from different data sets reported [here](https://pjreddie.com/darknet/yolo/).

### Use your own detection objects

In order to use your own detection objects you need to provide your weights and your cfg file inside the directories:

    catkin_workspace/src/darknet_ros/darknet_ros/yolo_network_config/weights/
    catkin_workspace/src/darknet_ros/darknet_ros/yolo_network_config/cfg/

In addition, you need to create your config file for ROS where you define the names of the detection objects. You need to include it inside:

    catkin_workspace/src/darknet_ros/darknet_ros/config/

Then in the launch file you have to point to your new config file in the line:

    <rosparam command="load" ns="darknet_ros" file="$(find darknet_ros)/config/your_config_file.yaml"/>

### Unit Tests

Run the unit tests using the [Catkin Command Line Tools](http://catkin-tools.readthedocs.io/en/latest/index.html#)

    catkin build darknet_ros --no-deps --verbose --catkin-make-args run_tests

You will see the image above popping up.

## Basic Usage

In order to get YOLO ROS: Real-Time Object Detection for ROS to run with your robot, you will need to adapt a few parameters. It is the easiest if duplicate and adapt all the parameter files that you need to change from the `darknet_ros` package. These are specifically the parameter files in `config` and the launch file from the `launch` folder.

## Nodes

### Node: darknet_ros

This is the main YOLO ROS: Real-Time Object Detection for ROS node. It uses the camera measurements to detect pre-learned objects in the frames.

### ROS related parameters

You can change the names and other parameters of the publishers, subscribers and actions inside `darknet_ros/config/ros.yaml`.

#### Subscribed Topics

* **`/camera_reading`** ([sensor_msgs/Image])

    The camera measurements.

#### Published Topics

* **`object_detector`** ([std_msgs::Int8])

    Publishes the number of detected objects.

* **`bounding_boxes`** ([darknet_ros_msgs::BoundingBoxes])

    Publishes an array of bounding boxes that gives information of the position and size of the bounding box in pixel coordinates.

* **`detection_image`** ([sensor_msgs::Image])

    Publishes an image of the detection image including the bounding boxes.

#### Actions

* **`camera_reading`** ([sensor_msgs::Image])

    Sends an action with an image and the result is an array of bounding boxes.

### Detection related parameters

You can change the parameters that are related to the detection by adding a new config file that looks similar to `darknet_ros/config/yolo.yaml`.

* **`image_view/enable_opencv`** (bool)

    Enable or disable the open cv view of the detection image including the bounding boxes.

* **`image_view/wait_key_delay`** (int)

    Wait key delay in ms of the open cv window.

* **`yolo_model/config_file/name`** (string)

    Name of the cfg file of the network that is used for detection. The code searches for this name inside `darknet_ros/yolo_network_config/cfg/`.

* **`yolo_model/weight_file/name`** (string)

    Name of the weights file of the network that is used for detection. The code searches for this name inside `darknet_ros/yolo_network_config/weights/`.

* **`yolo_model/threshold/value`** (float)

    Threshold of the detection algorithm. It is defined between 0 and 1.

* **`yolo_model/detection_classes/names`** (array of strings)

    Detection names of the network used by the cfg and weights file inside `darknet_ros/yolo_network_config/`.
