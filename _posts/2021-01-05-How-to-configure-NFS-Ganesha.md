---
title:  How to configure NFS-Ganesha without a CES IP
date: 2021-01-05 12:03:20
categories:
  - Setup
tags:
  - ptf
  - nfs
  - gpfs
  - ganesha
---

Sometime we are not convenient to configure a CES IP for protocol node, due to lack of additional IP resource or no physical links on additional ethernet ports(especially remote machines).

This article is to configure NFS-Ganesha without a CES IP, it should be fine for most NFS relevant testing.

## Prerequisites

- An existing GPFS cluster, and have filesystem mounted.
- The NFS relevant packages were installed, which under /usr/lpp/mmfs/<5.x.y.z>/ganesha_rpms.

## Steps to configure NFS-Ganesha

1. Configure the CES shared root if not yet

       # mmshutdown -a
       # mmchconfig cesSharedRoot=<shared_root_path>
       # mmstartup -a
       
       Example:
       # mmchconfig cesSharedRoot=/gpfs/fs1

2. Enable a protocol node

       # mmgetstate -a           // Ensure that all nodes state are "active"
       # mmchnode --ces-enable -N Node1
       
3. Create and link a fileset for NFS

       # mmlsmount all           // Ensure the existing filesystem mounted
       # mmcrfileset <FileSystem> <fileset> --inode-space new
       # mmlinkfileset <FileSystem> <fileset> [-J JunctionPath]
       
       Examples:
       # mmcrfileset fs1 nfs_fileset --inode-space=new
       # mmlinkfileset fs1 nfs_fileset -J /gpfs/fs1/home/nfstest
      > In above example, you need to create the directory /gpfs/fs1/home/ firstly if not exist.

4. Start NFS service and set up the authentication method
 
       # mmces service enable NFS
       # systemctl daemon-reload
       # mmuserauth service create --data-access-method file --type userdefined

5. Create an NFS export
   
       # mmnfs export add <exported_path>
       # mmnfs export list
       
       Examples:
       # mmnfs export add /gpfs/fs1/home/nfstest/ -c "*(Access_Type=RW,Squash=no_root_squash)"
       # mmnfs export list -n /gpfs/fs1/home/nfstest/
       
6. Remove the NFS export after testing

       # mmnfs export remove <exported_path>
       
       Example:
       # mmnfs export remove /gpfs/fs1/home/nfstest/

## References

<a href="https://tuna.pok.stglabs.ibm.com/wiki/ES_protocol_install_guide#NFS_Installation_-_Rainer_Wolafka" target="_blank">NFS Installation</a> from GpfsWiki  
<a href="https://www.ibm.com/support/knowledgecenter/STXKQY_5.1.0/com.ibm.spectrum.scale.v5r10.doc/bl1adm_cesproconfig.htm" target="_blank">Configuring the CES and protocol configuration</a> from Knowledge Center  
<a href="https://www.ibm.com/support/knowledgecenter/STXKQY_5.1.0/com.ibm.spectrum.scale.v5r10.doc/bl1adm_command.htm" target="_blank">GPFS commands reference</a> from Knowledge Center  

> **Copyright Notice**: Original work by this blog’s author, licensed under <a href="https://creativecommons.org/licenses/by-nc-nd/4.0/" target="_blank">CC BY-NC-ND 4.0</a>. When reposting, please include a link to the original source and this copyright notice.