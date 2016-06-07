---
layout: post
title: vSphere 6 Platform Services Controller move from embedded to external 
---

At work we are in the middle of upgrading from vSphere 5.0 to 6.0 and ran into a into a interesting design issue. We started by deploying the vCenter 6 appliance in our DR using the embedded PSC. Everything was fine until we decided we wanted to install a vCenter server in our production site using the same SSO domain.

We quickly realized having two vCenter servers with embedded psc's with the same SSO was no longer a supported configuration.

PSCdesign

We could have scrapped our current vCenter as it was still not in production but after some digging aka Google we found that moving from an embedded psc to an external was now supported in 6.0 update 1 using the cmsso-util command.

Reconfigure an Embedded Deployment to an External Deployment

Before starting these steps make sure to deploy a new vCenter appliance selecting external PSC at install and join it to your existing SSO domain

SSH into your existing vCenter
At the prompt run the commands
Command> shell.set –enabled True
Command> shell
SSH session ino vCenter 6 appliance


Now on to the reconfiguring the appliance

Run the command:

cmsso-util reconfigure –repoint-psc <FQDN-of-External-PSC> –username <SSO-DomainAdmin> –domain-name <SSO-Domain> –passwd <SSO-DomainAdmin-Password>

In my case the command is:

cmsso-util reconfigure --repoint-psc "HL-PSC.homelab.local" --username "administrator" --domain-name "vsphere.local" --passwd "Lifeboat0@"

PSCreconfigure1

If you typed the command right the it will go through the process of re-configuring, it took about 5 minutes for me.

Once complete run the command to verify what PSC your vCenter is pointed at

/usr/lib/vmware-vmafd/bin/vmafd-cli get-ls-location --server-name localhost

PSCreconfigure2


As a double check if you close your SSH session then reconnect the login screen will state vCenter Server with external Platform Services Controller

Confirming the vCenter has a external PSC listed


Success! We have moved to an external psc and can move onto deploying an additional vCenter appliance in linked mode.
