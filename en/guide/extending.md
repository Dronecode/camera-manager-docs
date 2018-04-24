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

In order to support a new type of camera a custom class must be derived from `CameraDevice`. In addition, code must be added to detect the new device type and to instantiate the new custom camera device when this happens. The sections below detail these steps.

### 1. Extend CameraDevice Class

To add support for new type of camera device in DCM, a custom class must be derived from `CameraDevice`. 
An example `CameraDeviceCustom` (in green) can be seen in the [class diagram](../guide/architecture.md#class_diagram). 
The class `CameraDeviceCustom` represents a custom type of camera device.

**Action**: `CameraDeviceCustom` must implement the pure virtual functions of the base class `CameraDevice`. 

```cpp
class CameraDeviceCustom final : public CameraDevice {
public:
    CameraDeviceGazebo(std::string device);
    ~CameraDeviceGazebo();
    std::string getDeviceId();
    int getInfo(struct CameraInfo &camInfo);
    bool isGstV4l2Src();
    int init(CameraParameters &camParam);
    int uninit();
    int resetParams(CameraParameters &camParam);
    int setParam(CameraParameters &camParam, std::string param, const char *param_value,
                 size_t value_size, int param_type);
```

**Action**: `CameraDeviceCustom` may overload other methods to provide the functionality.

The configurable parameters of the custom camera device can be exported to the client (GCS) for control. Setting of these parameters and resetting of all the parameters must be handled by the `CameraDeviceCustom` class.

**Action*: `CameraDeviceCustom` must declare the parameter name (string), ID (int) and type (enum). Also set the default value of the parameter.

```cpp
int CameraDeviceCustom::init(CameraParameters &camParam)
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


### 2. Detect Custom Camera Device

There must be a logic to detect the custom camera device.
For example V4L2 camera devices are detected by scanning the Linux device nodes `/dev/video*` and Gazebo camera is detected based on [--enable-gazebo](../getting_started/building_installation.md#configure) compile time flag.

**Action**: Implement function to detect the custom camera device.

```cpp
int CameraServer::detect_devices_custom(ConfFile &conf, std::vector<CameraComponent *> &camList)
```
**Action**: Call the function `detect_devices_custom()` from the function that prepares the list of cameras in the system.

```cpp
// prepare the list of cameras in the system
int CameraServer::detectCamera(ConfFile &conf)
```

### 3. Instantiate Custom Camera Device

After detection of the custom camera device, `CameraServer` will instantiate a `CameraComponent` and pass the custom camera device ID to the `CameraComponent`. The `CameraComponent` will create an instance of `CameraDeviceCustom` object based on the string ID received from `CameraServer`.

**Action** : Add conditional statement in [create_camera_device](https://github.com/Dronecode/camera-manager/blob/master/src/CameraComponent.cpp#L354) function to find if the string ID is of type custom camera and instantiate `CameraDeviceCustom` object.

```cpp
    } else if (camdev_name.find("camera/custom") != std::string::npos) {
        return std::make_shared<CameraDeviceCustom>(camdev_name);
```

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
