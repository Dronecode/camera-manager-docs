# Quickstart Guide — Intel Aero

> **Tip** DCM is pre-integrated into *Intel Aero* images (1.6.1 and later). These instructions are only needed when developing and testing newer versions of DCM.

This guide provides turnkey instructions for building DCM on Ubuntu LTS 16.04 (with support for the RealSense 3D Camera, Aero bottom facing camera, MAVLink and Avahi) and then deploying it to *Intel Aero*.

## Build DCM

To build DCM:
1. Install the [Core](#core_deps), [Avahi](#avahi_deps) and [RealSense](#realsense_deps) pre-requisites listed above:
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
   # RealSense
   echo 'deb "http://realsense-alm-public.s3.amazonaws.com/apt-repo" xenial main' | sudo tee /etc/apt/sources.list.d/realsense-latest.list
   sudo apt-key adv --keyserver keys.gnupg.net --recv-key D6FB2970 
   sudo apt update -y
   sudo apt-get install librealsense-dev -y
   ```
1. Get the DCM source code by cloning the [camera-manager](https://github.com/Dronecode/camera-manager) repo (or your fork):
   ```sh
   git clone https://github.com/Dronecode/camera-manager.git
   cd camera-manager
   git submodule update --init --recursive
   ```
1. Configure DCM with the normal Aero settings:
   ```
   ./autogen.sh && ./configure --enable-aero --enable-realsense --enable-mavlink --enable-avahi
   ```
1. Build DCM for Aero:
   ```
   make
   ```

The *dcm* binary and *dronecode-camera-manager.service* files are generated in the root of the DCM source tree.

## Deploy DCM to Aero

To deploy DCM to Aero:

1. Copy the *dcm* binary using *scp*:
   ```
   scp dcm uname@ip-addr:/usr/bin/
   ```
   where:
   * `ip-addr` is IP address of Aero on the network (check using *ifconfig*)
   * `uname` is `root` (Yocto) or `<user-defined>` (Ubuntu)

1. Reboot Aero (Aero starts DCM on boot)

> **Note** The Aero configuration file ([aero.conf](https://github.com/Dronecode/camera-manager/blob/master/samples/config/aero.conf)) and [autostart file](../guide/autostart.md) (*dronecode-camera-manager.service*) typically do not change or need to be updated. If required, you could do so using the following commands:
  ```
  scp samples/config/aero.conf uname@ip-addr:/etc/dcm/main.conf
  scp dronecode-camera-manager.service uname@ip-addr:/lib/systemd/system/dronecode-camera-manager.service
  ```


## Verify Installation

1. Make sure DCM is running:
   ```sh
   systemctl status dcm
   ```
1. Run the [Sanity Tests](../test/sanity_tests.md) to verify that DCM is working correctly.
