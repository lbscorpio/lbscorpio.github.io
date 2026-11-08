---
title:  How to change network device naming from eth to eno on Linux
date: 2021-01-06 13:18:20
categories:
  - Procedure
tags:
  - linux
  - eth
  - eno
---

There are different naming schemes (e.g. eno1, ens1, enp2s0 and so on) for interface naming on Linux world. RHEL 8.x doesn't support ethX naming during upgrading from RHEL 7.x, so you need to change it before upgrading from RHEL 7.x if have.

## Prerequisites

- An existing RHEL 7.x with network interface naming ethX.
- The OS have installed the package **biosdevname**.

## Procedures

1. Open the /etc/default/grub file and find the GRUB_CMDLINE_LINUX variable, add "biosdevname=1" as kernel parameter values.

       Example:
       # cat /etc/default/grub
       GRUB_TIMEOUT=5
       GRUB_DISTRIBUTOR="$(sed 's, release .*$,,g' /etc/system-release)"
       GRUB_DEFAULT=saved
       GRUB_DISABLE_SUBMENU=true
       GRUB_TERMINAL_OUTPUT="console"
       GRUB_CMDLINE_LINUX="crashkernel=auto rd.lvm.lv=vg_c550f3dxn04/lv_root rd.lvm.lv=vg_c550f3dxn04/lv_swap biosdevname=1 rhgb quiet"
       GRUB_DISABLE_RECOVERY="true"
       
   > If have "net.ifnames=0" on the variable GRUB_CMDLINE_LINUX, please remove it too.

2. Rebuild the /boot/grub2/grub.cfg file

       # grub2-mkconfig -o /boot/grub2/grub.cfg
       
3. View the current device name, for example eth0 here.
 
       # nmcli connection show
       NAME         UUID                                  TYPE        DEVICE
       System eth0  5fb06bd0-0bb0-7ffb-45f1-d6edd65f3e03  ethernet    eth0
       eth1         e4290b04-3dc8-4759-ae0d-35dc1e38fabd  ethernet    --
       ib0          ed2118ea-8117-4935-a6f2-746aa9e5f490  infiniband  --
       usb0         b7becf20-7a79-4209-899d-33d8a39f47e5  ethernet    --
       
4. Modify the device name to eno1, and reboot the system.

       # nmcli connection modify "System eth0" connection.interface-name eno1
       # reboot

5. After the OS bootup, verify the device name.
 
       # nmcli connection show
       # ifconfig | grep flags
       
       Examples:
       # nmcli connection show
       NAME         UUID                                  TYPE        DEVICE
       System eth0  5fb06bd0-0bb0-7ffb-45f1-d6edd65f3e03  ethernet    eno1
       eth1         e4290b04-3dc8-4759-ae0d-35dc1e38fabd  ethernet    --
       ib0          ed2118ea-8117-4935-a6f2-746aa9e5f490  infiniband  --
       usb0         b7becf20-7a79-4209-899d-33d8a39f47e5  ethernet    --
       
       # ifconfig | grep flags
       eno1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
       eno2: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
       enp0s29f1u2: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
       lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536

## References

<a href="https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html-single/networking_guide/index#ch-Consistent_Network_Device_Naming" target="_blank">Consistent Network Device Naming from Red Hat</a>  
<a href="https://access.redhat.com/solutions/2435891" target="_blank">Is it safe to set net.ifnames=0 in RHEL7 and RHEL8?</a>  

If RHEL 7.x used ethX, you can see below risk on leapp report while upgrading to RHEL 8.x:

    Risk Factor: high (inhibitor)
    Title: Unsupported network configuration
    Summary: Detected multiple network interfaces using unstable kernel names (e.g. eth0, eth1). Upgrade process can not continue because stability of names can not be guaranteed. Please read the article at https://access.redhat.com/solutions/4067471 for more information.

> **Copyright Notice**: Original work by this blog’s author, licensed under <a href="https://creativecommons.org/licenses/by-nc-nd/4.0/" target="_blank">CC BY-NC-ND 4.0</a>. When reposting, please include a link to the original source and this copyright notice.