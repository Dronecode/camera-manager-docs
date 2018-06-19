# Dronecode Camera Manager

[![Build Status](https://travis-ci.org/Dronecode/camera-manager.svg?branch=master)](https://travis-ci.org/intel/camera-streaming-daemon)
<a href="https://scan.coverity.com/projects/01org-camera-streaming-daemon"><img alt="Coverity Scan Build Status" src="https://scan.coverity.com/projects/12056/badge.svg"/></a>

The *Dronecode Camera Manager* (DCM) is an extensible Linux camera server for interfacing any camera with the [Dronecode Platform](https://www.dronecode.org/).

DCM can connect to multiple cameras and provides access to them via the [MAVLink Camera Protocol](https://mavlink.io/en/protocol/camera.html) and RTSP video streams (it can also advertise available RTSP streams).

Out of the box the DCM connects to cameras that support the [Video4Linux (V4L2) API](https://linuxtv.org/downloads/v4l-dvb-apis/uapi/v4l/v4l2.html) and can also provide access to a camera running within a Gazebo simulation. The DCM has an extensible back-end that can be used to support other types of cameras and also new front-end protocols.

> **Tip** The *Camera Manager* is the easiest way for Camera OEMs to interface with the *Dronecode Platform*. Many cameras will just work "out of the box". At most, OEMs may need extend the backend to support their own Linux camera interface.


## Forums and Chat {#support}

The core development team and community are active on the following chat channel:

* [Slack](http://slack.px4.io) (sign up) - channel `#camera-payload-api`


## Reporting Bugs & Issues

If you have any problems using the *Camera Manager* first post them on the [support channels above](#support).

If directed by the development team, issues may be raised on [Github here](https://github.com/Dronecode/camera-manager/issues).


## Contributing

The [Contributing Guide](contribute/README.md) explains the contribution model and the main areas where you can help.


## Licence

The *Camera Manager* source code is free to use and modify under terms of the permissive
[Apache License 2.0](https://github.com/Dronecode/camera-manager/blob/master/LICENSE).

This documentation is licensed under *CC BY 4.0* ([Human readable overview](https://creativecommons.org/licenses/by/4.0/) | [LICENSE](https://github.com/Dronecode/camera-manager-docs/blob/master/LICENSE)).
