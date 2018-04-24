# Quickstart Guide — Ubuntu Camera

This Quickstart shows how to build DCM on Ubuntu LTS 16.04 (with support for MAVLink and Avahi) and connect to any supported cameras (or webcams) that are attached to Ubuntu. 

1. Install the [Core](#core_deps) and [Avahi](#avahi_deps) pre-requisites:
   ```sh
   sudo apt-get update -y
   sudo apt-get install git autoconf libtool python-pip -y
   sudo apt-get install gstreamer-1.0 \
       libgstreamer-plugins-base1.0-dev \
       libgstrtspserver-1.0-dev -y
   ## Required python packages
   sudo pip2 -q install -U future
   # Avahi
   sudo apt-get install libavahi-client-dev libavahi-core-dev libavahi-glib-dev -y
   ```
1. Get the DCM source code by cloning the the [camera-manager](https://github.com/Dronecode/camera-manager) repo (or your fork):
   ```sh
   git clone https://github.com/Dronecode/camera-manager.git
   cd camera-manager
   git submodule update --init --recursive
   ```
1. Configure DCM with support MAVLink and Avahi:
```
./autogen.sh && ./configure --enable-mavlink --enable-avahi
```
1. Build DCM:
   ```
   make
   ```
1. Attach [DCM-compatible cameras](../guide/overview.md#supported_cameras) to Ubuntu
1. Serve the sample [Camera Definition Files](../guide/camera_definition_file.md):
   * Open a new terminal to [/samples/def](https://github.com/Dronecode/camera-manager/tree/master/samples/def)
   * Enter the following command to start the server on the default port (8000):
     ```
     python -m SimpleHTTPServer
     ```
1. Run DCM specifying the Ubuntu [configuration file](../getting_started/building_installation.md#configuration-file-runtime).
   ```
   ./dcm -c samples/config/ubuntu.conf
   ```

Then run the [Sanity Tests](../test/sanity_tests.md) to verify that DCM is working correctly.
