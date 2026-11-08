---
title:  Upgrading RHEL without RHSM from 7.9 to 8.2
date: 2021-01-06 14:53:20
categories:
  - Procedure
tags:
  - linux
  - leapp
  - upgrade
---

Almost all of our testing machines are accessed remotely, so you have to patch or upgrade some of them to the specific versions some day. Now, I have a chance to upgrade a node from RHEL 7.9 to 8.2.

## Prerequisites

- An existing RHEL 7.9 with custom repositories, and a custom repository file of RHEL 8.2.
- The custom repositories should be including:
 
      RHEL 7 Base (to provide the latest RHEL 7 packages)
      RHEL 7 Extras (to provide the Leapp utility and its dependencies)
      RHEL 8 BaseOS
      RHEL 8 AppStream

- Ensure your system does not use more than one Network Interface Card (NIC) with a name based on the prefix used by the kernel (eth). If have, can follow up <a href="https://pages.github.ibm.com/lbosh/myblog/procedure/2021/01/06/How-to-change-network-device-naming-from-eth-to-eno/" target="_blank">this article</a> to change it.
- If the root filesystem is XFS, ensure its ftype is not 0, otherwise the upgrade will hit <a href="https://access.redhat.com/solutions/5057391" target="_blank">this known issue</a>.

## Procedures

1. Ensure you have the system locale set to en_US.UTF-8:
   
       # cat /etc/locale.conf
       
2. Update all packages to the latest RHEL 7 version if not

       # yum update -y

   > **Note**: Don't enable RHEL 8 yum repositories at this step.

3. Reboot the system if upgrading the kernel

       # reboot
       
4. Install the Leapp utility

       # yum install -y leapp leapp-repository

5. Download additional required data files "leapp-data12.tar.gz" from <a href="https://access.redhat.com/articles/3664871" target="_blank">Red Hat</a>, and extract them in the /etc/leapp/files/ directory.

6. Perform the pre-upgrade phase
 
       # leapp preupgrade --no-rhsm --enablerepo repository_id1 --enablerepo repository_id2 ...
       
       Example:
       # leapp preupgrade --no-rhsm --enablerepo baseos --enablerepo appstream

7. Examine the report in the /var/log/leapp/leapp-report.txt file, and manually resolve all the reported problems which marking as **inhibitor**, and provide answers to each question required by Leapp.

       # leapp answer --section remove_pam_pkcs11_module_check.confirm=True

8. Perform the upgrade phase if the leapp report all is GREEN.
       
       # leapp upgrade --no-rhsm --enablerepo repository_id1 --enablerepo repository_id2 ...
       
       Example:
       # leapp upgrade --no-rhsm --enablerepo baseos --enablerepo appstream
       
9. Reboot the system, Leapp upgrades all packages and automatically reboots to the RHEL 8 system, it will take about 70 minutes on this phase.

10. Once the system can be accessed, verifying the system state.

        # cat /etc/redhat-release
        Red Hat Enterprise Linux release 8.2 (Ootpa)
        
        # uname -r
        4.18.0-193.el8.x86_64

11. Perform the additional packages update

        # yum update --bugfix --enhancement --security
        
12. Update SSCA DB with the new OS version information if the node was registered in SSCA.

## References

<a href="https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/upgrading_from_rhel_7_to_rhel_8/index" target="_blank">UPGRADING FROM RHEL 7 TO RHEL 8</a>  
<a href="https://www.redhat.com/en/blog/upgrading-rhel-7-rhel-8-leapp-and-boom" target="_blank">Upgrading from RHEL 7 to RHEL 8 with Leapp and BOOM</a>  

> **Copyright Notice**: Original work by this blog’s author, licensed under <a href="https://creativecommons.org/licenses/by-nc-nd/4.0/" target="_blank">CC BY-NC-ND 4.0</a>. When reposting, please include a link to the original source and this copyright notice.