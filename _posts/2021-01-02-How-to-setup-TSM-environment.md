---
title:  How to setup a TSM environment
date: 2021-01-02 14:51:20
categories:
  - Setup
tags:
  - ptf
  - spectrum protect
  - tsm
  - hsm
  - mmimgbackup
---

Some GPFS command like mmimgbackup will call HSM command dsmc, so we should setup a TSM environment for some PTF testing.

This article is to deploy Spectrum Protect 8.1.11 using Single-site disk solution on an x86_64 Linux server.

## Prerequisites

- Download the specific install package from it2n01:/u/tsm or <a href="https://g03zcdwas001.ahe.boulder.ibm.com/software/xl/download/ticket.wss" target="_blank">Internal DSW portal</a>  
- A node with enough free disks/space, can refer <a href="https://www.ibm.com/support/knowledgecenter/SSEQVQ_8.1.11/srv.solutions/r_ssdisk_worksheet.html" target="_blank">the planning worksheets for preconfiguration</a> to prepare the space.
- The node have installed GUI, can remote access GUI via VNC if can't physical access.
- The node have installed the required software, refer to <a href="https://www.ibm.com/support/knowledgecenter/SSEQVQ_8.1.11/srv.solutions/r_ssdisk_sysreqs_software.html" target="_blank">Software requirements</a> for more information.

## Steps to install/configure TSM server

### Create user and group id for the TSM server instance

- Run below commands to create user `tsminst1` and its group

      # groupadd tsmsrvrs -g 500
      # useradd -d /home/tsminst1 -m -u 500 -g 500 -s /bin/bash tsminst1
      # passwd tsminst1    # ppslab

### Extract and run the TSM installer tool

- Extract the downloaded binary file

      # chmod +x SP_8.1.11_LIN86_SERSTG_AGT_ML.bin
      # ./SP_8.1.11_LIN86_SERSTG_AGT_ML.bin
        inflating: repository/plugins/com.tivoli.dsm.stagent.moreinfo_8.1.11.202011031146.jar
        inflating: repository/repository.config
        inflating: repository/repository.xml
         creating: .input/
        inflating: .input/installnew32.xml
        inflating: .input/installnew64.xml
        inflating: .input/install_response.xml
         creating: input/
        inflating: input/install_response_sample.xml
        inflating: input/uninstall_response_sample.xml
        inflating: input/update_response_sample.xml
        inflating: im32/post-install.xml
        inflating: im64/post-install.xml
        inflating: README.htm

