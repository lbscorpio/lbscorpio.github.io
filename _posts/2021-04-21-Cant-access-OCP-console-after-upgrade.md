---
title:  Can't access OCP console after upgrade
date: 2021-04-21 11:32:21
categories:
  - Troubleshoot
tags:
  - linux
  - ocp
  - kvm
---

I did OCP upgrade from v4.7.3 to v4.7.6 last night, but failed to access OCP console this morning.

## Environment

- Red Hat OpenShift 4.7
- All master and worker nodes configured with KVM VMs

## Issue

- Can't access OCP console UI from web browser

## Resolution

- Clean some big files on `/var`, I chose to clean YUM cache data for temporary.

      # yum clean all

- After `/var` have enough free space, then can resume the VMs.

      # virsh resume <VM_name>

## Diagnostic Steps

- Tried to use command `oc login` but failed.

- Checked VMs status, found that only all master nodes in `paused` state.

      # virsh list
       Id    Name                           State
      ----------------------------------------------------
       4     master1                        paused
       6     master2                        paused
       8     master3                        paused
       15    buildserver                    running

- Tried to resume and reboot the VMs but not working.

- After realizing that `/var` saved the VM images, found it was full.

      # df -h  /var
      Filesystem            Size  Used Avail Use% Mounted on
      /dev/mapper/rhel-var  200G  200G   20K 100% /var

- Checking the Top-10 files size on `/var`, found that VM images and YUM cache files.

      # find /var -type f -print0 | xargs -0 du -h | sort -rh | head -n 10
      80G	/var/lib/libvirt/images/master1.qcow2
      76G	/var/lib/libvirt/images/master2.qcow2
      30G	/var/lib/libvirt/images/master3.qcow2
      3.3G	/var/cache/yum/x86_64/7Server/ftp3/gen/other.xml.sqlite
      3.2G	/var/cache/yum/x86_64/7Server/ftp3/gen/other.xml
      760M	/var/cache/yum/x86_64/7Server/ftp3/fd8840af9aa89e94f8e4570aede2165c9b65aabe-other.xml.gz
      437M	/var/cache/yum/x86_64/7Server/ftp3/gen/primary.xml.sqlite
      365M	/var/cache/yum/x86_64/7Server/ftp3/gen/primary.xml
      244M	/var/lib/rpm/Packages
      119M	/var/lib/containers/storage/overlay/
      
- So cleaned YUM cache to release more space.

      # yum clean all
      # df -h  /var
      Filesystem            Size  Used Avail Use% Mounted on
      /dev/mapper/rhel-var  200G  192G  8.6G  96% /var

## Reference

<a href="https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/virtualization_deployment_and_administration_guide/sect-starting_suspending_resuming_saving_and_restoring_a_guest_virtual_machine-starting_a_defined_domain" target="_blank">virsh command usage</a>  from Red Hat documentation

> **Copyright Notice**: Original work by this blog’s author, licensed under <a href="https://creativecommons.org/licenses/by-nc-nd/4.0/" target="_blank">CC BY-NC-ND 4.0</a>. When reposting, please include a link to the original source and this copyright notice.