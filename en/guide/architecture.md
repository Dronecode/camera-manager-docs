# Architecture

The diagram below shows the class diagram of *Camera Streaming Daemon*

![Camera Server class diagram](../../assets/camera_server_class_diagram.png)

## CameraServer

`CameraServer` is the class to detect camera devices, create and configure `CameraComponent` for each of them. The configuration settings is read from the conf file

## CameraComponent

`CameraComponent` is the class that creates and manages `CameraDevice` and other classes to provide camera services.

## MavlinkServer

`MavlinkServer` is the class that handles MAVLink communication.

## CameraDevice

`CameraDevice` is the abstract base class for all camera devices. Each camera device has a string identifier. Based on how the camera frames can be read, there are two types of camera devices.
- First is the camera device that supports gstreamer source element. The gst source element for such camera devices can be plugged in any gstreamer pipeline to implement the features like image capture, video streaming etc.
- Second is the camera device that does not support gstreamer source element. Such type of class must implement the read function to return camera frame buffers. Gstreamer pipeline with appsrc element is created for such devices. The appsrc element is fed with frames returned by read function.

## CameraParameters

`CameraParameters` is the class to store and retrieve parameters and its value. Camera parameter is identified by a name(string), ID(int) and type(enum). The parameter name, ID, type and its value(converted as string) is stored in the map data structure in the CameraParameters class. Few commonly used camera parameters are defined with name, ID and possible values. Developer can either use the predefined parameters or declare new custom parameters as per their requirement.

## ImageCapture

`ImageCapture` is the abstract base class for the image capture implementation

## VideoCapture

`VideoCapture` is the abstract base class for the video capture implementation

## VideoStream

`VideoStream` is the abstract base class for the Video Streaming implementation