- Run the TSM installer tool

      # ./install.sh
      GUI failed to start. For more information see /tmp/install.569.log.
      Launching console mode...
      Preprocessing the input.
      Loading repositories...
      Preparing and resolving the selected packages...
      Preparing and resolving the selected packages...
      Preparing and resolving the selected packages...
      Preparing and resolving the selected packages...
      Preparing and resolving the selected packages...
      Preparing and resolving the selected packages...
      Preparing and resolving the selected packages...
      
      
      =====> IBM Installation Manager> Install
      
      Select packages to install:
           1. [X] IBM® Installation Manager 1.9.1.3
           2. [X] IBM Spectrum Protect server 8.1.11.20201103_1152
           3. [X] IBM Spectrum Protect languages 8.1.11.20201103_1149
           4. [X] IBM Spectrum Protect license 8.1.11.20201103_1148
           5. [X] IBM Spectrum Protect storage agent 8.1.11.20201103_1147
           6. [X] IBM Spectrum Protect device driver  8.1.11.20201103_1150
           7. [X] IBM Spectrum Protect Operations Center 8.1.11000.20200930_0721
      
           O. Check for Other Versions, Fixes, and Extensions
      
           N. Next,      C. Cancel
      -----> [N]
      Validating package prerequisites...
      ......
      
      
      =====> IBM Installation Manager> Install> Licenses
      
      Read the following license agreements carefully.
      View a license agreement by entering the number:
           1. IBM Installation Manager - License Agreement
      
      Options:
           A. [ ] I accept the terms in the license agreement
           D. [ ] I do not accept the terms in the license agreement
      
           B. Back,      C. Cancel
      -----> [C] A
      
      
      =====> IBM Installation Manager> Install> Licenses
      
      Read the following license agreements carefully.
      View a license agreement by entering the number:
           1. IBM Installation Manager - License Agreement
      
      Options:
           A. [X] I accept the terms in the license agreement
           D. [ ] I do not accept the terms in the license agreement
      
           B. Back,      N. Next,      C. Cancel
      -----> [N]
      
      
      =====> IBM Installation Manager> Install> Licenses> Shared Directory
      
      Installation Manager installation location:
              /opt/IBM/InstallationManager/eclipse
      
      Shared Resources Directory:
              /opt/IBM/IBMIMShared
      
      Options:
           L. Change Installation Manager Installation Location
           M. Change Shared Resources Directory
      
           B. Back,      N. Next,      C. Cancel
      -----> [N]
      Finding compatible package groups...
      
      
      =====> IBM Installation Manager> Install> Licenses> Shared Directory> Location
      
      New package group:
           1. [X] IBM Spectrum Protect
      
      Selected group id: "IBM Spectrum Protect"
      Selected location: "/opt/tivoli/tsm"
      Selected architecture: 64-bit
      
      Options:
           M. Change Location
      
           B. Back,      N. Next,      C. Cancel
      -----> [N]
      
      
      =====> IBM Installation Manager> Install> Licenses> Shared Directory>
        Location> Features
      
      IBM® Installation Manager
      
      IBM Spectrum Protect server
      
      IBM Spectrum Protect languages
           1. [ ] Spanish
           2. [ ] Portuguese
           3. [ ] German
           4. [ ] French
           5. [ ] Italian
           6. [ ] Russian
           7. [ ] Japanese
           8. [ ] Korean
           9. [ ] Traditional Chinese
          10. [ ] Simplified Chinese
      
      IBM Spectrum Protect license
      
      IBM Spectrum Protect storage agent
      
      IBM Spectrum Protect device driver
      
      IBM Spectrum Protect Operations Center
          11. [X] Operations Center
      
      
           B. Back,      N. Next,      C. Cancel
      -----> [N] 10
      
      
      =====> IBM Installation Manager> Install> Licenses> Shared Directory>
        Location> Features
      
      IBM® Installation Manager
      
      IBM Spectrum Protect server
      
      IBM Spectrum Protect languages
           1. [ ] Spanish
           2. [ ] Portuguese
           3. [ ] German
           4. [ ] French
           5. [ ] Italian
           6. [ ] Russian
           7. [ ] Japanese
           8. [ ] Korean
           9. [ ] Traditional Chinese
          10. [X] Simplified Chinese
      
      IBM Spectrum Protect license
      
      IBM Spectrum Protect storage agent
      
      IBM Spectrum Protect device driver
      
      IBM Spectrum Protect Operations Center
          11. [X] Operations Center
      
      
           B. Back,      N. Next,      C. Cancel
      -----> [N]
      
      =====> IBM Installation Manager> Install> Licenses> Shared Directory>
        Location> Features> Custom panels
      
      
      ---- Configuration for IBM Spectrum Protect Operations Center 8.1.11000.20200930_0721
      
      
      Specify the secure communication settings - Secure (https) port
      -----> [11090]
      
      ---- Create password
      
      
               Password length:
               - Minimum: 6 characters
               - Maximum: 64 characters
      
               The password must contain at least:
               - One uppercase letter (A - Z)
               - One lowercase letter (a - z)
               - One digit (0 - 9)
               - Two non-alphanumeric characters: ~ # $ % ^ @ * _ - + =  | ( ) { } [ ] : ; < > , . ? /
      
      Create password
      ----->
      
      Confirm password
      ----->
      
      ---- Configuration for IBM Spectrum Protect server 8.1.11.20201103_1152
      
      
      Select the product that you purchased:
           1. IBM Spectrum Protect
           2. IBM Spectrum Protect Extended Edition
           3. IBM Spectrum Protect for Data Retention
      
      -----> 1
      
      Read the following license agreements carefully.
      View a license agreement by entering the number:
           1. IBM Spectrum Protect - Software License Agreement
           2. IBM Spectrum Protect - Non-IBM Terms
      Options:
           A. [ ] I accept the terms in the license agreements.
           D. [ ] I do not accept the terms in the license agreements.
      
      -----> A
      Read the following license agreements carefully.
      View a license agreement by entering the number:
           1. IBM Spectrum Protect - Software License Agreement
           2. IBM Spectrum Protect - Non-IBM Terms
      Options:
           A. [X] I accept the terms in the license agreements.
           D. [ ] I do not accept the terms in the license agreements.
      
      
      ---- Configuration for IBM Spectrum Protect storage agent 8.1.11.20201103_1147
      
      
      Read the following license agreements carefully.
      View a license agreement by entering the number:
           1. IBM Spectrum Protect for Storage Area Networks - Software License Agreement
           2. IBM Spectrum Protect for Storage Area Networks - Non-IBM Terms
      Options:
           A. [ ] I accept the terms in the license agreements.
           D. [ ] I do not accept the terms in the license agreements.
      
      -----> A
      Read the following license agreements carefully.
      View a license agreement by entering the number:
           1. IBM Spectrum Protect for Storage Area Networks - Software License Agreement
           2. IBM Spectrum Protect for Storage Area Networks - Non-IBM Terms
      Options:
           A. [X] I accept the terms in the license agreements.
           D. [ ] I do not accept the terms in the license agreements.
      
      
      
           B. Back,      N. Next,      C. Cancel
      -----> [N]
      
      
      =====> IBM Installation Manager> Install> Licenses> Shared Directory>
        Location> Features> Custom panels> Summary
      
      Target Location:
        Package Group Name         :  IBM Installation Manager
        Installation Directory     :  /opt/IBM/InstallationManager/eclipse
        Package Group Name         :  IBM Spectrum Protect
        Installation Directory     :  /opt/tivoli/tsm
        Shared Resources Directory :  /opt/IBM/IBMIMShared
      
      Translations:
              English
      
      Packages to be installed:
              IBM® Installation Manager 1.9.1.3
              IBM Spectrum Protect server 8.1.11.20201103_1152
              IBM Spectrum Protect languages 8.1.11.20201103_1149
              IBM Spectrum Protect license 8.1.11.20201103_1148
              IBM Spectrum Protect storage agent 8.1.11.20201103_1147
              IBM Spectrum Protect device driver  8.1.11.20201103_1150
              IBM Spectrum Protect Operations Center 8.1.11000.20200930_0721
      
      Options:
           G. Generate an Installation Response File
      
           B. Back,      I. Install,      C. Cancel
      -----> [I] G
      
      =====> IBM Installation Manager> Install> Licenses> Shared Directory>
        Location> Features> Custom panels> Summary> Save response file
      
      Enter a directory path and a file name to generate the response file. Use .xml as the file extension. If a file name is specified without a full directory path, the response file is saved to /tmp/tsm. To skip the generation of a response file, press Enter.
      -----> response.xml
      
      =====> IBM Installation Manager> Save response file
      
      Response file was generated successfully: response.xml
      
           O. OK
      -----> [O]
      
      
      =====> IBM Installation Manager> Install> Licenses> Shared Directory>
        Location> Features> Custom panels> Summary
      
      Target Location:
        Package Group Name         :  IBM Installation Manager
        Installation Directory     :  /opt/IBM/InstallationManager/eclipse
        Package Group Name         :  IBM Spectrum Protect
        Installation Directory     :  /opt/tivoli/tsm
        Shared Resources Directory :  /opt/IBM/IBMIMShared
      
      Translations:
              English
      
      Packages to be installed:
              IBM® Installation Manager 1.9.1.3
              IBM Spectrum Protect server 8.1.11.20201103_1152
              IBM Spectrum Protect languages 8.1.11.20201103_1149
              IBM Spectrum Protect license 8.1.11.20201103_1148
              IBM Spectrum Protect storage agent 8.1.11.20201103_1147
              IBM Spectrum Protect device driver  8.1.11.20201103_1150
              IBM Spectrum Protect Operations Center 8.1.11000.20200930_0721
      
      Options:
           G. Generate an Installation Response File
      
           B. Back,      I. Install,      C. Cancel
      -----> [I]
                       25%                50%                75%                100%
      ------------------|------------------|------------------|------------------|
      ............................................................................
      
      =====> IBM Installation Manager> Install> Licenses> Shared Directory>
        Location> Features> Custom panels> Summary> Completion
      
      The install completed successfully.
      
      INFORMATION: Multiple informations are generated.
      
           V. View Message Details
      
      Options:
           F. Finish
      -----> [F] V
      
      =====> IBM Installation Manager> Information
      
      Multiple informations are generated.
      
      Details:
      To learn about best practices for configuring, monitoring, and operating an IBM Spectrum Protect solution, go to IBM Knowledge Center:   http://www.ibm.com/support/knowledgecenter/SSEQVQ/landing/welcome_sseqvq.htm   Search for IBM Spectrum Protect data protection solutions.
      
      The IBM Spectrum Protect Operations Center is now installed.  You can access it at the following URL:
      
      https://c550f6u13.pok.stglabs.ibm.com:11090/oc
      
           O. OK,      D. Hide Details
      -----> [O]
      
      =====> IBM Installation Manager> Install> Licenses> Shared Directory>
        Location> Features> Custom panels> Summary> Completion
      
      The install completed successfully.
      
      INFORMATION: Multiple informations are generated.
      
           V. View Message Details
      
      Options:
           F. Finish
      -----> [F]

