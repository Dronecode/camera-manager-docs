# Camera Definition File

As defined in the [MAVLink Camera Protocol](https://mavlink.io/en/protocol/camera.html), a [Camera Definition File](https://mavlink.io/en/protocol/camera_def.html) may be used to specify information about a camera's advanced settings and parameters. A camera will supply the URI for its definition file on request (if one has been defined), so that the file can be downloaded and parsed by the client.

> **Tip** Camera definition files for a number of cameras are provided in [/samples/def](https://github.com/intel/camera-streaming-daemon/tree/master/samples/def). Definition files for attached Video4Linux (V4L2) devices can be create using [export_v4l2_param_xml](#export_v4l2_param_xml).  

The *Camera Definition Files* can be hosted anywhere that is accessible to clients (including *QGroundControl* and Dronecode SDK apps). Often the files are served from the computer running CSD.

CSD is able to supply the URIs for attached cameras using a mapping given in the [CSD Configuration File > \[uri section\]](../guide/configuration_file.md#uri).

This topic explains how to:
* Serve the definition files on the CSD companion computer.
* Map attached cameras to their definition file URIs in the CSD configuration file.
* Create *Camera Definition Files* for attached Video4Linux (V4L2) devices using the *export_v4l2_param_xml* tool. 


## Hosting Camera Definition Files

Often the computer that is hosting CSD also serves the *Camera Definition Files*. If this is needed, we recommend using the Python *SimpleHTTPServer* because it is lightweight, and because the fact that it is present in many Linux distributions (as part of Python) means that it requires no additional setup.

> **Note** Intel Aero uses *SimpleHTTPServer* by default (so this does not need to be set up on Aero).

<span></span>
> **Tip** The definition files may be hosted at any URI that is *accessible to requesting clients*. On some connected systems this could mean "anywhere on the Internet".

To run *SimpleHTTPServer*:

1. Open a terminal in the directory that you want to serve.
1. Enter the following command to start the server on the default port (8000):
   ```
   python -m SimpleHTTPServer
   ```
   > **Note** Above uses Python 2.7. If you prefer Python 3, instead enter:
     ```
     python -m http.server
     ```
     
Assuming the directory contained a file **camera-def.xml** it would now be accessible:
* On the serving computer: 
  ```
  http://127.0.0.1:8000/camera-def.xml
  ```
* On the network (the `<drone_ip>` can be obtained using *ifconfig*):
  ```
  http://<drone_ip>:8000/camera-def.xml
  ```


## CSD Definition File Mapping

The CSD mapping between a camera device id and the URI for a *Camera Definition File* is specified in the [\[uri section\]](../guide/configuration_file.md#uri) of the *CSD Configuration File*. 

The fragments below show typical mappings for Aero and Ubuntu:
* Aero
  ```
  [uri]
  video13=http://192.168.8.1:8000/camera-def-rs-rgb.xml
  ```
* Ubuntu
  ```
  [uri]
  video0=http://127.0.0.1:8000/camera-def-uvc.xml
  ```

> **Note** The "Aero URI" host is the address of Aero on the network. The Ubuntu address can be "localhost" (internal) because when testing both CSD and clients (e.g. *QGroundControl*) are typically being run on the same computer and can see this address.


## V4L2 Camera Definition File Generator {#export_v4l2_param_xml}

The CSD project includes the *export_v4l2_param_xml* tool to automate creation of *Camera Definition Files* for attached Video4Linux (V4L2) devices. 
The tool queries the device for all available settings in order to populate the file.

The command syntax is:
```
./export_v4l2_param_xml -d  <device node>  - f <output camera-def-file-name>
```
where:
* `device node`: The V3L2 device (query available devices on the command line using: `ls -l /dev/video*`).
* `output camera-def-file-name`: The generated file path/filename.


The tool is provided as C++ source code in the CSD tree: [/tools/export_v4l2_param_xml](https://github.com/Dronecode/camera-streaming-daemon/tree/master/tools/export_v4l2_param_xml)). It is dependent on *tinyxml* which can be installed using:
```
$ sudo apt install libtinyxml-dev -y
```

To build and run the tool for a device `/dev/video0` (from the CSD root directory):
```
cd tools/export_v4l2_param_xml
make
./export_v4l2_param -d /dev/video0 -f camera_def.xml
```
