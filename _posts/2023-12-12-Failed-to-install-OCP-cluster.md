---
title:  Failed to install OCP cluster
date: 2023-12-12 21:28:37
categories:
  - Troubleshoot
tags:
  - fusion
  - ocp
  - rhcos
  - coreos
---

I have to reinstall one baremetal environment to OCP 4.12 for a field case testing, but hit several issues and recorded here for future reference.

## Environment

- Dell PowerEdge R730xd
- OCP 4.12

## Issue#1: Failed to do PXE boot with UEFI mode

- When doing PXE boot with UEFI mode, the console will report "Boot Failed".

### Resolution

- Change the boot mode to BIOS

  Hit `F2` key during booting, enter "System Setup" screen, "System BIOS" -> "Boot Settings" -> Set the boot mode to BIOS.

## Issue#2: bootstrap reported error creating container storage

- After installed CoreOS on master nodes and rebooted them including bootstrap, the bootstrap reported below errors in a loop:

      Dec 12 09:43:13 mc26 systemd[1]: Started Bootstrap a Kubernetes cluster.
      Dec 12 09:43:22 mc26 bootkube.sh[28230]: Check if API and API-Int URLs are resolvable during bootstrap
      Dec 12 09:43:22 mc26 bootkube.sh[28230]: Checking validity of api.g10.mcocp.com of type API_URL
      Dec 12 09:43:22 mc26 bootkube.sh[28230]: Starting stage check-api-url
      Dec 12 09:43:22 mc26 bootkube.sh[28230]: Successfully resolved API_URL api.g10.mcocp.com
      Dec 12 09:43:22 mc26 bootkube.sh[28230]: Unable to reach API_URL's https endpoint at https://9.197.49.20:6443/version
      Dec 12 09:43:22 mc26 bootkube.sh[28230]: It might be too early for the https://9.197.49.20:6443/version to be available.
      Dec 12 09:43:22 mc26 bootkube.sh[28230]: Checking validity of api-int.g10.mcocp.com of type API_INT_URL
      Dec 12 09:43:22 mc26 bootkube.sh[28230]: Starting stage check-api-int-url
      Dec 12 09:43:23 mc26 bootkube.sh[28230]: Successfully resolved API_INT_URL api-int.g10.mcocp.com
      Dec 12 09:43:23 mc26 bootkube.sh[28230]: Unable to reach API_INT_URL's https endpoint at https://9.197.49.20:6443/version
      Dec 12 09:43:23 mc26 bootkube.sh[28230]: It might be too early for the https://9.197.49.20:6443/version to be available.
      Dec 12 09:43:23 mc26 bootkube.sh[29436]: https://localhost:2379 is healthy: successfully committed proposal: took = 11.958133ms
      Dec 12 09:43:23 mc26 bootkube.sh[28230]: Starting cluster-bootstrap...
      Dec 12 09:43:24 mc26 bootkube.sh[28230]: Waiting for CEO to finish...
      Dec 12 09:43:24 mc26 bootkube.sh[29512]: Error: error creating container storage: the container name "wait-for-ceo" is already in use by e1bf132c541e0d52dfd13b5a04a420029d6d0500ed3905b5a2b4b4b8f098aa0c. You have to remove that container to be able to reuse that name: that name is already in use
      Dec 12 09:43:24 mc26 systemd[1]: bootkube.service: Main process exited, code=exited, status=125/n/a
      Dec 12 09:43:24 mc26 systemd[1]: bootkube.service: Failed with result 'exit-code'.
      Dec 12 09:43:29 mc26 systemd[1]: bootkube.service: Service RestartSec=5s expired, scheduling restart.
      Dec 12 09:43:29 mc26 systemd[1]: bootkube.service: Scheduled restart job, restart counter is at 21.
      Dec 12 09:43:29 mc26 systemd[1]: Stopped Bootstrap a Kubernetes cluster.
  
  > Note: After accessing bootstrap node, issuing command `journalctl -b -f -u release-image.service -u bootkube.service` can monitor the installation process.

### Resolution

- Verify that the offending container does not exist anywhere

      [root@master01 ~]# crictl ps -a | grep "wait-for-ceo"
      [root@master01 ~]# podman ps -a | grep "wait-for-ceo"

- Remove the storage on bootstrap node

      [root@bootstrap ~]# podman rm --storage wait-for-ceo

## Issue#3: Failed to create Etcd cluster

