## Overview

The diagram below shows the main components of a system that is running the *Camera Streaming Daemon* on a Linux companion computer.

![Camera streaming daemon overview](../../assets/camera_streaming_daemon_overview.png)

The server:

* Scans and attaches to all compatible cameras. Cameras that support the [Video4Linux (V4L2) API](https://linuxtv.org/downloads/v4l-dvb-apis/uapi/v4l/v4l2.html) work out of the box (the server can be extended to support other camera protocols/APIs).
* Advertises and shares RTSP video streams for all connected cameras. These can be consumed by the GCS and other video player services.
* Captures and stores/shares video and still images for all connected cameras via the [MAVLink Camera Protocol](https://mavlink.io/en/protocol/camera.html). GCS requests for camera actions are forwarded by PX4. 

> **Note** The *Camera Streaming Daemon* also integrates with Gazebo, providing a simulated camera backend. In this case the diagram/configuration can be slightly different as the GCS can communicate directly with the server (there is no need for messages to be forwarded by PX4).