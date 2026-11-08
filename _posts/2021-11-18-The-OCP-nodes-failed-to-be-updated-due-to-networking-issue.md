---
title:  The OCP nodes failed to be updated due to networking issue
date: 2021-11-18 21:01:37
categories:
  - Troubleshoot
tags:
  - fusion
  - ocp
  - rhcos 
  - coreos
---

Recently the lab network have problem and often fail to access public registry, I found that OCP nodes failing to be updated during upgrade.

## Environment

- OCP 4.9

## Issue

- During OCP upgrade, some nodes stuck on updating, its status showed as `Ready,SchedulingDisabled` for hours.

## Resolution

- Access the affected node(here using pd176) and enable proxy

      $ ssh core@pd176
      [pd176] $ sudo -i
      [pd176] # export HTTP_PROXY=proxy.emea.ibm.com:80

- Run the following command to pivot into the expected image

      # /run/bin/machine-config-daemon pivot "quay.io/openshift-release-dev/ocp-v4.0-art-dev@sha256:ac2ad5b01eed7b0222dabbfb5c6d4b19ddc85bdaab66738f7ff620e4acffb6ac"
      WARN[0000] Failed to decode the keys ["storage.options.override_kernel_check"] from "/etc/containers/storage.conf".
      I1117 13:18:12.452115   11008 run.go:18] Running: nice -- ionice -c 3 oc image extract --path /:/run/mco-machine-os-content/os-content-542260111 --registry-config /var/lib/kubelet/config.json quay.io/openshift-release-dev/ocp-v4.0-art-dev@sha256:ac2ad5b01eed7b0222dabbfb5c6d4b19ddc85bdaab66738f7ff620e4acffb6ac
      I1117 13:20:46.001938   11008 rpm-ostree.go:325] Running captured: rpm-ostree status --json
      I1117 13:20:46.225003   11008 rpm-ostree.go:233] Previous pivot: quay.io/openshift-release-dev/ocp-v4.0-art-dev@sha256:480e39d63063bae8992542905d48442fd1d9d1325a5136a3be8256d123efe490
      I1117 13:20:53.632739   11008 rpm-ostree.go:265] Pivoting to: 49.84.202111022104-0 (a6ff6114eb7cd8b80d39b8bde48449e900f5ea8e681a563e4b77699b8b46578c)
      I1117 13:20:53.632859   11008 rpm-ostree.go:297] Executing rebase from repo path /run/mco-machine-os-content/os-content-542260111/srv/repo with customImageURL pivot://quay.io/openshift-release-dev/ocp-v4.0-art-dev@sha256:ac2ad5b01eed7b0222dabbfb5c6d4b19ddc85bdaab66738f7ff620e4acffb6ac and checksum a6ff6114eb7cd8b80d39b8bde48449e900f5ea8e681a563e4b77699b8b46578c
      I1117 13:20:53.632902   11008 update.go:1859] Running: rpm-ostree rebase --experimental /run/mco-machine-os-content/os-content-542260111/srv/repo:a6ff6114eb7cd8b80d39b8bde48449e900f5ea8e681a563e4b77699b8b46578c --custom-origin-url pivot://quay.io/openshift-release-dev/ocp-v4.0-art-dev@sha256:ac2ad5b01eed7b0222dabbfb5c6d4b19ddc85bdaab66738f7ff620e4acffb6ac --custom-origin-description Managed by machine-config-operator
      0 metadata, 0 content objects imported; 0 bytes content written
      Checking out tree a6ff611... done
      error: error running rpm-ostree rebase --experimental /run/mco-machine-os-content/os-content-542260111/srv/repo:a6ff6114eb7cd8b80d39b8bde48449e900f5ea8e681a563e4b77699b8b46578c --custom-origin-url pivot://quay.io/openshift-release-dev/ocp-v4.0-art-dev@sha256:ac2ad5b01eed7b0222dabbfb5c6d4b19ddc85bdaab66738f7ff620e4acffb6ac --custom-origin-description Managed by machine-config-operator: error: No enabled repositories

