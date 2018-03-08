# Deploying CSD to a Drone Platform

> **Caution** This page is unreviewed/under construction.

This topic explains how to set up a companion computer with CSD dependencies and deploy CSD to it from a Ubuntu development computer.

> **Note** Companion computers are relatively resource-constrained, so it makes sense to develop on Ubuntu and only deploy the required binary files.


## Enable AutoStart

[Autostart CSD > Auto-start on Other Platforms](../guide/autostart.md#enable) explains how to set up Linux to automatically restart CSD when the platform is rebooted.


## CSD Binary Dependencies

TBD.
<!-- What are they -->
<!-- How are they deployed? Is it a question of creating an "image"? -->


## Deploy CSD to Drone


The following files must be present on the target platform (in addition to the CSD dependencies):

File | Source File | Destination File
--- | ---
CSD binary | **/csd** (generated in root) | **/usr/bin/csd**
[Auto-start definition](../guide/autostart.md) | **/csd.system** (generated in root) | **/lib/system/system/csd.system**
[CSD configuration file](../guide/configuration_file.md) for platform | [/samples/files/](https://github.com/intel/camera-streaming-daemon/tree/master/samples/files)`<your_system>`.conf | **/etc/csd/main.conf**

To deploy CSD to a companion computer using *scp*:

1. Copy the *csd* binary using the following command (from the CSD root):
   ```
   scp csd uname@ip-addr:/usr/bin/
   ```
   where:
   * `ip-addr` is IP address of the drone on the network (check using *ifconfig*) and `uname` is a user name with write privileges (system dependent).
   
   > **Note** The configuration and autostart files typically do not change and rarely need to be updated. If required you can use the same approach as above to copy them. 

1. Reboot Computer (to restart CSD on boot)


## Verify Deployment

1. Make sure CSD is running (on the drone):
   ```sh
   systemctl status csd
   ```
1. Run the [Sanity Tests](../test/sanity_tests.md) to verify that CSD is working correctly.