### Configure TSM server instance with GUI

- Launch IBM Spectrum Protect Instance Configuration Wizard via VNC viewer

      # /opt/tivoli/tsm/server/bin/dsmicfgx
      Preparing to install
      Extracting the JRE from the installer archive...
      Unpacking the JRE...
      Extracting the installation resources from the installer archive...
      Configuring the installer for this system's environment...

      Launching installer...

- Specify the directories for TSM file systems in the GUI

      Instance User ID
      Instance Directory
      Database Directories
      Recovery Log Directories:
        Active log directory
        Primary archive log directory
        Active log mirror directory
        Secondary archive log directory

      Recovery Log Dir
      Recovery Log Archive Dir
      Recovery Log Archive Failover Dir

## Registering Licenses and configuring the TSM server

### Registering Licenses

- Accessing the TSM server

  ```
  # dsmadmc
  IBM Spectrum Protect
  Command Line Administrative Interface - Version 8, Release 1, Level 11.28
  (c) Copyright by IBM Corporation and other(s) 1990, 2020. All Rights Reserved.

  Enter your user id:  tsmadmin

  Enter your password:

  Session established with server C550F6U13: Linux/x86_64
    Server Version 8, Release 1, Level 11.000
    Server date/time: 12/31/2020 05:10:03  Last access: 12/31/2020 02:29:57
  
  Protect: C550F6U13>q license

                                                Last License Audit: 12/31/2020 02:18:25
                       Number of Data Protection for Oracle in use: 0
              Number of Data Protection for Oracle in try buy mode: 0
                Number of Data Protection for Microsoft SQL in use: 0
       Number of Data Protection for Microsoft SQL in try buy mode: 0
           Number of Data Protection for Microsoft Exchange in use: 0
  Number of Data Protection for Microsoft Exchange in try buy mode: 0
                              Number of TDP for Lotus Notes in use: 0
                     Number of TDP for Lotus Notes in try buy mode: 0
                 Number of Data Protection for Lotus Domino in use: 0
        Number of Data Protection for Lotus Domino in try buy mode: 0
                                 Number of TDP for Informix in use: 0
                        Number of TDP for Informix in try buy mode: 0
                                  Number of TDP for SAP R/3 in use: 0
                         Number of TDP for SAP R/3 in try buy mode: 0
                                      Number of TDP for ESS in use: 0
                             Number of TDP for ESS in try buy mode: 0
                                  Number of TDP for ESS R/3 in use: 0
                         Number of TDP for ESS R/3 in try buy mode: 0
                            Number of TDP for EMC Symmetrix in use: 0
                   Number of TDP for EMC Symmetrix in try buy mode: 0
                        Number of TDP for EMC Symmetrix R/3 in use: 0
               Number of TDP for EMC Symmetrix R/3 in try buy mode: 0
                                      Number of TDP for WAS in use: 0
                             Number of TDP for WAS in try buy mode: 0
               Is IBM Spectrum Protect for Data Retention in use ?: No
             Is IBM Spectrum Protect for Data Retention licensed ?: No
                      Is IBM Spectrum Protect Basic Edition in use: Yes
                    Is IBM Spectrum Protect Basic Edition licensed: No
                   Is IBM Spectrum Protect Extended Edition in use: No
                 Is IBM Spectrum Protect Extended Edition licensed: No
                                         Server License Compliance: FAILED
     ```

