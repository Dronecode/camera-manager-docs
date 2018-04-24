# CSD Configuration File

CSD loads a configuration file with custom options/settings when it is started.

> **Tip** The [samples/config](https://github.com/intel/camera-streaming-daemon/tree/master/samples/config) directory contains sample configuration files that you can use for Ubuntu, Aero and other platforms. 

By default CSD will look for a configuration file in **/etc/csd/main.conf**. You can over-ride this file location using the `CSD_CONF_FILE` environment variable or with the `-c` switch when starting CSD.

The headings below explain the structure and main sections of the configuration file (see also [samples/config/config.sample](https://github.com/intel/camera-streaming-daemon/blob/master/samples/config/config.sample)).


## File Structure

The configuration file is composed of sections, each containing *keys* and *values*. For example:
```
[section-name]
someKey=someValue
aDifferentKey=anotherValue

[anothersection]
...

```

> **Tip** The keys/values are case insensitive (i.e. `Key=Value` is the same as `key=value`). By convention lower case is used.

## Expected Sections

The following sections, along with key/value definitions, are likely to be present in most configuration files. 

### [gstreamer]

Key | Description | Default
-- | --- | ---
`muxer` | Muxer used to create the *gstreamer* pipeline. | rtpjpegpay
`encoder` | Encoder used to create the *gstreamer* pipeline. | jpegenc
`converter` | Converter used to create the *gstreamer* pipeline. | videoconvert

Example:
```
[gstreamer]
muxer=rtph264pay
encoder=x264enc
converter=autovideoconvert
```

> **Note** The settings above select *gstreamer* plugins that are compatible with *QGroundControl*.


### [v4l2]

Key | Description | Default
-- | --- | ---
`blacklist` | Comma separated list of `/dev/video` devices that should not be exported (for RTSP or MAVLink). For example, if the video streams from cameras `/dev/video123` and `/dev/video456` could not be exported you would set: `blacklist = video123,video456`.| <empty>

Example (Aero):
```
[v4l2]
blacklist=video0,video1,video3,video4,video5,video6,video7,video8,video9,video10,video11,video12
```

> **Note** V4L2 creates multiple device "nodes" for each camera device (these are suitable for different purposes). The `blacklist` key is used to ignore additional nodes, so that CSD only creates a single interface for each camera.


### [mavlink]

Key | Description | Default
-- | --- | ---
`port` | MAVLink destination UDP port. | 14550
`broadcast_addr` | Broadcast address to send MAVLink heartbeat messages. | 255.255.255.255
`rtsp_server_addr` | IP address or hostname of the interface where the RTSP server is running. This is the address that will be used by the client to make the RTSP request. | 0.0.0.0
`system_id` | System ID of the CSD to be used in MAVLink communications. This should typically match the ID of the connected autopilot (i.e. the CSD/connected cameras are considered MAVLink *components* of the autopilot *system*). If not defined CSD will use 1 until it is able to read a value from an autopilot heartbeat. | 1

Example (for Ubuntu):
```
[mavlink]
port=14550
broadcast_addr=127.0.0.255
rtsp_server_addr=127.0.0.1
system_id=1
```

> **Note** There is no `component-id` key because these IDs are auto-allocated by CSD. If more than 6 cameras are added (5 when using Gazebo) the additional cameras will not be addressable, and CSD will log an error.

### [uri] {#uri}

Key | Description | Default
-- | --- | ---
`<camera-device-id>` | URI for the [Camera Definition File](../guide/camera_definition_file.md) of the camera device `<camera-device-id>` | -

> **Note** CSD needs to be able to supply URI locations of [Camera Definition Files](../guide/camera_definition_file.md) for attached cameras. CSD determines the URI for each camera using the mappings in this section.

Example (Gazebo): 
```
[uri]
gazebo=http://127.0.0.1:8000/camera-def-gazebo.xml
```

Example (Aero):
```
[uri]
video13=http://192.168.8.1:8000/camera-def-rs-rgb.xml
```

Example (Ubuntu):
```
[uri]
video0=http://127.0.0.1:8000/camera-def-uvc.xml
```

### [imgcap] {#imgcap}

This section defines the *default* values used for image capture. With the exception of `location` it should be possible override the values over MAVLink using parameters defined in the [Camera Definition File](../guide/camera_definition_file.md).

> **Note** At time of writing these default values cannot yet be overridden (see [#161](https://github.com/Dronecode/camera-streaming-daemon/issues/161)).

Key | Description | Default
-- | --- | ---
`width` | Width of the image to be captured in pixels. | Full width of camera frame for sensor type (i.e 1080P - 1920, 720P - 1280, etc.)
`height` | Height of the image to be captured in pixels. | Full height of camera frame for sensor type (i.e 1080P - 1080, 720P - 720, etc).
`format` | Image format (number). <br>Possible values:<ul><li>2 (Jpeg/<code>IMAGE_FILE_JPEG</code>)</li></ul>Notes:<ul><li>Currently only Jpeg is <a href="https://github.com/Dronecode/camera-streaming-daemon/issues/163">supported for image capture</a>.</li></ul> | 2 (JPEG).
`location` | The local path with write permission where captured images will be saved (usually the system-standard "Temp" directory). The path should be accessible and writeable. | /tmp/

Example:
```
[imgcap]
location=~/Temp/
```

### [vidcap] {#vidcap}

This section defines the *default* values used for video capture. With the exception of `location` it should be possible override the values over MAVLink using parameters defined in the [Camera Definition File](../guide/camera_definition_file.md).

> **Note** At time of writing these default values cannot yet be overridden (see [#161](https://github.com/Dronecode/camera-streaming-daemon/issues/161)).

Key | Description | Default
--- | --- | ---
`width` | Width of the video to be captured in pixels. | Full width of camera frame for sensor type (i.e 1080P - 1920, 720P - 1280, etc).
`height` | Height of the video to be captured in pixels. | Full height of camera frame for sensor type (i.e 1080P - 1080, 720P - 720, etc).
`framerate` | Camera framerate for video capture. | Default framerate queried from camera sensor (e.g. 25).
`bitrate` | Bitrate of the encoded video data in KBps. Supported values: 1 - 2048000. | 512
`encoder` | Encoder (number). <br>Possible values:<ul><li>3 (H.264/<code>VIDEO_CODING_AVC</code>)</li></ul>Notes:<ul><li>Currently only H.264 is <a href="https://github.com/Dronecode/camera-streaming-daemon/issues/168">supported for video capture</a>.</li></ul>  | 3 (AVC).
`format` | Video file format (number). <br>Possible values:<ul><li>1 (Moving Pictures Expert Group 4/<code>VIDEO_FILE_MP4</code>)</li></ul>Notes:<ul><li>Currently only MP4 is <a href="https://github.com/Dronecode/camera-streaming-daemon/issues/169">supported for video capture</a>.</li></ul> | 1 (MP4)
`location` | The local path with write permission where captured videos will be saved (usually the system-standard "Temp" directory). | -


Example (Ubuntu):
```
[vidcap]
width=640
height=480
framerate=25
bitrate=1000
encoder=3
format=1
location=/tmp/
```


### [gazebo]

Key | Description | Default
--- | --- | ---
`camtopic` | Gazebo topic where camera images are published. | -

Example:
```
[gazebo]
camtopic=~/typhoon_h480/cgo3_camera_link/camera/image
```
