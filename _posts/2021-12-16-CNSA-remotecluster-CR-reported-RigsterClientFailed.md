---
title:  CNSA remotecluster CR reported RegisterClientFailed
date: 2021-12-16 20:01:50
categories:
  - Troubleshoot
tags:
  - fusion
  - cnsa
---

IBM Spectrum® Scale container native is a containerized version of IBM Spectrum Scale. IBM Spectrum® Scale container native storage access(CNSA) using a remote mount attached file system to provide a persistent data store to be accessed by the applications through the IBM Spectrum Scale CSI driver by using Persistent Volumes (PVs).

## Environment

- OCP 4.9
- CNSA 5.1.2.1

## Issue

- During CNSA deployment on OCP, the remotecluster CR reported error `RegisterClientFailed`.

## Resolution

- Manually create a self-signed certificate for remote GPFS GUI

      $ openssl req -x509 -newkey rsa:4096 -sha256 -days 3650 -nodes \
      -keyout example.key -out example.crt -subj "/C=CN/ST=SH/L=SH/O=IBM/OU=gpfsgui/CN=www.example.com" \
      -addext "subjectAltName=DNS:www.example.com,IP:9.1.2.3"

- Install the cert and key files on remote GPFS GUI

  GPFS GUI console -> Services -> GUI -> tab Nodes -> Choose button "Installs an already issued certificate and the corresponding private key." to import the above certificate/key files(example.crt/example.key).

- Restart the GPFS GUI service on the GPFS GUI node

      # systemctl restart gpfsgui

## Diagnostic Steps

- From the remotecluster CR status, can see below error:

      status:
        conditions:
        - lastTransitionTime: "2021-12-14T10:14:52Z"
          message: Could not register client cluster on storage cluster.
          reason: RegisterClientFailed
          status: "False"
          type: Ready

- Checking the CNSA operator logs, can locate the detailed message.

      2021-12-14T13:24:00.949Z	V(1)	controller-runtime.manager.events	Warning	{"object": {"kind":"RemoteCluster","namespace":"ibm-spectrum-scale","name":"ibm-spectrum-scale-remote","uid":"15a687a6-ed88-4e7d-9c03-23abcc965598","apiVersion":"scale.spectrum.ibm.com/v1beta1","resourceVersion":"13718399"}, "reason": "RegisterClientFailed", "message": "Post \"https://mc81.cn.ibm.com:443/scalemgmt/v2/remotemount/remoteclusters\": x509: certificate relies on legacy Common Name field, use SANs or temporarily enable Common Name matching with GODEBUG=x509ignoreCN=0"}

- So the problem is the remote cluster GUI certificate issue.

## Reference

<a href="https://stackoverflow.com/questions/10175812/how-to-generate-a-self-signed-ssl-certificate-using-openssl" target="_blank">How to generate a self-signed SSL certificate using OpenSSL?</a>  

> **Copyright Notice**: Original work by this blog’s author, licensed under <a href="https://creativecommons.org/licenses/by-nc-nd/4.0/" target="_blank">CC BY-NC-ND 4.0</a>. When reposting, please include a link to the original source and this copyright notice.