# Auto-start DCM on Boot

The *Dronecode Camera Manager* (DCM) [build process](../getting_started/building_installation.md) generates a file **dronecode-camera-manager.service** that can be used to help auto-start DCM on boot on platforms that support the [systemd](https://en.wikipedia.org/wiki/Systemd) service. 

## Auto-start on Intel Aero

DCM is already integrated with *systemd* on Intel Aero (for both Yocto and Ubuntu OS).

The only work required is to ensure that **dronecode-camera-manager.service** is placed in **/lib/system/system**.


## Auto-start on Other Platforms {#enable}

The following steps can be used to integrate DCM with *systemd* on a new platform. 

1. Enable *systemd*, following the OS-specific instructions for your platform. 
1. Put the **dronecode-camera-manager.service** file in **/lib/system/system** directory (the file is generated in the root of the DCM repository when you [build the Camera Manager](../getting_started/building_installation.md)).
1. Reload *systemd* daemon
   ```sh
   systemctl daemon-reload
   ```
1. Enable the service to start DCM automatically on subsequent boot
   ```sh
   systemctl enable dronecode-camera-manager.service
   ```

   > **Note** You can also start the service just for the current session:
      ```sh
      systemctl start dronecode-camera-manager.service
      ```
