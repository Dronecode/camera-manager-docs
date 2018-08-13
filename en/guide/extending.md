# Extending Camera Manager

The *Dronecode Camera Manager* (DCM) is [architected](../guide/architecture.md) so that it can be extended to integrate with any type of camera (attached to the host OS) and expose any camera feature. This topic explains main steps for extending the different features. 

> **Tip** Out of the box DCM already [supports many cameras](../guide/overview.md#supported-cameras-supported_cameras) including most regular Linux cameras (those that use the Video4Linux API). It also provides access to camera features including image capture, video capture and UDP and RTSP video streaming via *GStreamer*. 

## Extensible Features
 
Feature | Description
--- | ---
Custom Camera Devices/Parameters | DCM can be extended to support any (other) camera type and any configurable camera parameter can be declared and exported to a client (GCS).
Image Capture | Developers can implement new types of image capture (for example, image capture using *OpenCV* instead of *GStreamer*).
Video Capture | Developers can implement new types of video capture (for example, video capture using a multimedia framework other than *GStreamer*).
Video Streaming | Developers can implement new types of video streaming (e.g. HLS protocol).

## Support Custom Camera Device

Developers may want to add support for a new type of camera device that is not currently supported by DCM. To support a custom type of camera, a `PluginXXX` class derived from `PluginBase` and `CameraDeviceXXX` class derived from `CameraDevice` must be implemented. An example of a plugin for a custom camera device with classes `PluginCustom`(https://github.com/Dronecode/camera-manger/tree/master/plugins) and `CameraDeviceCustom`(https://github.com/Dronecode/camera-manger/tree/master/plugins) is added in the project and may be referenced.
The sections below detail these steps.

### 1. Extend PluginBase Class

Plugins are self-registering objects that discovers, enlists and creates camera devices.

**Action**: `PluginCustom` must implement the pure virtual functions of the base class `PluginBase`.

The plugin will prepare a list of camera devices that are attached to the system.

**Action**: `PluginCustom` must implement a logic to discover camera devices.

### 2. Extend CameraDevice Class

To add support for new type of camera device in DCM, `CameraDeviceCustom` class must be derived from `CameraDevice`. 

**Action**: `CameraDeviceCustom` must implement the pure virtual functions of the base class `CameraDevice`. 

There are few methods in `CameraDevice` class that are not pure virtual. By default it returns `NOT_SUPPORTED`.

**Action**: `CameraDeviceCustom` may overload not-pure virtual methods to provide the functionality.

The configurable parameters of the custom camera device can be exported to the client (GCS) for control. Setting of these parameters and resetting of all the parameters must be handled by the `CameraDeviceCustom` class.

**Action*: `CameraDeviceCustom` must declare the parameter name (string), ID (int) and type (enum). Also set the default value of the parameter.

```cpp
CameraDevice::Status CameraDeviceCustom::init(CameraParameters &camParam)
{
    camParam.setParameterIdType(CameraParameters::CAMERA_MODE,
                                CameraParameters::PARAM_ID_CAMERA_MODE,
                                CameraParameters::PARAM_TYPE_UINT32);
    camParam.setParameter(CameraParameters::CAMERA_MODE,
                          (uint32_t)CameraParameters::ID_CAMERA_MODE_VIDEO);
    camParam.setParameterIdType("brightness", 101,
                                CameraParameters::PARAM_TYPE_UINT8);
    camParam.setParameter("brightness", (uint8_t)50);
    camParam.setParameterIdType("contrast", 102,
                                CameraParameters::PARAM_TYPE_UINT32);
    camParam.setParameter("contrast", (uint32_t)50);
    camParam.setParameterIdType("hue", 103,
                                CameraParameters::PARAM_TYPE_INT32);
    camParam.setParameter("hue", (int32_t)-10);
    camParam.setParameterIdType("zoom", 104,
                                CameraParameters::PARAM_TYPE_REAL32);
    camParam.setParameter("zoom", (float)0);
    camParam.setParameterIdType("white_balance_mode", 105,
                                CameraParameters::PARAM_TYPE_UINT32);
    camParam.setParameter("white_balance_mode", (uint32_t)0);

    return 0;
}
```

**Action**: The `CameraDeviceCustom` must handle setting and resetting of the declared parameters.


### 3. Create CameraDevice

The `CameraServer` will query the `PluginManager` for the list of camera devices that are detected. `CameraServer` will create the `CameraDevice` if the device is not in the blacklist.

## Custom RTSP Video Stream

Developers might want to provide a custom RTSP video stream for cameras that don't have V4L2 support, where users need to set very specific parameters, or to do custom post-processing in video before exporting.

For these use cases, it is possible to create a custom `StreamBuilder` class, creating custom `Stream` elements. 
The `StreamManager` class keeps a list of all created `Stream` elements, each one representing a single available video stream. 
These `Stream` objects are used by `AvahiPublisher` and `RTSPServer` classes to perform the streaming and stream-advertising.

There are two samples in [samples directory](https://github.com/01org/camera-streaming-daemon/tree/master/samples) using this approach, 
**camera-sample-custom** and **camera-sample-realsense**. 
We will explore **camera-sample-custom** in order to learn all steps necessary to create a DCM custom video stream class.

### Camera Sample Custom

Camera Sample Custom adds two new classes to the manager in order to support custom classes: `StreamCustom` and `StreamBuilderCustom`. 
`StreamCustom` is a class that extends `Stream` in order to represent a custom `Stream`. 
`StreamBuilderCustom` extends `StreamBuilder` in order to build the custom streams.
 No changes in main file or any other classes are necessary.

#### StreamCustom

`StreamCustom` class is a class that extends `Stream` class:

```cpp
class StreamCustom : public Stream {
```

`Stream` class has some methods that need to be implemented by child classes: `get_path()`, `get_name()`, `get_gstreamer_pipeline()` and `get_formats()`. 
Optionally `finalize_gstreamer_pipeline()` can be overridden to clean any resource that was created by `create_gstreamer_pipeline()`.

##### get_path()

Returns the path that will be used to access the video resource in RTSP URI.

```cpp
const std::string StreamCustom::get_path() const
{
    return "/custom";
}
```

##### get_name()

Returns a human readable name of the video stream.

```cpp
const std::string StreamCustom::get_name() const
{
    return "Custom Stream";
}
```

##### get_formats()

Returns an optional list of supported formats and resolutions. In this case, an empty list.

```cpp
const std::vector<Stream::PixelFormat> &StreamCustom::get_formats() const
{                                                                        
    static std::vector<Stream::PixelFormat> formats;                     
    return formats;                                                      
}                                                                        
```

##### creates_gstreamer_pipeline()

Creates the gstreamer pipeline to access the video to be exported.

For this sample we use a really simple pipeline, that uses gstreamer videotestsrc to generate a sample video. 
For a more complex example, take a look at the [realsense sample](https://github.com/01org/camera-streaming-daemon/tree/master/samples/stream_realsense.cpp).

This method is called when the client starts the video streaming. 
Cleanup for resources created by this method can be performed at `finalize_gstreamer_pipeline` method.

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

##### finalize_gstreamer_pipeline()

 Called when gstreamer pipeline is finalized so all needed cleanup can be performed.

Note that there is no need of freeing or `g_object_unref` the pipeline. This method is used only if custom class allocate any other resource for the video. 

For the custom sample there is no need of having this function implemented. 
For a more complex example, take a look at the [realsense sample](https://github.com/01org/camera-streaming-daemon/tree/master/samples/stream_realsense.cpp).

```cpp
void StreamRealSense::finalize_gstreamer_pipeline(GstElement *pipeline)
{
    //Perform clean up
}
```

#### StreamBuilderCustom

This is the class that will create the custom streams. 
It is essential that it extends `StreamBuilder`, because `StreamBuilder` constructor will register it as a `StreamBuilder` class.

```cpp
class StreamBuilderCustom final : public StreamBuilder {
```

And it is necessary to create one instance of the `StreamBuilderCustom` object. 
We suggest doing it as a static variable in the cpp file:
```cpp
static StreamBuilderCustom stream_builder;
```

Method build_streams should be implemented to create the desired streams. 
This method is called by `StreamManager` and `Streams` created by it are kept during the execution of *Camera Manager* to be used when publishing the Avahi services and when creating the *gstreamer* pipeline to stream the video.

```cpp
std::vector<Stream *> StreamBuilderCustom::build_streams()       
{                                                                
    std::vector<Stream *> streams;                               
                                                                 
    streams.push_back(new StreamCustom());                       
                                                                 
    return streams;                                              
}                                                                
```

### Adding StreamCustom and StreamBuilderCustom to the build system

Now it is necessary to build and link `StreamCustom` and `StreamBuildCustom` classes to your final binary. 
We use [samples/Makefile.am](https://github.com/01org/camera-streaming-daemon/tree/master/samples/Makefile.am) file to add them:
```makefile
camera_sample_custom_SOURCES = \  
   ${base_files} \               
   ../src/main.cpp \             
   stream_builder_custom.h \     
   stream_builder_custom.cpp \   
   stream_custom.cpp \           
   stream_custom.h               
```

Note that the main file used is the same main file from DCM.