- Registering three Licenses

  ```
  Protect: C550F6U13>REGI LICENSE FILE=dataret.lic
  ANR2852I Current license information:
  ANR2853I New license information:
  ANR2828I Server is licensed to support IBM Spectrum Protect for Data Retention.
     
  Protect: C550F6U13>REGI LICENSE FILE=tsmbasic.lic
  ANR2852I Current license information:
  ANR2828I Server is licensed to support IBM Spectrum Protect for Data Retention.
  ANR2853I New license information:
  ANR2828I Server is licensed to support IBM Spectrum Protect for Data Retention.
  ANR2828I Server is licensed to support IBM Spectrum Protect Basic Edition.
     
  Protect: C550F6U13>REGI LICENSE FILE=tsmee.lic
  ANR2852I Current license information:
  ANR2828I Server is licensed to support IBM Spectrum Protect for Data Retention.
  ANR2828I Server is licensed to support IBM Spectrum Protect Basic Edition.
  ANR2853I New license information:
  ANR2828I Server is licensed to support IBM Spectrum Protect for Data Retention.
  ANR2828I Server is licensed to support IBM Spectrum Protect Basic Edition.
  ANR2828I Server is licensed to support IBM Spectrum Protect Extended Edition.
     
  Protect: C550F6U13>q license
     
                                                Last License Audit: 12/31/2020 02:18:25
                       Number of Data Protection for Oracle in use: 0
              Number of Data Protection for Oracle in try buy mode: 0
                Number of Data Protection for Microsoft SQL in use: 0
       Number of Data Protection for Microsoft SQL in try buy mode: 0
           Number of Data Protection for Microsoft Exchange in use: 0
  Number of Data Protection for Microsoft Exchange in try buy mode: 0
                              Number of TDP for Lotus Notes in use: 0
                     Number of TDP for Lotus Notes in try buy mode: 0
                 Number of Data Protection for Lotus Domino in use: 0
        Number of Data Protection for Lotus Domino in try buy mode: 0
                                 Number of TDP for Informix in use: 0
                        Number of TDP for Informix in try buy mode: 0
                                  Number of TDP for SAP R/3 in use: 0
                         Number of TDP for SAP R/3 in try buy mode: 0
                                      Number of TDP for ESS in use: 0
                             Number of TDP for ESS in try buy mode: 0
                                  Number of TDP for ESS R/3 in use: 0
                         Number of TDP for ESS R/3 in try buy mode: 0
                            Number of TDP for EMC Symmetrix in use: 0
                   Number of TDP for EMC Symmetrix in try buy mode: 0
                        Number of TDP for EMC Symmetrix R/3 in use: 0
               Number of TDP for EMC Symmetrix R/3 in try buy mode: 0
                                      Number of TDP for WAS in use: 0
                             Number of TDP for WAS in try buy mode: 0
               Is IBM Spectrum Protect for Data Retention in use ?: No
             Is IBM Spectrum Protect for Data Retention licensed ?: Yes
                      Is IBM Spectrum Protect Basic Edition in use: Yes
                    Is IBM Spectrum Protect Basic Edition licensed: Yes
                   Is IBM Spectrum Protect Extended Edition in use: No
                 Is IBM Spectrum Protect Extended Edition licensed: Yes
                                         Server License Compliance: Valid
  ```

