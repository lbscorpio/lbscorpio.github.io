---
title:  How to configure PXE Boot for creating RHCOS machine
date: 2021-09-02 11:43:18
categories:
  - Setup
tags:
  - linux
  - coreos
  - rhcos
  - pxe
  - tftp
---

I tried to install CoreOS on physical server x3650 M5 from its ISO but failed, so have to try the PXE boot.

This article will guide you how to configure PXE boot for RHEL 7.x/8.x environment.

## Prerequisites

- Have configured an HTTP/DHCP server
- A TFTP server capable of servicing TFTP requests made by systems attempting to perform a PXE Boot.
- The files that the server performing a PXE Boot are going to load, which can be obtained from the <a href="https://mirror.openshift.com/pub/openshift-v4/dependencies/rhcos/latest/latest/" target="_blank">RHCOS image mirror</a> page.
  
  ```
  kernel: rhcos-<version>-live-kernel-<architecture>
  initramfs: rhcos-<version>-live-initramfs.<architecture>.img
  rootfs: rhcos-<version>-live-rootfs.<architecture>.img
  ```

## DHCP Configuration

- Set up `/etc/dhcp/dhcpd.conf`, to add the PXE server n.n.n.n.

  ```
  subnet 9.115.251.0  netmask 255.255.255.0 {
        host mc181 { hardware ethernet 52:54:00:2a:37:96; fixed-address 9.115.x.x; option host-name mc181; next-server n.n.n.n; filename "pxelinux.0"; }
  ```

- Restart DHCP service

  ```
  # systemctl restart dhcpd
  ```

## TFTP Configuration

- If you have not already installed a TFTP server, you should do so: 

  ```
  # yum install -y tftp-server
  ```

- Edit the `/etc/xinetd.d/tftp` file to set the disable parameter to no: 

  ```
          disable                 = no
  ```

- Restart the inetd server

  ```
  # systemctl restart xinetd
  ```

> **NOTE**: RHEL 8 can skip this step, but need to start the `tftp` service.

## PXELinux Configuration

- If you have not already installed syslinux, you should do so:

  ```
  yum install -y syslinux
  ```
  
- Copy the PXELinux binary, the menu system and the multiboot com32 module from the syslinux package to your tftp server root folder.

  ```
  # cp /usr/share/syslinux/pxelinux.0 /var/lib/tftpboot
  # cp /usr/share/syslinux/vesamenu.c32 /var/lib/tftpboot
  # cp /usr/share/syslinux/mboot.c32 /var/lib/tftpboot
  ```
  > **NOTE**: For RHEL 8, you need to copy files `ldlinux.c32`, `libcom32.c32`, `libutil.c32` too.

- Create a PXE Menu configuration file `/var/lib/tftpboot/pxelinux.cfg/pxe.conf` with the following configuration:

  ```
  MENU TITLE  PXE Server
  NOESCAPE 1
  ALLOWOPTIONS 1
  PROMPT 0
  menu width 80
  menu rows 14
  MENU TABMSGROW 24
  MENU MARGIN 10
  menu color border               30;44      #ffffffff #00000000 std
  ```
  
- Create a PXE configuration file `/var/lib/tftpboot/pxelinux.cfg/default` for RHCOS PXE boot.

  ```
  default vesamenu.c32
  timeout 600
  ontimeout bootstrap.HTTP
  menu include pxelinux.cfg/pxe.conf

  label bootstrap.HTTP
  kernel rhcos-<version>-live-kernel-<architecture>
  append initrd=rhcos-<version>-live-initramfs.<architecture>.img,rhcos-<version>-live-rootfs.<architecture>.img coreos.inst.install_dev=/dev/sda coreos.inst.ignition_url=http://<http_server>:8090/bootstrap.ign
  
  label master.HTTP
  kernel rhcos-<version>-live-kernel-<architecture>
  append initrd=rhcos-<version>-live-initramfs.<architecture>.img,rhcos-<version>-live-rootfs.<architecture>.img coreos.inst.install_dev=/dev/sda coreos.inst.ignition_url=http://<http_server>:8090/master.ign

  label worker.HTTP
  kernel rhcos-<version>-live-kernel-<architecture>
  append initrd=rhcos-<version>-live-initramfs.<architecture>.img,rhcos-<version>-live-rootfs.<architecture>.img coreos.inst.install_dev=/dev/sda coreos.inst.ignition_url=http://<http_server>:8090/worker.ign
  ```

## Boot the target server to install CoreOS

- In my case, booting up the x server x3650 M5, hit `F1` to enter BIOS Setup, then choose PXE boot.

## References

<a href="https://docs.oracle.com/html/E50247_08/vmiug-install-pxe.html" target="_blank">Oracle VM installation from PXE</a>  
<a href="https://docs.fedoraproject.org/en-US/fedora-coreos/bare-metal/" target="_blank">Installing CoreOS on Bare Metal</a>  
<a href="https://docs.openshift.com/container-platform/4.8/installing/installing_bare_metal/installing-bare-metal.html#installation-user-infra-machines-pxe_installing-bare-metal" target="_blank">Installing RHCOS by using PXE or iPXE booting</a>  

> **Copyright Notice**: Original work by this blog’s author, licensed under <a href="https://creativecommons.org/licenses/by-nc-nd/4.0/" target="_blank">CC BY-NC-ND 4.0</a>. When reposting, please include a link to the original source and this copyright notice.