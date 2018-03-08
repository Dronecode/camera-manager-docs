# Auto-start CSD on Boot

The CSD [build process](../getting_started/building_installation.md) generates a file **csd.service** that can be used to help auto-start CSD on boot on platforms that support the [systemd](https://en.wikipedia.org/wiki/Systemd) service. 

## Auto-start on Intel Aero

CSD is already integrated with *systemd* on Intel Aero (for both Yocto and Ubuntu OS).

The only work required is to ensure that **csd.service** is placed in **/lib/system/system**.


<span id="enable"></span>
## Auto-start on Other Platforms

The following steps can be used to integrate CSD with *systemd* on a new platform. 

1. Enable *systemd*, following the OS-specific instructions for your platform. 
1. Put the **csd.service** file in **/lib/system/system** directory (the file is generated in the root of the CSD repository when you [build CSD](../getting_started/building_installation.md)).
1. Reload *systemd* daemon
   ```sh
   systemctl daemon-reload
   ```
1. Enable the service to start CSD automatically on subsequent boot
   ```sh
   systemctl enable csd.service
   ```

   > **Note** You can also start the service just for the current session:
      ```sh
      systemctl start csd.service
      ```
