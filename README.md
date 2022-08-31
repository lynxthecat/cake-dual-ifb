# cake-dual-ifb
Set up CAKE using dual upload and download IFBs:

![image](https://user-images.githubusercontent.com/10721999/186377537-70b37cdf-1d41-419c-9e97-6facfef3e52e.png)

That is, create 'ifb-ul' by mirroring the ingress from br-lan and br-guest and create 'ifb-dl' by mirroring the egress from br-lan and br-guest. And skip out the LAN-LAN traffic. Then apply CAKE on 'ifb-ul' and 'ifb-dl'. 

This permits CAKE to properly function despite complex setups like use of VPN pbr and a guest LAN.

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
   
 ## DSCP restoration from conntracks
 
 My preferred way to handle DSCPs is to set them locally at the LAN client level and have the router:
 
- firstly, determine the DSCPs associated with upload packets; and
- secondly, apply those DSCPs to corresponding download packets associated with the same connection.
 
 This is achieved by setting the DSCPs to the 'conntrack marks' on upload, using nftables, and restoring those DSCPs from the 'conntrack marks' on download, using tc-ctinfo.
 
 This additionally requires package:

- kmod-sched-ctinfo

 To set this up:
 
  ```bash
      opkg update; opkg install kmod-sched-ctinfo
      cd /etc/nftables.d/
      wget https://raw.githubusercontent.com/lynxthecat/cake-wg-pbr/main/cake-dual-ifb.nft
      /etc/init.d/firewall restart
   ```
 
 Verify correct operation using tcpdump:
 
   ```bash
      opkg update; opkg install tcpdump
      # First check DSCPs correctly set by your LAN client on upload
      tcpdump -i ifb-ul -vv udp
      # Second check corresponding DSCPs are getting set by router on download
      tcpdump -i ifb-dl -vv udp
   ``` 
   
If using Microsoft Windows, DSCPs can be set at the application level by creating the registry key 'QoS' (it not present) as in:

HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Tcpip\QoS\

And then creating the string "Do not use NLA" inside the QoS key with value "1"

![image](https://user-images.githubusercontent.com/10721999/187535155-d4fd286b-9f20-40ce-8ff9-98ed36591721.png)

And then by creating appropriate QoS policies in the Local Group Policy Editor:

![image](https://user-images.githubusercontent.com/10721999/187462933-78ebdee9-8121-4cad-8547-25b1a397572f.png)
