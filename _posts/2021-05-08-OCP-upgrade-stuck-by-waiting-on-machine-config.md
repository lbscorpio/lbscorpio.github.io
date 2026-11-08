---
title:  OCP upgrade stuck by waiting on machine-config
date: 2021-05-08 17:50:20
categories:
  - Troubleshoot
tags:
  - ocp
  - oc
---

I did OCP upgrade several times, found that it often stuck by waiting on machine-config. Not sure the root cause yet, but confirmed a valid workaround.

## Environment

- Red Hat OpenShift 4.7

## Issue

- OCP upgrade stuck by waiting on machine-config, which reported `99-master-generated-registries` controller version mismatch.

## Resolution

- Confirm only `99-master-generated-registries` controller version old and others already latest.
- Update its metadata manually by the following command:

      # oc edit mc 99-master-generated-registries
      metadata:
        annotations:
      >>  machineconfiguration.openshift.io/generated-by-controller-version: <update_new_version_here>
- Later the upgrade will continue and finish.

## Diagnostic Steps

- Checking current upgrade state, found that only one cluster operator `machine-config` not available.

      # date; oc get clusterversions
      Sat May  8 18:00:47 CST 2021
      NAME      VERSION   AVAILABLE   PROGRESSING   SINCE   STATUS
      version   4.7.8     True        True          7h38m   Working towards 4.7.9: 560 of 668 done (83% complete)
      # date; oc get clusteroperators
      Sat May  8 18:01:43 CST 2021
      NAME                                       VERSION   AVAILABLE   PROGRESSING   DEGRADED   SINCE
      ...output omitted...
      machine-config                             4.7.8     False       True          True       6h44m

- Checking its conditions status, there were two errors which pointed to the controller version mismatch.

      # date; oc describe clusteroperators machine-config
      Sat May  8 18:08:04 CST 2021
      Name:         machine-config
      Namespace:
      Labels:       <none>
      Annotations:  exclude.release.openshift.io/internal-openshift-hosted: true
                    include.release.openshift.io/self-managed-high-availability: true
                    include.release.openshift.io/single-node-developer: true
      API Version:  config.openshift.io/v1
      Kind:         ClusterOperator
      ...output omitted...
      Status:
        Conditions:
          ...output omitted...
          Last Transition Time:  2021-05-08T03:50:07Z
          Message:               Unable to apply 4.7.9: timed out waiting for the condition during syncRequiredMachineConfigPools: pool master has not progressed to latest configuration: controller version mismatch for 99-master-generated-registries expected ac79a2ffc6002f086b3fa80003b278b635d2055a has 116603ff3d7a39c0de52d7d16fe307c8471330a0: all 3 nodes are at latest configuration rendered-master-1c5dcb64c35c3b6fe87f2ba6c76aa602, retrying
          Reason:                RequiredPoolsFailed
          Status:                True
          Type:                  Degraded
          Last Transition Time:  2021-05-08T03:17:26Z
          Message:               Cluster not available for 4.7.9
          Status:                False
          Type:                  Available
        Extension:
          Last Sync Error:  pool master has not progressed to latest configuration: controller version mismatch for 99-master-generated-registries expected ac79a2ffc6002f086b3fa80003b278b635d2055a has 116603ff3d7a39c0de52d7d16fe307c8471330a0: all 3 nodes are at latest configuration rendered-master-1c5dcb64c35c3b6fe87f2ba6c76aa602, retrying
          Master:           all 3 nodes are at latest configuration rendered-master-1c5dcb64c35c3b6fe87f2ba6c76aa602
          Worker:           all 3 nodes are at latest configuration rendered-worker-d99f619a2e383cf571b689fb8a84f8ee

- Checking all machineconfig, they should be updated in fact. But `99-master-generated-registries` still kept the old one.

      # date; oc get machineconfigpool
      Sat May  8 18:14:25 CST 2021
      NAME     CONFIG                                             UPDATED   UPDATING   DEGRADED   MACHINECOUNT   READYMACHINECOUNT   UPDATEDMACHINECOUNT   DEGRADEDMACHINECOUNT   AGE
      master   rendered-master-1c5dcb64c35c3b6fe87f2ba6c76aa602   True      False      False      3              3                   3                     0                      45d
      worker   rendered-worker-d99f619a2e383cf571b689fb8a84f8ee   True      False      False      3              3                   3                     0                      45d

      # date; oc get machineconfig
      Sat May  8 18:16:25 CST 2021
      NAME                                               GENERATEDBYCONTROLLER                      IGNITIONVERSION   AGE
      00-master                                          ac79a2ffc6002f086b3fa80003b278b635d2055a   3.2.0             45d
      00-worker                                          ac79a2ffc6002f086b3fa80003b278b635d2055a   3.2.0             45d
      01-master-container-runtime                        ac79a2ffc6002f086b3fa80003b278b635d2055a   3.2.0             45d
      01-master-kubelet                                  ac79a2ffc6002f086b3fa80003b278b635d2055a   3.2.0             45d
      01-worker-container-runtime                        ac79a2ffc6002f086b3fa80003b278b635d2055a   3.2.0             45d
      01-worker-kubelet                                  ac79a2ffc6002f086b3fa80003b278b635d2055a   3.2.0             45d
      03-worker-extensions                                                                          3.1.0             40d
      99-ibm-attach                                                                                 2.2.0             43d
      99-master-generated-registries                     116603ff3d7a39c0de52d7d16fe307c8471330a0   3.2.0             45d
      99-master-ssh                                                                                 3.1.0             45d
      99-openshift-machineconfig-worker-kargs                                                                         40d
      99-worker-generated-containerruntime               ac79a2ffc6002f086b3fa80003b278b635d2055a   3.2.0             40d
      99-worker-generated-registries                     39de2f3d075d84c945f02d209c02c17b24d5dd71   3.2.0             45d
      99-worker-ssh                                                                                 3.1.0             45d
      rendered-master-12b5487116baeb36c4394d6f174a082f   116603ff3d7a39c0de52d7d16fe307c8471330a0   3.2.0             9d
      rendered-master-1c5dcb64c35c3b6fe87f2ba6c76aa602   ac79a2ffc6002f086b3fa80003b278b635d2055a   3.2.0             6h52m
      rendered-master-99353247eb255890ab38b6681a627f5c   116603ff3d7a39c0de52d7d16fe307c8471330a0   3.2.0             12d
      rendered-master-d5dc1e815e7cd6ee200aad853733c935   a29f0d80366f4a6eb72f163cb11249b8b965a22b   3.2.0             17d
      rendered-worker-1b18f382325f3b810824faf66220e0a3   39de2f3d075d84c945f02d209c02c17b24d5dd71   3.2.0             45d
      ...output omitted...

- So I tried to update the controller version of `99-master-generated-registries`
manually, and it worked.

> **Copyright Notice**: Original work by this blog’s author, licensed under <a href="https://creativecommons.org/licenses/by-nc-nd/4.0/" target="_blank">CC BY-NC-ND 4.0</a>. When reposting, please include a link to the original source and this copyright notice.