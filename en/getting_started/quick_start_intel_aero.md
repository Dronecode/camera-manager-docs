# Quickstart Guide — Intel Aero

> **Tip** CSD is pre-integrated into *Intel Aero* images (1.6.1 and later). These instructions are only needed when developing and testing newer versions of CSD.

This guide provides turnkey instructions for building CSD on Ubuntu LTS 16.04 (with support for the RealSense 3D Camera, Aero bottom facing camera, MAVLink and Avahi) and then deploying it to *Intel Aero*.

## Build CSD

To build CSD:
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
1. Get the CSD source code by cloning the the [camera-streaming-daemon](https://github.com/intel/camera-streaming-daemon) repo (or your fork):
   ```sh
   git clone https://github.com/intel/camera-streaming-daemon.git
   cd camera-streaming-daemon
   git submodule update --init --recursive
   ```
1. Configure CSD with the normal Aero settings:
   ```
   ./autogen.sh && ./configure --enable-aero --enable-realsense --enable-mavlink --enable-avahi
   ```
1. Build CSD for Aero:
   ```
   make
   ```

The *csd* binary and *csd.system* files are generated in the root of the CSD source tree.

## Deploy CSD to Aero

To deploy CSD to Aero:

1. Copy the *csd* binary using *scp*:
   ```
   scp csd uname@ip-addr:/usr/bin/
   ```
   where:
   * `ip-addr` is IP address of Aero on the network (check using *ifconfig*)
   * `uname` is `root` (Yocto) or `<user-defined>` (Ubuntu)

1. Reboot Aero (Aero starts CSD on boot)

> **Note** The Aero configuration file ([aero.conf](https://github.com/intel/camera-streaming-daemon/blob/master/samples/files/aero.conf)) and [autostart file](../guide/autostart.md) (*csd.service*) typically do not change or need to be updated. If required, you could do so using the following commands:
  ```
  scp samples/files/aero.conf uname@ip-addr:/etc/csd/main.conf
  scp csd.system uname@ip-addr:/lib/system/system/csd.system
  ```
  


## Verify Installation

1. Make sure CSD is running:
   ```sh
   systemctl status csd
   ```
1. Run the [Sanity Tests](../test/sanity_tests.md) to verify that CSD is working correctly.