- Due to the networking issue, it might fail to execute the command `rpm-ostree rebase`. You can run below command to use the cache.

      # rpm-ostree rebase -C --experimental /run/mco-machine-os-content/os-content-542260111/srv/repo:a6ff6114eb7cd8b80d39b8bde48449e900f5ea8e681a563e4b77699b8b46578c --custom-origin-url pivot://quay.io/openshift-release-dev/ocp-v4.0-art-dev@sha256:ac2ad5b01eed7b0222dabbfb5c6d4b19ddc85bdaab66738f7ff620e4acffb6ac --custom-origin-description "Managed by machine-config-operator"
      0 metadata, 0 content objects imported; 0 bytes content written
      Checking out tree a6ff611... done
      Resolving dependencies... done
      Checking out packages... done
      Running pre scripts... done
      Running post scripts... done
      Running posttrans scripts... done
      Writing rpmdb... done
      Writing OSTree commit... done
      Staging deployment... done
      Freed: 74.2 MB (pkgcache branches: 0)
      Upgraded:
        NetworkManager 1:1.30.0-10.el8_4 -> 1:1.30.0-13.el8_4
        NetworkManager-cloud-setup 1:1.30.0-10.el8_4 -> 1:1.30.0-13.el8_4
        NetworkManager-libnm 1:1.30.0-10.el8_4 -> 1:1.30.0-13.el8_4
        NetworkManager-ovs 1:1.30.0-10.el8_4 -> 1:1.30.0-13.el8_4
        NetworkManager-team 1:1.30.0-10.el8_4 -> 1:1.30.0-13.el8_4
        NetworkManager-tui 1:1.30.0-10.el8_4 -> 1:1.30.0-13.el8_4
        cri-o 1.22.0-74.rhaos4.9.gitd745cab.el8 -> 1.22.0-77.rhaos4.9.gitd745cab.el8
        curl 7.61.1-18.el8_4.1 -> 7.61.1-18.el8_4.2
        iptables 1.8.4-17.el8 -> 1.8.4-17.el8_4.1
        iptables-libs 1.8.4-17.el8 -> 1.8.4-17.el8_4.1
        kernel 4.18.0-305.19.1.el8_4 -> 4.18.0-305.25.1.el8_4
        kernel-core 4.18.0-305.19.1.el8_4 -> 4.18.0-305.25.1.el8_4
        kernel-devel 4.18.0-305.19.1.el8_4 -> 4.18.0-305.25.1.el8_4
        kernel-headers 4.18.0-305.19.1.el8_4 -> 4.18.0-305.25.1.el8_4
        kernel-modules 4.18.0-305.19.1.el8_4 -> 4.18.0-305.25.1.el8_4
        kernel-modules-extra 4.18.0-305.19.1.el8_4 -> 4.18.0-305.25.1.el8_4
        libcurl 7.61.1-18.el8_4.1 -> 7.61.1-18.el8_4.2
        libnfsidmap 1:2.3.3-41.el8_4.2 -> 1:2.3.3-41.el8_4.3
        libsmbclient 4.13.3-4.el8_4 -> 4.13.3-5.el8_4
        libsolv 0.7.16-2.el8 -> 0.7.16-3.el8_4
        libwbclient 4.13.3-4.el8_4 -> 4.13.3-5.el8_4
        nfs-utils 1:2.3.3-41.el8_4.2 -> 1:2.3.3-41.el8_4.3
        openshift-hyperkube 4.9.0-202110220435.p0.git.a44d0f0.assembly.stream.el8 -> 4.9.0-202111020225.p0.git.d8c4430.assembly.stream.el8
        platform-python 3.6.8-38.el8_4 -> 3.6.8-39.el8_4
        python3-libs 3.6.8-38.el8_4 -> 3.6.8-39.el8_4
        samba-client-libs 4.13.3-4.el8_4 -> 4.13.3-5.el8_4
        samba-common 4.13.3-4.el8_4 -> 4.13.3-5.el8_4
        samba-common-libs 4.13.3-4.el8_4 -> 4.13.3-5.el8_4
        sudo 1.8.29-7.el8 -> 1.8.29-7.el8_4.1
        tzdata 2021c-1.el8 -> 2021e-1.el8
      Run "systemctl reboot" to start a reboot

- Reboot the affected node
     
      # systemctl reboot

## Diagnostic Steps

- The nodes stuck on updating

      # oc get node
      NAME    STATUS                     ROLES           AGE   VERSION
      pd113   Ready                      master,worker   88d   v1.22.1+d8c4430
      pd114   Ready                      master,worker   88d   v1.22.1+d8c4430
      pd115   Ready                      master,worker   88d   v1.22.1+d8c4430
      pd175   Ready                      worker          88d   v1.22.0-rc.0+a44d0f0
      pd176   Ready,SchedulingDisabled   worker          88d   v1.22.0-rc.0+a44d0f0
      pd177   Ready                      worker          88d   v1.22.0-rc.0+a44d0f0

- The affected nodes were not updated

      # oc get mcp
      NAME     CONFIG                                             UPDATED   UPDATING   DEGRADED   MACHINECOUNT   READYMACHINECOUNT   UPDATEDMACHINECOUNT   DEGRADEDMACHINECOUNT   AGE
      master   rendered-master-ad6b2b324da1d602a64856357a7b3bbe   True      False	 False      3              3                   3                     0                      88d
      worker   rendered-worker-3f14d2c16c2c2476de8ce785b56d3709   False     True	 True       3              0                   0                     1                      88d

- After checking the related machine-config-daemon logs, can see similar events as below:

      # oc -n openshift-machine-config-operator logs machine-config-daemon-2qmxv -c machine-config-daemon
      ...
      unexpected on-disk state validating against rendered-worker-1d18a2fca29ad7999d5fe2e61f2944fd: expected target osImageURL "quay.io/openshift-release-dev/ocp-v4.0-art-dev@sha256:ac2ad5b01eed7b0222dabbfb5c6d4b19ddc85bdaab66738f7ff620e4acffb6ac", have "quay.io/openshift-release-dev/ocp-v4.0-art-dev@sha256:480e39d63063bae8992542905d48442fd1d9d1325a5136a3be8256d123efe490"

## Reference

<a href="https://access.redhat.com/solutions/4951051" target="_blank">OCP 4 node degraded: expected target osImageURL</a>  
<a href="https://access.redhat.com/solutions/5598401" target="_blank">Cluster Updates Without Error but Machine Config Pools Degraded with "Marking Degraded due to: unexpected on-disk state" on OCP 4.6 and newer</a>  

> **Copyright Notice**: Original work by this blog’s author, licensed under <a href="https://creativecommons.org/licenses/by-nc-nd/4.0/" target="_blank">CC BY-NC-ND 4.0</a>. When reposting, please include a link to the original source and this copyright notice.