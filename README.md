# cake-dual-ifb
Set up CAKE using ul and dl ifbs:

![image](https://user-images.githubusercontent.com/10721999/186377537-70b37cdf-1d41-419c-9e97-6facfef3e52e.png)

## Required packages

This script requires at least the following packages:

- **tc-tiny**
- **kmod-ifb**
- **kmod-sched**
- **kmod-sched-core**
- **kmod-sched-cake**

## Installation on OpenWrt

To install:

  ```bash
   opkg update; opkg install tc-tiny kmod-ifb kmod-sched kmod-sched-core kmod-sched-cake
   cd /etc/init.d/
   wget https://raw.githubusercontent.com/lynxthecat/cake-dual-ifb/main/cake-dual-ifb
   chmod +x ./cake-dual-ifb
   cd /etc/hotplud.g/iface/
   wget https://raw.githubusercontent.com/lynxthecat/cake-wg-pbr/main/11-cake-dual-ifb
   chmod +x ./11-cake-dual-ifb
   ```
   
   Verify interfaces (e.g. replace or delete br-lan / br-guest lines as required)
