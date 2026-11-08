---
title: ImageContentSourcePolicy didn't work on OCP 4.9
date: 2022-03-02 16:33:37
categories:
  - Troubleshoot
tags:
  - fusion
  - ocp
  - cnsa
---

The container images used by CNSA by our manifests reference their production ICR locations (which is only available at our GA date). In order to facilitate testing and development, you will need to create a ImageContentSourcePolicy to establish a mirror to artifactory.

But it didn't work on our lab environment, even I tried two OCP clusters.

## Environment

- OCP 4.9
- CNSA 5.1.3 dev build

## Issue

- After applied below resource `ImageContentSourcePolicy` on the OCP cluster, it didn't work.

  ```
  apiVersion: operator.openshift.io/v1alpha1
  kind: ImageContentSourcePolicy
  metadata:
    name: prod-icr-to-artifactory-mirror
  spec:
    repositoryDigestMirrors:
    - mirrors:
      - sys-spectrum-scale-team-cloud-native-docker-local.artifactory.swg-devops.com
      source: cp.icr.io/cp/spectrum/scale
    - mirrors:
      - sys-spectrum-scale-team-cloud-native-docker-local.artifactory.swg-devops.com
      source: icr.io/cpopen
  ```

## Resolution

- The resource `image.config.openshift.io/cluster` have incorrect config on the affected OCP clusters, removed the `blockedRegistries` configuration under its spec parameter `registrySources` then solved the issue.

## Diagnostic Steps

- Access the affected node(here using mc181) and checking configuration, it didn't apply the new config from the resource `ImageContentSourcePolicy`. Below output was the default configuration.

      $ ssh core@mc181 cat /etc/containers/registries.conf
      unqualified-search-registries = ['registry.access.redhat.com', 'docker.io']
      
- Checking the logs from `machine-config-controller`, it reported below errors:

      $ oc -n openshift-machine-config-operator logs machine-config-controller-xxx
      2022-02-20T15:39:17.037247339Z I0220 15:39:17.037008       1 container_runtime_config_controller.go:365] Error syncing image config openshift-config: could not Create/Update MachineConfig: could not update policy json with new changes: invalid images config: only one of AllowedRegistries or BlockedRegistries may be specified

- Run the following command to confirm the image configuration.

      $ oc get image.config.openshift.io/cluster -o yaml
      ...
      spec:
        allowedRegistriesForImport:
        - domainName: quay.io
          insecure: false
        - domainName: registry.redhat.io
          insecure: false
        registrySources:
          allowedRegistries:
          - quay.io
          - registry.redhat.io
          blockedRegistries:
          - untrusted.com
          insecureRegistries:
          - 9.110.70.75
          - mc75.cn.ibm.com

- According to the official OCP docs, either `blockedRegistries` or `allowedRegistries` can be set, but not both. So the problem was the invalid image configuration.

## References

<a href="https://github.ibm.com/IBMSpectrumScale/scale-core/blob/master/docs/dev/container_image_auth.md#update-imagecontentsourcepolicy-to-establish-a-mirror" target="_blank">Creating an ImageContentSourcePolicy to establish a mirror</a>  
<a href="https://docs.openshift.com/container-platform/4.9/openshift_images/image-configuration.html" target="_blank">Image configuration resources on OCP 4.9</a>  

> **Copyright Notice**: Original work by this blog’s author, licensed under <a href="https://creativecommons.org/licenses/by-nc-nd/4.0/" target="_blank">CC BY-NC-ND 4.0</a>. When reposting, please include a link to the original source and this copyright notice.