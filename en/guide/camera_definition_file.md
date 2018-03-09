# Camera Definition File

A MAVLink [Camera Definition File](https://mavlink.io/en/protocol/camera_def.html) contains information about a camera's advanced settings and parameters. If specified for a particular camera, these are downloaded by clients using a URI supplied by the camera (CSD).

This topic explains how to:
* Specify the mapping (in CSD) between cameras and their definition file URIs.
* Serve the definition files on the CSD companion computer.

> **Tip** CSD does not need to know anything about the *content* of the definition file. For more information see [MAVLink Camera Protocol](https://mavlink.io/en/protocol/camera.html) and [Camera Definition File](https://mavlink.io/en/protocol/camera_def.html).


## Hosting Camera Definition Files

Usually the computer that is hosting CSD should also serve the *Camera Definition Files* (to ensure that they are accessible to clients running on the same network). We recommend using the Python *SimpleHTTPServer* because it is lightweight, and because the fact that it is present in many Linux distributions (as part of Python) means that it requires no additional setup.

> **Tip** The definition files may be hosted at any URI that is *accessible to requesting clients*. On some connected systems this could mean "anywhere on the Internet".

To run *SimpleHTTPServer*:

1. Open a terminal in the directory that you want to serve.
2. Enter the following command to start the server on port 8080 (or specify another port)
   ```
   python -m SimpleHTTPServer 8080
   ```
   > **Note** Above uses Python 2.7. If you prefer Python 3, instead enter:
     ```
     python -m http.server
     ```
     
Assuming the directory contained a file **camera-def.xml** it would now be accessible:
* On the serving computer: 
  ```
  http://127.0.0.1:8080/camera-def.xml
  ```
* On the network (the `drone_ip>` can be obtained using *ifconfig*):
  ```
  http://<drone_ip>:8080/camera-def.xml
  ```


## CSD Definition File Mapping

The CSD mapping between a camera device id and the URI for a *Camera Definition File* URI is specified in the [\[uri section\]](../guide/configuration_file.md#uri) of the *CSD Configuration File*. 

The fragments below show typical mappings for Aero and Ubuntu:
* Aero
  ```
  [uri]
  video13=http://192.168.8.1:8000/camera-def-rs-rgb.xml
  ```
* Ubuntu
  ```
  [uri]
  video0=http://127.0.0.1/camera-def-R200rgb.xml
  video1=http://127.0.0.1/camera-def.xml
  ```

> **Note** The "Aero URI" host is the address of Aero on the network. The Ubuntu address can be "localhost" (internal) because when testing both CSD and clients (e.g. *QGroundControl*) are typically being run on the same computer and can see this address.