### Update node parameters for GPFS and add space for the TSM database

- Update node parameters for GPFS

  ```
  Protect: C550F6U13>update node C550F6U28 PASSExp=0 MAXNUMMP=256 TXNGroupmax=32767
  ```

- Add space for the TSM database if needed

  ```
  Protect: C550F6U13>define vol backuppool /tsm/TSMfile02/bakfile numberofvolumes=46 formatsize=10000
  ```

### Configure the TSM server's policy

- Check policy

  ```
  Protect: C550F6U13>q mgmt f=d
    
              Policy Domain Name: STANDARD
                 Policy Set Name: ACTIVE
                 Mgmt Class Name: STANDARD
            Default Mgmt Class ?: Yes
                     Description: Installed default management class.
      Space Management Technique: None
         Auto-Migrate on Non-Use: 0
      Migration Requires Backup?: Yes
           Migration Destination: SPACEMGPOOL
  Last Update by (administrator): SERVER_CONSOLE
           Last Update Date/Time: 12/31/2020 02:06:43
                Managing profile:
                 Changes Pending: No
    
              Policy Domain Name: STANDARD
                 Policy Set Name: STANDARD
                 Mgmt Class Name: STANDARD
            Default Mgmt Class ?: Yes
                     Description: Installed default management class.
      Space Management Technique: None
         Auto-Migrate on Non-Use: 0
      Migration Requires Backup?: Yes
           Migration Destination: SPACEMGPOOL
  Last Update by (administrator): SERVER_CONSOLE
  more...   (<ENTER> to continue, 'C' to cancel)
    
           Last Update Date/Time: 12/31/2020 02:06:43
                Managing profile:
                 Changes Pending: No
    ```

