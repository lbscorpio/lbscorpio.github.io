---
title:  How to setup Podman on MacOS
date: 2022-02-21 19:58:21
categories:
  - Procedure
tags:
  - fusion
  - docker
  - podman
  - macos
---

Docker announced their product `Docker Desktop` is going to require a subscription for companies with more than 250 employees. I decided to use Podman as an alternative in advance due to `Docker Desktop` will require a paid license for us soon.

## Prerequisites

- Intel-based or Arm-based Apple machine, tested on Apple M1s with Podman 4.5.1 without problem.
- Homebrew installed on your MacOS, see <a href="https://brew.sh/" target="_blank">here</a> for steps.

## Install Podman Mac client

- Install podman with the following command

      $ brew install podman

## To start the Podman-managed VM

- Create a virtual machine in qemu (installed by home-brew)

      $ podman machine init --disk-size 20
      Downloading VM image: fedora-coreos-35.20220227.2.0-qemu.x86_64.qcow2.xz: done
      Extracting compressed file
      Image resized.

   **Note**: The default disk size is 10 GB, it is not enough for a development/testing environment, so have better to increase it. 


- Start the virtual machine and verify the installation information

      $ podman machine start
      INFO[0000] waiting for clients...
      INFO[0000] listening tcp://127.0.0.1:7777
      INFO[0000] new connection from  to /var/folders/lx/544p3srs67bfwp_476n_dwb00000gn/T/podman/qemu_podman-machine-default.sock
      Waiting for VM ...
      Machine "podman-machine-default" started successfully   

      $ podman version
      Client:
      Version:      3.4.4
      API Version:  3.4.4
      Go Version:   go1.17.3
      Built:        Thu Dec  9 02:41:11 2021
      OS/Arch:      darwin/amd64
      
      Server:
      Version:      3.4.4
      API Version:  3.4.4
      Go Version:   go1.16.8
      Built:        Thu Dec  9 05:45:07 2021
      OS/Arch:      linux/amd64

## Backup Podman configuration if needed

- Remote ssh destinations information for the Podman service

      $ podman system connection list
      Name                         Identity                                  URI
      podman-machine-default*      /Users/lbosh/.ssh/podman-machine-default  ssh://core@localhost:64621/run/user/1000/podman/podman.sock
      podman-machine-default-root  /Users/lbosh/.ssh/podman-machine-default  ssh://root@localhost:64621/run/podman/podman.sock

- If the connection lost by mistake (e.g. issue `podman machine rm` but not destroy it at last), you cannot connect to Podman and most of podman commands failure as below errors:

      Cannot connect to Podman. Please verify your connection to the Linux system using `podman system connection list`, or try `podman machine init` and `podman machine start` to manage a new Linux VM
      Error: unable to connect to Podman socket: Get "http://d/v3.4.4/libpod/_ping": dial unix ///var/folders/lx/544p3srs67bfwp_476n_dwb00000gn/T/podman-run--1/podman/podman.sock: connect: no such file or directory
  
- Then you can add the connections manually with the backup info.

      $ podman system connection add --identity ~/.ssh/podman-machine-default podman-machine-default ssh://core@localhost:64621/run/user/1000/podman/podman.sock
      $ podman system connection add --identity ~/.ssh/podman-machine-default podman-machine-default-root ssh://root@localhost:64621/run/podman/podman.sock

## Replace docker for uninstalling `Docker Desktop`

- Symlink docker for podman

      $ cd "$(dirname `which podman`)"
      $ ln -s podman docker

- Or add an alias to `~/.zshrc` and save

      $ vi ~/.zshrc
      alias docker=podman

## Test if the command docker working fine

- Run the following command and should see the similar outputs as below:

      $ docker run hello-world
      Resolved "hello-world" as an alias (/etc/containers/registries.conf.d/000-shortnames.conf)
      Trying to pull docker.io/library/hello-world:latest...
      Getting image source signatures
      Copying blob sha256:2db29710123e3e53a794f2694094b9b4338aa9ee5c40b930cb8063a1be392c54
      Copying blob sha256:2db29710123e3e53a794f2694094b9b4338aa9ee5c40b930cb8063a1be392c54
      Copying config sha256:feb5d9fea6a5e9606aa995e879d862b825965ba48de054caab5ef356dc6b3412
      Writing manifest to image destination
      Storing signatures
      
      Hello from Docker!
      This message shows that your installation appears to be working correctly.
      
      To generate this message, Docker took the following steps:
       1. The Docker client contacted the Docker daemon.
       2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
          (amd64)
       3. The Docker daemon created a new container from that image which runs the
          executable that produces the output you are currently reading.
       4. The Docker daemon streamed that output to the Docker client, which sent it
          to your terminal.
      
      To try something more ambitious, you can run an Ubuntu container with:
       $ docker run -it ubuntu bash
      
      Share images, automate workflows, and more with a free Docker ID:
       https://hub.docker.com/
      
      For more examples and ideas, visit:
       https://docs.docker.com/get-started/

## Add insecure registry to Podman configuration if needed

- In a development/testing environment, we often use local registry (e.g. Harbor) to store images. Podman might fail to pull/push images from such insecure registry by default, similar errors as below:

      Error: initializing source docker://mc75.cn.ibm.com/isf/isf-cns-operator:0.0.1: pinging container registry mc75.cn.ibm.com: Get "https://mc75.cn.ibm.com/v2/": x509: certificate signed by unknown authority

- SSH into a Podman-managed virtual machine and switch to root prompt

      $ podman machine ssh
      [core@localhost ~]$ sudo -i
      [root@localhost ~]#

- Update the configuration file and save, then exit.

      [root@localhost ~]# vi /etc/containers/registries.conf
      [[registry]]
      location = "mc75.cn.ibm.com"
      insecure = true
      [root@localhost ~]# exit
      [core@localhost ~]$ exit

- Verify if can pull the image from the insecure registry

      $ docker pull mc75.cn.ibm.com/isf/isf-cns-operator:0.0.1

## References

<a href="https://w3.ibm.com/w3publisher/docker-desktop/podman" target="_blank">Podman Installation</a> from w3Publisher  
<a href="https://pages.github.ibm.com/cio-pricing-system/podman-documentation/" target="_blank">Replacing Docker Desktop with Podman Desktop</a>  
<a href="https://podman.io/" target="_blank">Podman official website</a>  
<a href="https://developers.redhat.com/cheat-sheets/podman-basics-cheat-sheet" target="_blank">Podman Basics Cheat Sheet</a>

> **Copyright Notice**: Original work by this blog’s author, licensed under <a href="https://creativecommons.org/licenses/by-nc-nd/4.0/" target="_blank">CC BY-NC-ND 4.0</a>. When reposting, please include a link to the original source and this copyright notice.