---
title:  How to install VNC Server on RHEL 7.x
date: 2020-12-31 15:35:20
categories:
  - Procedure
tags:
  - linux
  - vnc
---

Some testing might use GUI to operate tasks, or some configuration wizard need GUI to proceed, but we often remote access the servers. 

This article will guide you how to install VNC Server and configure it for RHEL 7.x environment.

## Prerequisites

- A linux server which installed Environment Groups "**Server with GUI**"
- The linux default run level is graphical, firewall is disabled.

## Steps to install/configure VNC Server

1. Verify the installed group list

       # yum grouplist
       Installed Environment Groups:
          Server with GUI         --> Here identify this OS installed GUI
       Available Environment Groups:
          Minimal Install
          Infrastructure Server
          File and Print Server
          Cinnamon Desktop
          MATE Desktop
          Basic Web Server
          Virtualization Host
 
   Install GUI environment if need by following commands:
   
       # yum groupinstall -y  "Server with GUI"
       # systemctl get-default
       # systemctl set-default graphical.target
       # reboot
 
2. Install VNC Server via YUM

       # yum install -y tigervnc-server
       
3. Configure VNC Server

       # cp /lib/systemd/system/vncserver@.service /etc/systemd/system/.
       # vi /etc/systemd/system/vncserver@.service
       Replace <USER> with the actual user name(e.g. root)
       
       # systemctl daemon-reload
       # vncpasswd                 // Set the password used to access VNC desktops

4. Start VNC Server
  
       # systemctl start vncserver@:1
       # systemctl enable vncserver@:1       // Optional
       # systemctl status vncserver@:1
       
## Reference

- MacOS install a VNC viewer to access the VNC Server

      # brew install --cask tigervnc-viewer

> **Copyright Notice**: Original work by this blog’s author, licensed under <a href="https://creativecommons.org/licenses/by-nc-nd/4.0/" target="_blank">CC BY-NC-ND 4.0</a>. When reposting, please include a link to the original source and this copyright notice.