- Update policy

  ```
  Protect: C550F6U13>update mgmt standard standard standard SPACEMGTECHnique=AUTO MIGREQUIRESBKUP=no
  ANR1522I Management class STANDARD updated in policy domain STANDARD, set STANDARD.
    
  Protect: C550F6U13>q mgmt f=d
    
              Policy Domain Name: STANDARD
                 Policy Set Name: ACTIVE
                 Mgmt Class Name: STANDARD
            Default Mgmt Class ?: Yes
                     Description: Installed default management class.
      Space Management Technique: None
         Auto-Migrate on Non-Use: 0
      Migration Requires Backup?: Yes
           Migration Destination: SPACEMGPOOL
  Last Update by (administrator): SERVER_CONSOLE
           Last Update Date/Time: 12/31/2020 02:06:43
                Managing profile:
                 Changes Pending: No
    
              Policy Domain Name: STANDARD
                 Policy Set Name: STANDARD
                 Mgmt Class Name: STANDARD
            Default Mgmt Class ?: Yes
                     Description: Installed default management class.
      Space Management Technique: Automatic
         Auto-Migrate on Non-Use: 0
      Migration Requires Backup?: No
           Migration Destination: SPACEMGPOOL
  Last Update by (administrator): TSMADMIN
  more...   (<ENTER> to continue, 'C' to cancel)
    
           Last Update Date/Time: 12/31/2020 09:36:40
                Managing profile:
                 Changes Pending: Yes
  ```

- Activate policy

  ```
  Protect: C550F6U13>activate pol STANDARD STANDARD
    
  Do you wish to proceed? (Yes (Y)/No (N)) Y
  ANR1514I Policy set STANDARD activated in policy domain STANDARD.
    
  Protect: C550F6U13>q mgmt f=d
    
              Policy Domain Name: STANDARD
                 Policy Set Name: ACTIVE
                 Mgmt Class Name: STANDARD
            Default Mgmt Class ?: Yes
                     Description: Installed default management class.
      Space Management Technique: Automatic
         Auto-Migrate on Non-Use: 0
      Migration Requires Backup?: No
           Migration Destination: SPACEMGPOOL
  Last Update by (administrator): TSMADMIN
           Last Update Date/Time: 12/31/2020 09:36:40
                Managing profile:
                 Changes Pending: No
    
              Policy Domain Name: STANDARD
                 Policy Set Name: STANDARD
                 Mgmt Class Name: STANDARD
            Default Mgmt Class ?: Yes
                     Description: Installed default management class.
      Space Management Technique: Automatic
         Auto-Migrate on Non-Use: 0
      Migration Requires Backup?: No
           Migration Destination: SPACEMGPOOL
  Last Update by (administrator): TSMADMIN
  more...   (<ENTER> to continue, 'C' to cancel)
    
           Last Update Date/Time: 12/31/2020 09:36:40
                Managing profile:
                 Changes Pending: No
  ```

## Steps to install/configure TSM client

