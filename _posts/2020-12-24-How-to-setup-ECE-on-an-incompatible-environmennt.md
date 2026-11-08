---
title:  How to setup ECE on an incompatible environment
date: 2020-12-24 18:15:00
categories:
  - Setup
tags:
  - ptf
  - ece
  - tslsenclslot
  - recoverygroup
---

ECE configuration require SAS-attached HDD, SSD drives that use specified LSI adapters or enterprise-class NVMe drives, at least 25 Gbps bandwidth networking. Based on my previous testing in project MachineCity, it is more sensitive than NSD configuration.

Unfortunately, our lab might have few supported environment for testing, so have to setup ECE on an incompatible environment. 

## Prerequisites

- Passwordless SSH must be set up using the FQDN and the short name of the node.

- The following rpms are required to be installed:

  - sg3_utils
  - nvme-cli
  - storcli (if you use SAS drives with LSI HBA) 
  - dmidecode
  - PyYAML

## Steps to install ECE

1. Extract the installation package on the installer node ONLY, can be found under it2n01:/u/images/SPECTRUM_SCALE.

       # ./Spectrum_Scale_Erasure_Code-5.x.y.z-x86_64-Linux-install --silent

2. Specify the installer node and the setup type in the cluster definition file

       # cd /usr/lpp/mmfs/5.x.y.z/installer/
       # ./spectrumscale setup -s <InstallerNodeIP> -st ece

3. Add different role nodes for ECE in the cluster definition file

       # ./spectrumscale node add <NodeName> -q -m -so
       # ./spectrumscale node add <NodeName> -a -g
       # ./spectrumscale node add <NodeName> -a -m -so
       # ./spectrumscale node list
       
       Example:
       # ./spectrumscale node add c550f5u01 -q -m -so

4. Define the recovery group for ECE in the cluster definition file

       ## Only create one recoverygroup 
       # ./spectrumscale recoverygroup define -N <Node1,Node2,...,NodeN>
       
       ## Create more than one recoverygroups
       # ./spectrumscale recoverygroup define -rg <RGName> -nc <ScaleOutNodeClassName> -N <Node1,Node2,...,NodeN>
       
       Example:
       # ./spectrumscale recoverygroup define -rg rg_1 -nc nc_1 -N c550f5u01,c550f5u02,c550f5u03,c550f5u04
       
5. (Optionals) Configure cluster name, disable callhome and so on.

       # ./spectrumscale config gpfs -c <clusterName>
       # ./spectrumscale config gpfs --ephemeral_port_range 60000-61000
       # ./spectrumscale callhome disable
  
6. Perform environment prechecks before issuing install command

       # ./spectrumscale install --precheck --skip no-ece-check
       
7. Create recoverygroup using mmvdisk

       ## Before ECE 5.0.5.4
       # mmvdisk rg create --rg <RGName> --nc <RelevantNodeClassName> -v no
       
       ## Start from ECE 5.0.5.4, MUST turn off "strict pdisk slot location" checking firstly
       # echo 999 | mmchconfig nsdRAIDStrictPdiskSlotLocation=0 -i
       # mmvdisk rg create --rg <RGName> --nc <RelevantNodeClassName> -v no
       
       Example:
       # mmvdisk rg create --rg rg_1 --nc nc_1 -v no

8. Perform the installation toolkit installation procedure

       # ./spectrumscale install --skip no-ece-check
       
9. Using mmvdisk to create vdisk sets for additional parameters
  
       # mmvdisk vdiskset define --vdisk-set <vdisksetName> --recovery-group <RGName> --code <RaidCode> --block-size <BlockSize> --set-size <n%> --declustered-array <DAName> --nsd-usage <NSDUsage> --storage-pool <PoolName>
       # mmvdisk vdiskset create --vdisk-set all
       
       Examples:
       # mmvdisk vdiskset define --vdisk-set HDD-Data --recovery-group rg_1 --code 8+3p --block-size 16M --set-size 100% --declustered-array DA1 --nsd-usage dataonly --storage-pool data
       # mmvdisk vdiskset define --vdisk-set SSD-Meta --recovery-group rg_1 --code 8+3p --block-size 2M --set-size 100% --declustered-array DA2 --nsd-usage metadataonly --storage-pool system
       # mmvdisk vdiskset create --vdisk-set all
       Or
       # mmvdisk vdiskset define --vdisk-set HDD1-Data --recovery-group rg_1 --code 4+3p --block-size 4m --set-size 30% --declustered-array DA1 --nsd-usage dataonly --storage-pool data
       # mmvdisk vdiskset define --vdisk-set HDD2-Data --recovery-group rg_2 --code 4+3p --block-size 8m --set-size 70% --declustered-array DA1 --nsd-usage dataonly --storage-pool data
       # mmvdisk vdiskset define --vdisk-set SSD1-Meta --recovery-group rg_1 --code 4+2p --block-size 1M --set-size 50% --declustered-array DA2 --nsd-usage metadataonly --storage-pool system
       # mmvdisk vdiskset define --vdisk-set SSD2-Meta --recovery-group rg_1 --code 4+2p --block-size 2M --set-size 50% --declustered-array DA2 --nsd-usage metadataonly --storage-pool system
       # mmvdisk vdiskset create --vdisk-set all

    > RAID code '4+3p' requires 7 non-spare pdisks, 'mmvdisk vdiskset create' will be failed if not enough spare pdisks.
       
10. Using mmvdisk to create file system and mount it to all nodes

        # mmvdisk vdiskset create --file-system <FSName> --vdisk-set <VSName>
        # mmmount all -a
        
        Example:
        # mmvdisk filesystem create --file-system ptf_FS1 --vdisk-set SSD-Meta,HDD-Data --mmcrfs -T /ptf_FS1
        
## References

<a href="https://www.ibm.com/support/knowledgecenter/STXKQY_ECE_5.1.0/ibmspectrumscaleece510_welcome.html" target="_blank">The Knowledge Center of ECE 5.1.0</a>  
<a href="https://tuna.pok.stglabs.ibm.com/perseuswiki/Using_the_pdisktopology_user_exit#ESS_3200_temporary_facsimile_until_PEMS_is_available" target="_blank">Turn off "strict pdisk slot location" checking</a>  
<a href="https://www.spectrumscaleug.org/wp-content/uploads/2020/04/Spectrum-Scale-Erasure-Code-Edition-ECE.pdf" target="_blank">Spectrum Scale Erasure Code Edition</a> (From SSUG)  

> **Copyright Notice**: Original work by this blog’s author, licensed under <a href="https://creativecommons.org/licenses/by-nc-nd/4.0/" target="_blank">CC BY-NC-ND 4.0</a>. When reposting, please include a link to the original source and this copyright notice.