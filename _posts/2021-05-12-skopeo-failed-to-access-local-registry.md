---
title:  skopeo failed to access local registry with error "certificate signed by unknown authority"
date: 2021-05-12 18:02:50
categories:
  - Troubleshoot
tags:
  - skopeo
  - docker
---

`skopeo` is a command line utility that performs various operations on container images and image repositories.

## Environment

- macOS Big Sur 11.3.1

## Issue

- Failed to run `skopeo` commands to the local registry, with error "certificate signed by unknown authority".

## Resolution

- Obtain the local registry certificate firstly

      $ echo -n | openssl s_client -showcerts -connect <local_registry_IP>:443 2>/dev/null | sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' > cert.crt

- Add the certificate to the Mac's keychain

      $ sudo security add-trusted-cert -d -r trustRoot -k /Library/Keychains/System.keychain cert.crt

## Diagnostic Steps

- At the beginning, I just want to inspect one docker image on the local registry, but found errors as below:

      $ skopeo inspect --raw docker://9.110.70.75/ibmcom/ibm-storage-odf-operator:0.1.0-73ca917
      FATA[0000] Error parsing image name "docker://9.110.70.75/ibmcom/ibm-storage-odf-operator:0.1.0-73ca917": error pinging docker registry 9.110.70.75: Get "https://9.110.70.75/v2/": x509: certificate signed by unknown authority

- Later, I tried to login the same registry but failed with same error.

      $ skopeo login 9.110.70.75 -u mcdev -p Machinec1ty
      FATA[0000] error authenticating creds for "9.110.70.75": error pinging docker registry 9.110.70.75: Get "https://9.110.70.75/v2/": x509: certificate signed by unknown authority

- `skopeo` does not require a daemon, can't use configuration file like `/etc/docker/daemon.json`. So adding the certification to the trusted store should be fine.

## References

<a href="https://github.com/containers/skopeo" target="_blank">skopeo on GitHub</a>  
<a href="https://blog.container-solutions.com/adding-self-signed-registry-certs-docker-mac" target="_blank">Adding Self-signed Registry Certs to Docker & Docker for Mac</a>

> **Copyright Notice**: Original work by this blog’s author, licensed under <a href="https://creativecommons.org/licenses/by-nc-nd/4.0/" target="_blank">CC BY-NC-ND 4.0</a>. When reposting, please include a link to the original source and this copyright notice.