# Deploying DSM to a Drone Platform

> **Caution** This page is unreviewed/under construction.

This topic explains how to set up a companion computer with DSM dependencies and deploy DSM to it from a Ubuntu development computer.

> **Note** Companion computers are relatively resource-constrained, so it makes sense to develop on Ubuntu and only deploy the required binary files.


## Enable AutoStart

[Autostart DSM > Auto-start on Other Platforms](../guide/autostart.md#enable) explains how to set up Linux to automatically restart DSM when the platform is rebooted.


## DSM Binary Dependencies

TBD.
<!-- What are they -->
<!-- How are they deployed? Is it a question of creating an "image"? -->


## Deploy DCM to Drone

The following files must be present on the target platform (in addition to the DCM dependencies):

File | Source File | Destination File
--- | ---
DCM binary | **/dcm** (generated in root) | **/usr/bin/dcm**
[Auto-start definition](../guide/autostart.md) | **/dronecode-camera-manager.service** (generated in root) | **/lib/system/system/dronecode-camera-manager.service**
[DCM configuration file](../guide/configuration_file.md) for platform | [/samples/config/](https://github.com/Dronecode/camera-manager/tree/master/samples/config)`<your_system>`.conf | **/etc/dcm/main.conf**

To deploy DCM to a companion computer using *scp*:

1. Copy the *dcm* binary using the following command (from the DCM root):
   ```
   scp dcm uname@ip-addr:/usr/bin/
   ```
   where:
   * `ip-addr` is IP address of the drone on the network (check using *ifconfig*) and `uname` is a user name with write privileges (system dependent).

   > **Note** The configuration and autostart files typically do not change and rarely need to be updated. If required you can use the same approach as above to copy them. 

1. Reboot Computer (DCM now restarts on boot)


## Verify Deployment

1. Make sure DCM is running (on the drone):
   ```sh
   systemctl status dcm
   ```
1. Run the [Sanity Tests](../test/sanity_tests.md) to verify that DCM is working correctly.