- After issue#2 was resolved, bootstrap stuck with below events:

      Dec 12 12:56:07 mc26 bootkube.sh[6230]: https://localhost:2379 is healthy: successfully committed proposal: took = 91.371615ms
      Dec 12 12:56:08 mc26 bootkube.sh[1442]: Starting cluster-bootstrap...
      Dec 12 12:56:08 mc26 bootkube.sh[1442]: Waiting for CEO to finish...
      Dec 12 12:56:08 mc26 bootkube.sh[6306]: I1212 12:56:08.864034       1 waitforceo.go:67] waiting on condition EtcdRunningInCluster in etcd CR /cluster to be True.
      Dec 12 12:59:33 mc26 bootkube.sh[6306]: I1212 12:59:33.361334       1 waitforceo.go:67] waiting on condition EtcdRunningInCluster in etcd CR /cluster to be True.
      Dec 12 13:00:52 mc26 bootkube.sh[6306]: I1212 13:00:52.073137       1 waitforceo.go:67] waiting on condition EtcdRunningInCluster in etcd CR /cluster to be True.
      Dec 12 13:00:52 mc26 bootkube.sh[6306]: I1212 13:00:52.197609       1 waitforceo.go:67] waiting on condition EtcdRunningInCluster in etcd CR /cluster to be True.
      Dec 12 13:00:52 mc26 bootkube.sh[6306]: I1212 13:00:52.764534       1 waitforceo.go:67] waiting on condition EtcdRunningInCluster in etcd CR /cluster to be True.

### Diagnostic Steps

- After checking the output of CLI `oc get node`, only one master was Ready, other two nodes didn't exist.

- Later checking the output of CLI `oc get csr`, found that many pending CSR not approved yet.

      csr-s8hgh   20m     kubernetes.io/kube-apiserver-client-kubelet   system:serviceaccount:openshift-machine-config-operator:node-bootstrapper   <none>              Pending
      csr-t9n2p   3h25m   kubernetes.io/kube-apiserver-client-kubelet   system:serviceaccount:openshift-machine-config-operator:node-bootstrapper   <none>              Pending
      csr-vpsxc   4h28m   kubernetes.io/kube-apiserver-client-kubelet   system:serviceaccount:openshift-machine-config-operator:node-bootstrapper   <none>              Pending
      csr-vq7sf   5m23s   kubernetes.io/kube-apiserver-client-kubelet   system:serviceaccount:openshift-machine-config-operator:node-bootstrapper   <none>              Pending
      csr-wgfpw   3h40m   kubernetes.io/kube-apiserver-client-kubelet   system:serviceaccount:openshift-machine-config-operator:node-bootstrapper   <none>              Pending
      csr-wxxmd   143m    kubernetes.io/kube-apiserver-client-kubelet   system:serviceaccount:openshift-machine-config-operator:node-bootstrapper   <none>              Pending
      csr-xks2d   4h58m   kubernetes.io/kube-apiserver-client-kubelet   system:serviceaccount:openshift-machine-config-operator:node-bootstrapper   <none>              Pending
      csr-xvx8x   81m     kubernetes.io/kube-apiserver-client-kubelet   system:serviceaccount:openshift-machine-config-operator:node-bootstrapper   <none>              Pending

### Resolution

- Manual approve all pending CSR from bootstrap node, you might run below command two times, then other two master nodes will be added and the bootstrapping will be completed.

      [root@bootstrap ~]# oc get csr -o go-template='{{range .items}}{{if not .status}}{{.metadata.name}}{{"\n"}}{{end}}{{end}}' | xargs oc adm certificate approve

## References

<a href="https://cloud.tencent.com/developer/article/1640415" target="_blank">Openshift4 的安装过程</a>  
<a href="https://www.dell.com/support/manuals/en-us/poweredge-r730xd/r730xd_ompublication/boot-settings?guid=guid-9f5af39d-242e-479c-81b5-aa92e8bee63b&lang=en-us" target="_blank">Dell PowerEdge R730xd Owner's Manual</a>  
<a href="https://access.redhat.com/solutions/5067971" target="_blank">Error: the container name (...) is already in use by (...) But container does not exist in OpenShift 4.x</a>  

> **Copyright Notice**: Original work by this blog’s author, licensed under <a href="https://creativecommons.org/licenses/by-nc-nd/4.0/" target="_blank">CC BY-NC-ND 4.0</a>. When reposting, please include a link to the original source and this copyright notice.