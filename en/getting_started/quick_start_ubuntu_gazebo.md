# Quickstart Guide — Gazebo Camera

> **Warning** These instructions are in progress. At time of writing build without `--enable-mavlink` and connect on `udp port 5600` (RTSP publishing not supported). To be reviewed 9 March 2018. 

This Quickstart shows how to build CSD on Ubuntu LTS 16.04 (with support for MAVLink and Avahi) and connect to a camera running within a Gazebo/PX4 vehicle simulation. 


## Install Gazebo/SITL

First install Gazebo/SITL (CSD requires the dependency is installed before building):

1. Install [Gazebo and PX4 SITL](#gazebo_deps) using the scripts in the *PX4 Developer Guide*: [Development Environment on Linux > jMAVSim/Gazebo Simulation](https://dev.px4.io/en/setup/dev_env_linux.html#jmavsimgazebo-simulation).

1. Edit the *typhoon_h480* configuration file in the PX4 source (**/Firmware/posix-configs/SITL/init/ekf2/typhoon_h480**).
   * Replace:
     ```
     mavlink start -x -u 14556 -r 4000000
     ```
   * With:
     ```
     mavlink start -x -u 14556 -r 4000000 -f
     mavlink start -x -u 24550 -f -o 34550
     ```
     

## Set up CSD

To set up CSD for use with Gazebo:

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
1. Get the CSD source code by cloning the the [camera-streaming-daemon](https://github.com/intel/camera-streaming-daemon) repo (or your fork):
   ```sh
   git clone https://github.com/intel/camera-streaming-daemon.git
   cd camera-streaming-daemon
   git submodule update --init --recursive
   ```
1. Configure CSD as shown:
```
./autogen.sh && ./configure --enable-mavlink --enable-avahi --enable-gazebo
```

## Run CSD/Gazebo

To run CSD on Ubuntu along with Gazebo PX4-SITL:
 
1. Open *QGroundControl* build from master branch.
1. Start PX4-SITL gazebo from the PX4 **/Firmware** folder:
   ```
   make posix gazebo_typhoon_h480
   ```
1. Serve the sample [Camera Definition Files](../guide/camera_definition_file.md):
   * Open a new terminal to [/samples/def](https://github.com/intel/camera-streaming-daemon/tree/master/samples/def)
   * Enter the following command to start the server on the default port (8000):
     ```
     python -m SimpleHTTPServer
     ```
1. Run CSD, specifying the Gazebo [configuration file](../getting_started/building_installation.md#configuration-file-runtime):
   ```sh
   ./csd -c samples/files/gazebo.conf
   ```

Then run the [Sanity Tests](../test/sanity_tests.md) to verify that CSD is working correctly.