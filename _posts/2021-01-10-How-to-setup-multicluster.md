---
title:  How to setup a GPFS multicluster environment
date: 2021-01-10 18:53:20
categories:
  - Setup
tags:
  - ptf
  - gpfs
  - multicluster
---

A GPFS cluster can configure mixed OS as nodes, and also can mount a remote GPFS file system. This article is to setup a multicluster for regression test.

There are two different cluster roles on a multicluster environment, server cluster(aka owning cluster or home cluster) and client cluster(aka accessing cluster). The client cluster is the one that mounts the remote file system from the server cluster.

## Prerequisites

- Two existing GPFS clusters, at least one file system mounted on the server cluster.
- The package _gpfs.gskit_ must be installed on both clusters.

## Steps to setup a multicluster environment

### Generate authentication key files on both clusters

- Run below commands **on both clusters** to generate new key files
 
      # mmshutdown -a
      # mmauth genkey new
      # mmlsconfig | grep cipherList       // Check if cipherList is specified or not
      # mmchconfig cipherList=AUTHONLY     // Specify cipherList if not set yet
      # mmstartup -a

- Copy the new generated public key files(/var/mmfs/ssl/id_rsa_new.pub) to remote cluster. For example, use scp command to copy the one of server cluster to client cluster, and vice versa.

### Add the client cluster public key to the server cluster

    (server cluster)# mmauth add <client_cluster> -k <the_public_key_of_client_cluster>
    
    Example:
    # mmauth add rhpower.GPFS -k 613id_rsa_new.pub
    mmauth: Propagating the cluster configuration data to all affected nodes.
    mmauth: Command successfully completed
    
### Add the server cluster on the client cluster

    (client cluster)# mmremotecluster add <server_cluster> -k <the_public_key_of_server_cluster> -n <nodes_of_server_cluster>
    
    Example:
    # mmremotecluster add rhsles.GPFS -k ~/628id_rsa_new.pub -n c550f6u28,c550f5u28,c550f5u31
    mmremotecluster: Command successfully completed
    mmremotecluster: Propagating the cluster configuration data to all
      affected nodes.  This is an asynchronous process.

### Grant the access to the file system of server cluster

    (server cluster)# mmauth grant <client_cluster> -f <file_system>
    
    Example:
    # mmauth grant rhpower.GPFS -f fs2

    mmauth: Granting cluster rhpower.GPFS access to file system fs2:
            access type rw; root credentials will not be remapped.

    mmauth: Propagating the cluster configuration data to all affected nodes.
    mmauth: Command successfully completed 

### Define the remote file system on client cluster

    (client cluster)# mmremotefs add <new_local_file_system> -f <remote_file_system> -C <server_cluster> -A no
    
    Example:
    # mmremotefs add 628_fs2 -f fs2 -C rhsles.GPFS -A no
    mmremotefs: Propagating the cluster configuration data to all
      affected nodes.  This is an asynchronous process.

### Mount the file system on the client cluster

    (client cluster)# mmmount <new_local_file_system> -a
    
    Example:
    # mmmount 628_fs2 -a

## References

<a href="https://tuna.pok.stglabs.ibm.com/wiki/Multicluster" target="_blank">Multicluster</a> from GpfsWiki  
<a href="https://www.ibm.com/support/knowledgecenter/STXKQY_5.1.0/com.ibm.spectrum.scale.v5r10.doc/bl1adv_admmcch.htm" target="_blank">Accessing a remote GPFS file system</a> from Knowledge Center  
<a href="https://puppetfox.tistory.com/14" target="_blank">Setting up a multicluster environment using General Parallel File System</a>  

> **Copyright Notice**: Original work by this blog’s author, licensed under <a href="https://creativecommons.org/licenses/by-nc-nd/4.0/" target="_blank">CC BY-NC-ND 4.0</a>. When reposting, please include a link to the original source and this copyright notice.