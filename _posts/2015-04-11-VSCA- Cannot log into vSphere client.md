---
layout: post
title: VCSA - Cannot log into vSphere Client 
---

At work we are in the middle of upgrading to VMware 6.0 stumbled across a unique error. When trying to log in the vSphere thick client I get an error

"Windows session credentials cannot be used to log into this server.

Enter a user name and password."

VSCA vSphere Client Login Error



After some digging I found out it is a known bug (KB 2119332) when using the vCenter appliance and an external psc. According to the kb it is fixed in update 1 but I still was experiencing on 6.0 Update 1.

Thankfully the work around is uncheck use Windows session credentials and type your username (username@domain.local) and your password then boom you are in.