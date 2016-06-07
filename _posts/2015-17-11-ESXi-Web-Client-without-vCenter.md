---
layout: post
title: ESXi Web Client without vCenter 
---

While reading through the twitterverse I found an amazing VMware fling, the Embedded Host Client Fling. Flings are pet projects from VMware engineers and this one is my favorite so far. Some of the things you can do with this client

- VM operations (Power on, off, reset, suspend, etc).
- Creating a new VM, from scratch or from OVF/OVA (limited OVA support)
- Configuring NTP on a host
- Displaying summaries, events, tasks and notifications/alerts
- Providing a console to VMs
- Configuring host networking
- Configuring host advanced settings
- Configuring host services
 

So how do you install this wonderful tool?

1. SSH into your host
2. Run the command 
`esxcli software vib install -v https://labs.vmware.com/flings/esxi-embedded-host-client`  

![Alt text](/assets/img/FlingInstall.jpg "Fling Install")

3. Now go to https://ipaddress/ui and your set  
![Alt text](/assets/img/EmbeddedHostFling.jpg "Embedded Host Fling")

Hope you enjoyed this fling. Make sure to check out the [VMware Labs](https://labs.vmware.com/) for the other amazing flings.
