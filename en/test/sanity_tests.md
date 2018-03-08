# Sanity Tests

This section outlines some basic sanity tests that you can use to verify that CSD is working properly.

<!-- need to check all for Gazebo too -->

## Verify Linux Cameras

This test verifies that Linux itself is aware that a camera(s) are connected.

Enter the following command to list connected video sources:
```
ls -l /dev/video*
```

You should see different messages depending on whether or not a camera is connected:
```
crw-rw----+ 1 root video 81, 0 Mar  2 16:15 /dev/video0
```
```
ls: cannot access 'dev/video*': No such file or directory
  ```

If no camera is detected you'll need to further debug Linux (not a CSD problem).

> **Tip** You can also use the *v4l-utils* tools to detect compatible cameras. 
> ```
> sudo apt-get install v4l-utils
> v4l2-ctl --list-devices
> ```

## Verify CSD Startup

This test verifies what cameras CSD has detected, what streams were created, and also confirms which services were started (i.e. RTSP video streaming, MAVLink and Avahi).

Simply run CSD with verbose logging (via the `-v` flag). The example below shows the console output generated on a Ubuntu system with an integrated webcam:

```
$ ./csd -v -c samples/files/ubuntu.conf

ConfFile: Adding section 'gstreamer'
ConfFile: Adding section 'v4l2'
ConfFile: Adding section 'mavlink'
ConfFile: Adding section 'uri'
ConfFile: Adding section 'imgcap'
Found V4L2 camera device video0
v4l2 device :: /dev/video0
CameraComponent path:/dev/video0 with Camera Definition
V4L2 device : /dev/video0
addCameraComponent
CAMERA SERVER START
MAVLINK START

Open UDP [5]
Adding fd: 5 glib_flags: 1
Adding stream /video0 (VirtualBox Webcam - Integrated )
Adding stream /rsdepth (RealSense Depth Camera)
Adding stream /rsir (RealSense Infrared Camera)
Adding stream /rsir2 (RealSense Infrared Camera2)
AVAHI START

Starting Camera Streaming Daemon
UDP: [5] wrote 21 bytes
...
```


<!-- TODO Test above with Gazebo, Note that QGC latest does work, update intro to link forward to the quickstarts.Verify the quickstarts capture everything in Ubuntu quickstart version. --> 