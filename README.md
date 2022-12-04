# cake-dual-ifb

cake-dual-ifb sets up dual IFBs and applies CAKE on them using nftables and appropriate tc calls. DSCPs for use with diffserv4 can be set by the router and/or by LAN clients and conntrack restore is used to restore DSCPs on download based on upload DSCPs. 

 ### CAKE interfaces

Set up CAKE using dual upload and download IFBs:

![image](https://user-images.githubusercontent.com/10721999/188687637-630aa3c5-ee42-4139-be4d-283e1515b54d.png)

That is, create 'ifb-ul' by mirroring: 

- a) appropriately selected ingress from br-lan and br-guest; and 

- b) appropriately selected egress from wan. 

And create 'ifb-dl' by mirroring: 

- a) appropriately selected egress from br-lan and br-guest; and 
- b) appropriately selected ingress from wan. 

Appropriate selection is achieved through fwmarks using nftables to isolate and avoid duplication of the relevant flows including unencrytyped WireGuard traffic and OpenWrt->WAN and WAN->OpenWrt traffic, and skip out the LAN-LAN traffic. 

Then apply CAKE on 'ifb-ul' and 'ifb-dl'. 

This permits CAKE to properly function despite complex setups like use of VPN pbr and a guest LAN.

### DSCPs
 
 cake-dual-ifb is designed to handle DSCPs as follows:
 
- firstly, determine the DSCPs associated with upload packets; and
- secondly, apply those DSCPs to corresponding download packets associated with the same connection.
 
This is achieved by: 

- firstly, using nftables to set the DSCPs to the 'conntrack marks' on upload; and 
- secondly, using tc-ctinfo to restore those stored DSCPs from the 'conntrack marks' on download.
 
This facilitates setting DSCPs not only by the router itself but also by LAN clients. As compared to relying upon port ranges and the like, setting DSCPs in LAN clients offers a robust way to set DSCPs at the application level. 

### nftables

nftables is leveraged to apply fwmarks, DSCPs, and save DSCPs to conntracks as appropriate. 

To understand the logic adopted in cake-dual-ifb.nft consider this helpful diagram:

![image](https://user-images.githubusercontent.com/10721999/188932157-881bd4ef-e1ab-46d7-bd1b-966e78f00429.png)

Source: https://wiki.nftables.org/wiki-nftables/index.php/Netfilter_hooks


## Required packages

This script requires at least the following packages:

- **tc-tiny**
- **kmod-ifb**
- **kmod-sched**
- **kmod-sched-core**
- **kmod-sched-cake**
- **kmod-sched-ctinfo**
- **kmod-nft-bridge**

## Installation on OpenWrt

To install:

- place cake-dual-ifb in /etc/init.d
- chmod +x cake-dual-ifb
- place 11-cake-dual-ifb in /etc/hotplug.d/iface/
- chmod +x 11-cake-dual-ifb
- place cake-dual-ifb.nft in /usr/share/nftables.d/ruleset-post/
- edit cake-dual-ifb.nft for your use case 
- verify interfaces (e.g. replace or delete br-lan / br-guest lines as required)
   
### To setup DSCP setting by the router ###

- amend cake-dual-ifb.nft as appropriate

This can optionally override anything set by the LAN clients. 

### Setting DSCPs in Microsoft Windows Based LAN Clients ###

If using Microsoft Windows, DSCPs can be set at the application level by creating the registry key 'QoS' (it not present) as in:

HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Tcpip\QoS\

And then creating the string "Do not use NLA" inside the QoS key with value "1"

![image](https://user-images.githubusercontent.com/10721999/187535155-d4fd286b-9f20-40ce-8ff9-98ed36591721.png)

And then by creating appropriate QoS policies in the Local Group Policy Editor:

![image](https://user-images.githubusercontent.com/10721999/187747512-4c608e11-92a9-4484-b07f-3695baa98b85.png)

### Verifying Correct Operation and DSCP Handling ###

 Verify correct operation and DSCP handling using tcpdump:
 
   ```bash
      opkg update; opkg install tcpdump
      # First check correct flows and DSCPs correctly set by your LAN client on upload
      tcpdump -i ifb-ul -vv udp
      # Second check correct flows and corresponding DSCPs are getting set by router on download
      tcpdump -i ifb-dl -vv udp
   ``` 