1. Locate the specific version from it2n01:/u/tsm/tsm.client, find the relevant packages.

       For X86 node:
       # ls /u/tsm/tsm.client/8.1.11/X86/HSM/*.rpm
       gskcrypt64-8.0.55.17.linux.x86_64.rpm  TIVsm-API64.x86_64.rpm   TIVsm-BAcit.x86_64.rpm  TIVsm-HSM.x86_64.rpm
       gskssl64-8.0.55.17.linux.x86_64.rpm    TIVsm-APIcit.x86_64.rpm  TIVsm-BA.x86_64.rpm     TIVsm-WEBGUI.x86_64.rpm
       
2. Install all above packages except TIVsm-WEBGUI (optional), with **SCOUTFREE** mode.

       Examples for X86 node:
       # cd /u/tsm/tsm.client/8.1.11/X86/HSM/
       # export HSMINSTALLMODE=SCOUTFREE
       # rpm -ivh gsk*.rpm TIVsm-API*.rpm TIVsm-BA*.rpm TIVsm-HSM*.rpm

3. Login the Spectrum Protect Operations Center GUI(https://<TSM_server>:11090/oc/), add the client if needed.

4. Configure dsm.opt and dsm.sys files(under _/opt/tivoli/tsm/client/ba/bin_) on your environment.
       
       Examples:
       # cat /opt/tivoli/tsm/client/ba/bin/dsm.sys
       SErvername  tsms_c550f6u13
          COMMMethod         TCPip
          TCPPort            1500
          TCPServeraddress   c550f6u13.pok.stglabs.ibm.com
          NODENAME           C550F5U28
          passwordaccess     generate
          errorlogname       /opt/tivoli/tsm/client/ba/bin/dsmerror.log
          errorlogretention  5 d
          HSMLOGNAME         /tmp/hsm/hsm.log
          HSMLOGEVENTFLAGS   FILE
          SSL                YES
       
       # cat /opt/tivoli/tsm/client/ba/bin/dsm.opt
       HSMENABLEIMMEDIATEMIGRATE YES
       HSMDISABLEAUTOMIGDAEMONS  yes
       HSMGROUPEDMIGRATE YES
       HSMEXTOBJIDATTR YES
       clusternode yes

5. Import the TSM server certificate for SSL

       # dsmcert -add -server <TSM_server> -file <path_of_cert256.arm>
       
       Example:
       # dsmcert -add -server c550f6u13 -file ~/cert256.arm
       IBM Spectrum Protect
       dsmcert utility
         dsmcert Version 8, Release 1, Level 11.28 20200928A
         dsmcert date/time: 12/31/2020 22:28:48
        (c) Copyright by IBM Corporation and other(s) 1990, 2020. All Rights Reserved.

       Result : Success

6. Start the service then add the file system for testing

       # dsmmigfs start
       # dsmmigfs Add <file_system>
        
       Example:
       # dsmmigfs Add /gpfs/fs1
       
7. If you want to delete the node which installed TSM client, might need to delete TSM client packages firstly.

## Run Backup testing from client node

- Perform backup testing from client(for example c550f6u28 here)

  ```
  [root@c550f6u28 ~]# mmimgbackup fs1 -S snap1
  mmimgbackup: [I] Image backup of /dev/fs1 begins at Thu Dec 31 22:31:03 EST 2020.
  [I] GPFS Current Data Pool Utilization in KB and %
  Pool_Name          	    KB_Occupied	       KB_Total	 Percent_Occupied
  system              	        1646592	      975585280	     0.168779914%
  [I] 131129 of 268224 inodes used: 48.887870%.
  [I] Loaded policy rules from /fs1/.imgbackup/imgbackup_1229/.imagePolicy.
  Evaluating policy rules with CURRENT_TIMESTAMP = 2021-01-01@03:31:03 UTC
  Parsed 2 policy rules.
  /* Automatically generated policy file for image backup */
  RULE 'xl-d'  EXTERNAL LIST xl EXEC ''
  RULE 'xl-m'  LIST xl directories_plus
     
  [I] 2021-01-01@03:31:03.945 Directory entries scanned: 127075.
  [I] Directories scan: 127052 files, 23 directories, 0 other objects, 0 'skipped' files and/or errors.
  [I] 2021-01-01@03:31:04.055 Sorting 127075 file list records.
  [I] Source: fs "fs1" snap "snap1" path "/fs1/.snapshots/snap1".  \.......
  [I] Inodes scan: 127052 files, 23 directories, 0 other objects, 0 'skipped' files and/or errors.
  [I] 2021-01-01@03:31:12.147 Policy evaluation. 127075 files scanned.
  [I] The image file writer terminated with ec:0
  [I] Summary: 127075 inodes scanned, 127075 records written, 0 n/a, 0 errors.
  mmimgbackup: [I] Image backup of /dev/fs1 ends at Thu Dec 31 22:31:12 EST 2020.
  IBM Spectrum Protect
  Command Line Backup-Archive Client Interface
    Client Version 8, Release 1, Level 11.28 20200928A
    Client date/time: 12/31/2020 22:31:12
  (c) Copyright by IBM Corporation and other(s) 1990, 2020. All Rights Reserved.
     
  Node Name: C550F6U28
  Please enter your user id <C550F6U28>: tsmadmin
     
  Please enter password for user id "TSMADMIN":
     
  Session established with server C550F6U13: Linux/x86_64
    Server Version 8, Release 1, Level 11.000
    Server date/time: 12/31/2020 22:31:13  Last access: 12/31/2020 22:22:54
     
  Selective Backup function invoked.
     
  Directory-->               4,096 /fs1/.imgbackup/imgbackup_1229 [Sent]
  Directory-->               4,096 /fs1/.imgbackup [Sent]
  Directory-->               4,096 /fs1/.imgbackup/imgbackup_1229/mmPolicy.1312.6E0AE049 [Sent]
  Normal File-->               136 /fs1/.imgbackup/imgbackup_1229/.imagePolicy [Sent]
  Normal File-->        44,805,896 /fs1/.imgbackup/imgbackup_1229/mmPolicy.1312.6E0AE049/ImageArchive_20210101_03.31.03_000.sbr [Sent]
  Normal File-->             1,088 /fs1/.imgbackup/imgbackup_1229/mmPolicy.1312.6E0AE049/ImageArchive_20210101_03.31.03.idx [Sent]
  Selective Backup processing of '/fs1/.imgbackup/imgbackup_1229/*' finished without failure.
     
     
  Total number of objects inspected:            6
  Total number of objects backed up:            6
  Total number of objects updated:              0
  Total number of objects rebound:              0
  Total number of objects deleted:              0
  Total number of objects expired:              0
  Total number of objects failed:               0
  Total number of objects encrypted:            0
  Total number of objects grew:                 0
  Total number of retries:                      0
  Total number of bytes inspected:          42.74 MB
  Total number of bytes transferred:        42.73 MB
  Data transfer time:                        0.60 sec
  Network data transfer rate:           72,190.58 KB/sec
  Aggregate data transfer rate:         41,586.12 KB/sec
  Objects compressed by:                        0%
  Total data reduction ratio:                0.03%
  Elapsed processing time:               00:00:01
  mmimgbackup: Successful backup up image files from directory /fs1/.imgbackup/imgbackup_1229.
  ```

## References

<a href="https://www.ibm.com/support/knowledgecenter/SSEQVQ" target="_blank">IBM Spectrum Protect</a> from Knowledge Center  
<a href="https://tuna.pok.stglabs.ibm.com/wiki/Installing_and_configuring_TSM_server" target="_blank">Installing and configuring TSM server</a> from GpfsWiki  
<a href="https://www.ibm.com/support/pages/ibm-spectrum-protect-blueprints" target="_blank">IBM Spectrum Protect Blueprints</a>  
<a href="https://www.ibm.com/support/pages/node/84861" target="_blank">Overview - IBM Spectrum Protect Supported Operating Systems</a>  
<a href="https://www.ibm.com/support/pages/system/files/inline-files/IBM_Spectrum_Protect_HSM_Scale_Configuration_Guide.pdf" target="_blank">Deploying a Solution that Combines IBM Spectrum Protect for Space Management with IBM Spectrum Scale</a>  

> **Copyright Notice**: Original work by this blog’s author, licensed under <a href="https://creativecommons.org/licenses/by-nc-nd/4.0/" target="_blank">CC BY-NC-ND 4.0</a>. When reposting, please include a link to the original source and this copyright notice.