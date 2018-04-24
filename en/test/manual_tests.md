# Manual Testing

This topic shows how to test the main functionality provided by DCM:
* List the video streams that are being published and display the video streams.
* Control the camera using MAVLink.

## List Available Streams with Avahi

You can use `avahi-browse` to see the list of connected cameras and the details of their RTSP stream (hostname, address, port, etc).

The command (and some typical example output) are shown below:
```
$ avahi-browse -r _rtsp._udp

+ enp0s3 IPv4 /rsir2                                        _rtsp._udp           local
+ enp0s3 IPv4 /rsir                                         _rtsp._udp           local
+ enp0s3 IPv4 /rsdepth                                      _rtsp._udp           local
+ enp0s3 IPv4 /video0                                       _rtsp._udp           local
= enp0s3 IPv4 /rsir                                         _rtsp._udp           local
   hostname = [ubuntu-VirtualBox.local]
   address = [10.0.2.15]
   port = [8554]
   txt = ["name=RealSense Infrared Camera" "frame_size[0]=NV12(640x480)"]
= enp0s3 IPv4 /rsir2                                        _rtsp._udp           local
   hostname = [ubuntu-VirtualBox.local]
   address = [10.0.2.15]
   port = [8554]
   txt = ["name=RealSense Infrared Camera2" "frame_size[0]=NV12(640x480)"]
= enp0s3 IPv4 /video0                                       _rtsp._udp           local
   hostname = [ubuntu-VirtualBox.local]
   address = [10.0.2.15]
   port = [8554]
   txt = ["name=VirtualBox Webcam - Integrated " "frame_size[0]=MJPG(640x480,160x120,320x180,320x240,424x240,640x360,848x480,960x540,1280x720,1920x1080)"]
= enp0s3 IPv4 /rsdepth                                      _rtsp._udp           local
   hostname = [ubuntu-VirtualBox.local]
   address = [10.0.2.15]
   port = [8554]
   txt = ["name=RealSense Depth Camera" "frame_size[0]=NV12(640x480)"]
```

## Video Streaming

Testing of video streaming can be done using a vlc player or a gstreamer client pipeline or a QGroundcontrol app.

To test video streaming using vlc:
```sh
vlc rtsp://<ip_dcm_running_system>:8554/videox
```

To test video streaming using gstreamer client pipeline:
```
gst-launch-1.0 uridecodebin uri=<ip_dcm_running_system>:8554/videox
```

In order to test the video streaming using QGC, select **RTSP Video Source** in **General Setting > Video > Video Source** and fill in URI as `rtsp://ip_dcm_running_system:8554/videox`. Observe the video in the video area in *Fly View*.

## Camera Control 

Testing camera control parameters requires DCM be [built with MAVLink enabled](../getting_started/building_installation.md#configure), a QGC app build from master and a HTTP server (Follow https://www.digitalocean.com/community/tutorials/how-to-install-linux-apache-mysql-php-lamp-stack-on-ubuntu-16-04 to install the *Apache* server).

1. Host the camera definition file present in samples directory using a HTTP server. To host using the *Apache* server copy the file to /var/www/html:
   ```
   cp ~/camera-manager/samples/defcamera-def-rs-rgb.xml /var/www/html
   ```

   The aero image has an aero-http server to host camera definition file. Copying the camera definition files to /var/http will host the files.
   
1. Update the `uri` section in conf file with appropriate IP address. Sample conf files are present in files folder in samples directory.
1. Open QGC build from master. Select **General Settings > Video Source > RTSP Video Stream**. Fill in the video URI and observe the video in fly view. 
1. In *Fly View*, select camera from dropdown list located in top right corner under compass. Settings button in left top corner of the box can be used to set the control parameters of the camera.

### Image capture

Testing image capture requires that DCM is [built with MAVLink enabled](../getting_started/building_installation.md#configure) and a QGC app built from master.

1. In QGC, disable the video streaming: **General Settings > Video Source > Video Stream Disabled**.
1. Create a directory to store the image and update the location in `imgcap` section of the conf file.
1. Now, in fly view of QGC app select *camera mode* and click on the red button to capture an image. The image will be stored in the location given in `imgcap` section in *conf* file.
