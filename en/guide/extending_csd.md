# Extending Camera Streaming Daemon

## Create a Custom Camera Device

Camera-Streaming-Daemon has native support for auto detection of video cameras exported as Video4Linux(V4L2) devices. This covers most of regular Linux cameras, but there are some cameras that don't have V4L2 support or cases where user wants to set very specific parameters or do custom post-processing in video before exporting.

For these use cases, it is possible to create a custom CameraDevice class, representing a single custom camera device.
CameraServer class keeps a list of all created CameraComponents, each one holding a single available CameraDevice. These CameraDevice objects are used by CameraComponent class for camera control and ImageCapture and VideoStream classes to perform the image capture and streaming.

[CameraDeviceGazebo](https://github.com/intel/camera-streaming-daemon/blob/master/src/CameraDeviceGazebo.cpp) is an example of a custom CameraDevice. We will explore it in order to learn all the steps necessary to add new custom CameraDevice in Camera-Streaming-Daemon.

### CameraDevice

CameraDevice is the abstract base class for all camera devices.

```cpp
class CameraDevice {
```
Every CameraDevice is identified by a unique string. For V4L2 class of camera devices, its the device node videox (For example video0 ). For Gazebo, its the topic on which the camera frames are advertised.

Based on how the camera frames can be read, there are two types of camera devices.
- First is the camera device that supports gstreamer source element. The gst source element for such camera devices can be plugged in any gstreamer pipeline to implement the features like image capture, video streaming etc.
- Second is the camera device that does not support gstreamer source element. Such type of class must implement the read function to return camera frame buffers. Gstreamer pipeline with appsrc element is created for such devices. The appsrc element is fed with frames returned by read function.

### CameraParameters
TODO : Description

#### 1. Extend CameraDevice Class
TODO : Description

Example: CameraDeviceGazebo is the class that extends CameraDevice class.

```cpp
class CameraDeviceGazebo final : public CameraDevice {
```
TODO : Details on Gazebo class

#### 2. Detect Camera Device
TODO : Description

```cpp
// prepare the list of cameras in the system
int CameraServer::detectCamera(ConfFile &conf)
```
Example: For Gazebo CameraDevice

```cpp
int CameraServer::detect_devices_gazebo(ConfFile &conf, std::vector<CameraComponent *> &camList)
```

#### 3. Instantiate Camera Device
TODO : Description

```cpp
std::shared_ptr<CameraDevice> CameraComponent::create_camera_device(std::string camdev_name)
```
Example: For Gazebo CameraDevice

```cpp
    } else if (camdev_name.find("camera/image") != std::string::npos) {
        log_debug("Gazebo device : %s", camdev_name.c_str());
#ifdef ENABLE_GAZEBO
        return std::make_shared<CameraDeviceGazebo>(camdev_name);
```

#### 4. Declare Camera Parameters
TODO:Desription

## Create a Custom RTSP Video Stream

Camera-Streaming-Daemon has native support for auto detection of video cameras exported as Video4Linux(V4L2) devices. This covers most of regular Linux cameras, but there are some cameras that don't have V4L2 support or cases where user wants to set very specific parameters or do custom post-processing in video before exporting.

For these use cases, it is possible to create a custom StreamBuilder class, creating custom Streams elements. 
StreamManager class keeps a list of all created Stream elements, each one representing a single available video stream. These Stream objects are used by AvahiPublisher and RTSPServer classes to perform the advertisement and streaming.

There are two samples in [samples directory](https://github.com/01org/camera-streaming-daemon/tree/master/samples) using this approach, camera-sample-custom and camera-sample-realsense. We will explore camera-sample-custom in order to learn all steps necessary to create a custom video stream class in Camera-Streaming-Daemon

### Camera Sample Custom

Camera Sample Custom adds two new classes to the daemon in order to support custom classes: StreamCustom and StreamBuilderCustom. StreamCustom is a class that extends Stream in order to represent a custom Stream. StreamBuilderCustom extends StreamBuilder in order to build the custom streams. No changes in main file or any other classes are necessary.

#### StreamCustom

StreamCustom class is a class that extends Stream class

```cpp
class StreamCustom : public Stream {
```

Stream class has some methods that needs to be implemented by children classes: get_path, get_name, get_gstreamer_pipeline and get_formats. Optionally finalize_gstreamer_pipeline can be overridden to clean any resource that was created by create_gstreamer_pipeline.

##### get_path: Returns the path that will be used to access the video resouce in rtsp URI.

```cpp
const std::string StreamCustom::get_path() const
{
    return "/custom";
}
```

##### get_name: Returns a human readable name of the video stream.

```cpp
const std::string StreamCustom::get_name() const
{
    return "Custom Stream";
}
```

##### get_formats: Returns an optional list of supported formats and resolutions. In this case, an empty list.

```cpp
const std::vector<Stream::PixelFormat> &StreamCustom::get_formats() const
{                                                                        
    static std::vector<Stream::PixelFormat> formats;                     
    return formats;                                                      
}                                                                        
```

##### creates_gstreamer_pipeline: Creates the gstreamer pipeline to access the video to be exported.

For this sample we use a really simple pipeline, that uses gstreamer videotestsrc to generate a sample video. For a more complex example, take a look at the [realsense sample](https://github.com/01org/camera-streaming-daemon/tree/master/samples/stream_realsense.cpp).

This method is called when the client starts the video streaming. Cleanup for resources created by this method can be performed at `finalize_gstreamer_pipeline` method.

```cpp
GstElement *StreamCustom::create_gstreamer_pipeline(std::map<std::string, std::string> &params) const
{
    GError *error = nullptr;
    GstElement *pipeline;

    pipeline = gst_parse_launch("videotestsrc ! video/x-raw,width=640,height=480 ! "
                                "videoconvert ! jpegenc ! rtpjpegpay name=pay0",
                                &error);
    if (!pipeline) {
        log_error("Error processing pipeline for custom stream device: %s\n",
                  error ? error->message : "unknown error");
        if (error)
            g_clear_error(&error);

        return nullptr;
    }

    return pipeline;
}
```

##### finalize_gstreamer_pipeline: Called when gstreamer pipeline is finalized so all needed cleanup can be
performed.

Note that there is no need of freeing or g_object_unref the pipeline. This method is used only if custom class allocate any other resource for the video. 

For the custom sample there is no need of having this function implemented. For a more complex example, take a look at the [realsense sample](https://github.com/01org/camera-streaming-daemon/tree/master/samples/stream_realsense.cpp).

```cpp
void StreamRealSense::finalize_gstreamer_pipeline(GstElement *pipeline)
{
    //Perform clean up
}
```

#### StreamBuilderCustom
This is the class that will create the custom streams. It is essential that it extends StreamBuilder, because StreamBuilder constructor will register it as a `StreamBuilder` class.

```cpp
class StreamBuilderCustom final : public StreamBuilder {
```

And it is necessary to create one instance of the StreamBuilderCustom object. We suggest doing it as a static variable in the cpp file
```cpp
static StreamBuilderCustom stream_builder;
```

Method build_streams should be implemented to create the desired streams. This method is called by StreamManager and Streams created by it are kept during the execution of Camera-Streaming-Daemon to be used when publishing avahi services and when creating the *gstreamer* pipeline to stream the video.

```cpp
std::vector<Stream *> StreamBuilderCustom::build_streams()       
{                                                                
    std::vector<Stream *> streams;                               
                                                                 
    streams.push_back(new StreamCustom());                       
                                                                 
    return streams;                                              
}                                                                
```

### Adding StreamCustom and StreamBuilderCustom to the build system

Now it is necessary to build and link `StreamCustom` and `StreamBuildCustom` classes to your final binary. We use [samples/Makefile.am](https://github.com/01org/camera-streaming-daemon/tree/master/samples/Makefile.am) file to add them
```makefile
camera_sample_custom_SOURCES = \  
   ${base_files} \               
   ../src/main.cpp \             
   stream_builder_custom.h \     
   stream_builder_custom.cpp \   
   stream_custom.cpp \           
   stream_custom.h               
```

Note that the main file used is the same main file from camera-streaming-daemon
