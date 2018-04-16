# CSD Architecture

Theis topic provides an overview of the *Camera Streaming Daemon* architecture, using a class diagram and a description of the main classes.

> **Tip** Developers who want to support new types of cameras or custom video streams should see [Extending CSD](../guide/extending_csd.md).

## Class Diagram {#class_diagram}

![Camera Server class diagram](../../assets/camera_server_class_diagram.png)

## CameraServer

The `CameraServer` class detects camera devices attached to the host computer and creates (and configures) a `CameraComponent` for each of them. 
The configuration settings are read from the [CSD Configuration File](../guide/configuration_file.md).

## CameraComponent

The `CameraComponent` class creates and manages all the objects associated with a particular connected camera. 
This includes `CameraDevice` objects, which are used to connect to the hardware, and other objects that provide camera services.

## MavlinkServer

`MavlinkServer` handles MAVLink communication.

## CameraDevice

`CameraDevice` is the abstract base class for all camera devices.
Each camera device has a string identifier. 
Based on how the camera frames can be read, there are two types of camera devices:
- Camera devices that support the *gstreamer* source element. 
  The gst source element for these types of camera devices can be plugged into any *gstreamer* pipeline to implement the features like image capture, video streaming etc.
- Camera device that do not support the *gstreamer* source element. 
  A *Gstreamer* pipeline with *appsrc* element is created for such devices. 
  Concrete classes for this type of device must implement the `read()` function to return camera frame buffers. 
  The *appsrc* element is fed with frames returned by `read()` function.

## CameraParameters

`CameraParameters` is used to store and retrieve parameters and their values. 
Each parameter is identified by a name (string), ID (int) and type (enum). 
The parameter name, ID, type and its value (converted to a string) are stored in a map data structure in the `CameraParameters` class.

Many commonly used camera parameters are already defined in `CameraParameters`. 
Developers can either use the predefined parameters or declare new custom parameters (in their subclass of `CameraDevice`).

## ImageCapture

`ImageCapture` is the abstract base class for the image capture implementation.

## VideoCapture

`VideoCapture` is the abstract base class for the video capture implementation.

## VideoStream

`VideoStream` is the abstract base class for the Video Streaming implementation.
