---
title:  How to access Juniper/Lenovo/Brocade switches' console using Mac
date: 2020-12-02 18:15:00
categories:
  - Setup
tags:
  - gpfs
  - mac
  - switch
  - ex4200
  - g7052
  - 2498-24e
  - vdx6730
---

There are some idle switches in SH lab, need to verify them working or not before relocation. But I don't have their passwords, so have to access their console and confirm.

## Preparation

- One Type-C to USB adapter  
- One USB to RJ45 adapter  
- One utility which can setup serial configuration(e.g. screen on MacOS or script)

## Usage

1. Connect the cables between Mac(Type-C port) and switch(RJ45 console port, not mgmt port).

2. Open Terminal on MacOS, issue the command [serial](https://github.ibm.com/lbosh/myscripts/blob/master/others/serial) (A script which uses command screen):
   ```
   > serial
   1: /dev/tty.usbserial-14430
   Choose serial device file > 1
   1
   1 : 115200 (default)
   2 : 9600
   3 : 57600
   Choose baudrate > 2
   ```
3. Now insert the power cable to the switch, it will start to boot up.

## Appendix

- The script serial content

      #!/bin/bash
      # From https://blog.csdn.net/huyuan7494/article/details/82381142

      portname="/dev/tty.SLAB_USBtoUART"
      preset="$1"
      baudrate=115200

      screen_start()
      {
        screen $portname $baudrate
      }


      if [ -n "$preset" ];then
        if [ "$preset" == "bst" ];then
          portname="/dev/tty.usbserial-A106HG1J"
          screen_start
        elif [ "$preset" == "gw" ];then
          portname="/dev/tty.usbserial-A106HG1J"
          baudrate=57600
          screen_start
        elif [ "$preset" == "macro" ];then
          screen_start
        else
          echo "Preset not found.\n"
        fi
        exit 0
      fi

      array=(`ls /dev/tty.* | grep -v "/dev/tty.SOC" | grep -v "/dev/tty.MALS" | grep -v "/dev/tty.Bluetooth-Incoming-Port"`)

      len=${#array[*]}
      i=0

      while [ $i -lt $len ]; do
      echo "$[$i+1]: ${array[$i]}"
      let i++
      done

      echo -n "Choose serial device file > "
      read selected
      #echo "PORT : $selected"

      if [ -z "$selected" ]
      then
       selected=1
      fi

      if echo $selected | grep "^[0-9]*$" >&1
      then
       if ((  $selected-1 > $len  ))
       then
         exit
       fi
        portname=${array[$selected-1]}
        echo "1 : 115200 (default)"
        echo "2 : 9600"
        echo "3 : 57600"
        echo -n "Choose baudrate > "
        read baudrate
        if [ -z "$baudrate" ]; then
          baudrate=115200
        elif [ "$baudrate" == "1" ]; then
          baudrate=115200
        elif [ "$baudrate" == "2" ]; then
          baudrate=9600
        elif [ "$baudrate" == "3" ]; then
          baudrate=57600
        fi
        screen_start
      fi
      
- Juniper EX4200 booting output

      U-Boot 1.1.6 (May 19 2010 - 05:03:13)

      Board: EX4200-24T 3.12
      EPLD:  Version 6.0 (0x85)
      DRAM:  Initializing (1024 MB)
      FLASH: 8 MB
      USB:   scanning bus for devices... 2 USB Device(s) found
             scanning bus for storage devices... 1 Storage Device(s) found

      ELF file is 32 bit
      Consoles: U-Boot console

      FreeBSD/PowerPC U-Boot bootstrap loader, Revision 2.2
      (builder@shoth.juniper.net, Wed May 19 06:04:27 UTC 2010)
      Memory: 1024MB
      Loading /boot/defaults/loader.conf
      /kernel data=0x8d8e8c+0x947cc syms=[0x4+0x7a1a0+0x4+0xad2a2]


      Hit [Enter] to boot immediately, or space bar for command prompt.
      Booting [/kernel]...
      Kernel entry at 0x800000e0 ...
      GDB: no debug ports present
      KDB: debugger backends: ddb
      KDB: current backend: ddb
      Copyright (c) 1996-2010, Juniper Networks, Inc.
      All rights reserved.
      Copyright (c) 1992-2006 The FreeBSD Project.
      Copyright (c) 1979, 1980, 1983, 1986, 1988, 1989, 1991, 1992, 1993, 1994
              The Regents of the University of California. All rights reserved.
      JUNOS 10.4R1.9 #0: 2010-12-04 09:57:12 UTC
          builder@warth.juniper.net:/volume/build/junos/10.4/release/10.4R1.9/obj-powerpc/bsd/sys/compile/JUNIPER-EX
      Timecounter "decrementer" frequency 50000000 Hz quality 0
      cpu0: Freescale e500v2 core revision 2.2
      cpu0: HID0 80004080<EMCP,TBEN,EN_MAS7_UPDATE>
      real memory  = 1048576000 (1000 MB)
      avail memory = 1026605056 (979 MB)
      ETHERNET SOCKET BRIDGE initialising
      Initializing EXSERIES platform properties ...
      nexus0: <PPC e500 Nexus device>
      ocpbus0: <on-chip peripheral bus> on nexus0
      openpic0: <OpenPIC in on-chip peripheral bus> iomem 0xfef40000-0xfef600b3 on ocpbus0
      memctl0: <mpc85xx memory error reporting> iomem 0xfef20000-0xfef20e5b,0xfef02000-0xfef02e5b irq 32,34 on ocpbus0
      i2c0: <MPC85XX OnChip i2c Controller> iomem 0xfef03000-0xfef03014 irq 59 on ocpbus0
      i2c1: <MPC85XX OnChip i2c Controller> iomem 0xfef03100-0xfef03114 irq 59 on ocpbus0
      uart0: <16550 or compatible> iomem 0xfef04500-0xfef0450f irq 58 on ocpbus0
      uart0: console (9600,n,8,1)
      uart1: <16550 or compatible> iomem 0xfef04600-0xfef0460f irq 58 on ocpbus0
      lbc0: <Freescale 8533 Local Bus Controller> iomem 0xfef05000-0xfef05fff,0xff000000-0xffffffff irq 22 on ocpbus0
      cfi0: <AMD/Fujitsu - 8MB> iomem 0xff800000-0xffffffff on lbc0
      syspld0 iomem 0xff000000-0xff00ffff on lbc0
      tsec0: <eTSEC ethernet controller> iomem 0xfef24000-0xfef24fff irq 45,46,50 on ocpbus0
      tsec0: hardware MAC address 5c:5e:ab:74:d8:7f
      miibus0: <MII bus> on tsec0
      e1000phy0: <Marvell 88E1112 Gigabit PHY> on miibus0
      e1000phy0:  10baseT, 10baseT-FDX, 100baseTX, 100baseTX-FDX, 1000baseTX-FDX, auto
      pcib0: <Freescale MPC8544 PCI host controller> iomem 0xfef08000-0xfef08fff,0xf0000000-0xf3ffffff on ocpbus0
      pci0: <PCI bus> on pcib0
      pci0: <serial bus, USB> at device 18.0 (no driver attached)
      ehci0: <Philips ISP156x USB 2.0 controller> mem 0xf0001000-0xf00010ff irq 22 at device 18.2 on pci0
      usb0: EHCI version 1.0
      usb0: <Philips ISP156x USB 2.0 controller> on ehci0
      usb0: USB revision 2.0
      uhub0: Philips EHCI root hub, class 9/0, rev 2.00/1.00, addr 1
      uhub0: 2 ports with 2 removable, self powered
      umass0: STMicroelectronics ST72682  High Speed Mode, rev 2.00/2.10, addr 2
      pcib1: <Freescale MPC8544 PCI Express host controller> iomem 0xfef0a000-0xfef0afff,0xe0000000-0xe3ffffff,0xec000000-0xec0fffff irq 42 on ocpbus0
      pci1: <PCI bus> on pcib1
      pcib2: <PCI-PCI bridge> at device 0.0 on pci1
      pci2: <PCI bus> on pcib2
      mpfe0: <Juniper EX-series Packet Forwarding Engine> mem 0xa4000000-0xa40fffff,0xa0000000-0xa3ffffff irq 19 at device 0.0 on pci2
      pcib3: <Freescale MPC8544 PCI Express host controller> iomem 0xfef0b000-0xfef0bfff,0xe8000000-0xebffffff,0xec200000-0xec2fffff irq 43 on ocpbus0
      pci3: <PCI bus> on pcib3
      pcib4: <PCI-PCI bridge> at device 0.0 on pci3
      pci4: <PCI bus> on pcib4
      mpfe1: <Juniper EX-series Packet Forwarding Engine> mem 0xb4000000-0xb40fffff,0xb0000000-0xb3ffffff irq 18 at device 0.0 on pci4
      idma0: <mp85xxx DMA Controller> iomem 0xfef21000-0xfef21300 irq 36 on ocpbus0
      Initializing product: 41 ..
      bmeb: bmeb_lib_init done 0x82a98800, addr 0x808db444
      bme0:Virtual BME driver initializing
      Timecounters tick every 1.000 msec
      Loading the NETPFE picpeer module
      IPsec: Initialized Security Association Processing.
      da0 at umass-sim0 bus 0 target 0 lun 0
      da0: <ST ST72682 2.10> Removable Direct Access SCSI-2 device
      da0: 40.000MB/s transfers
      da0: 1000MB (2048000 512 byte sectors: 64H 32S/T 1000C)
      Trying to mount root from ufs:/dev/da0s1a
      WARNING: / was not properly dismounted
      Attaching /packages/jbase via /dev/mdctl...
      Mounted jbase package on /dev/md0...
      Verified manifest signed by PackageProduction_10_4_0
      Verified jboot signed by PackageProduction_10_4_0
      Verified jbase-10.4R1.9 signed by PackageProduction_10_4_0
      Mounted jcrypto-ex package on /dev/md1...
      Verified manifest signed by PackageProduction_10_4_0
      Verified jcrypto-ex-10.4R1.9 signed by PackageProduction_10_4_0
      Mounted jdocs-ex package on /dev/md2...
      Verified manifest signed by PackageProduction_10_4_0
      Verified jdocs-ex-10.4R1.9 signed by PackageProduction_10_4_0
      Mounted jkernel-ex package on /dev/md3...
      Verified manifest signed by PackageProduction_10_4_0
      Verified jkernel-ex-10.4R1.9 signed by PackageProduction_10_4_0
      Mounted jpfe-ex42x package on /dev/md4...
      Verified manifest signed by PackageProduction_10_4_0
      Verified jpfe-ex42x-10.4R1.9 signed by PackageProduction_10_4_0
      Mounted jroute-ex package on /dev/md5...
      Verified manifest signed by PackageProduction_10_4_0
      Verified jroute-ex-10.4R1.9 signed by PackageProduction_10_4_0
      Mounted jswitch-ex package on /dev/md6...
      Verified manifest signed by PackageProduction_10_4_0
      Verified jswitch-ex-10.4R1.9 signed by PackageProduction_10_4_0
      Mounted jweb-ex package on /dev/md7...
      Verified manifest signed by PackageProduction_10_4_0
      Verified jweb-ex-10.4R1.9 signed by PackageProduction_10_4_0
      Executing /packages/mnt/jweb-ex-10.4R1.9/mount.post..
      Automatic reboot in progress...
      ** Last Mounted on /
      ** Root file system
      ** Phase 1 - Check Blocks and Sizes
      ** Phase 2 - Check Pathnames
      ** Phase 3 - Check Connectivity
      ** Phase 4 - Check Reference Counts
      ** Phase 5 - Check Cyl groups
      1350 files, 63382 used, 30879 free (55 frags, 3853 blocks, 0.1% fragmentation)

      ***** FILE SYSTEM MARKED CLEAN *****
      ** Last Mounted on /config
      ** Phase 1 - Check Blocks and Sizes
      ** Phase 2 - Check Pathnames
      ** Phase 3 - Check Connectivity
      ** Phase 4 - Check Reference Counts
      ** Phase 5 - Check Cyl groups
      47 files, 50 used, 28277 free (13 frags, 3533 blocks, 0.0% fragmentation)

      ***** FILE SYSTEM MARKED CLEAN *****
      ** Last Mounted on /var
      ** Phase 1 - Check Blocks and Sizes
      ** Phase 2 - Check Pathnames
      ** Phase 3 - Check Connectivity
      ** Phase 4 - Check Reference Counts
      ** Phase 5 - Check Cyl groups
      384 files, 1475 used, 61324 free (180 frags, 7643 blocks, 0.3% fragmentation)

      ***** FILE SYSTEM MARKED CLEAN *****
      savecore: Router rebooting after a normal shutdown...
      No dump exists
      ** Last Mounted on /var/tmp
      ** Phase 1 - Check Blocks and Sizes
      ** Phase 2 - Check Pathnames
      ** Phase 3 - Check Connectivity
      ** Phase 4 - Check Reference Counts
      ** Phase 5 - Check Cyl groups
      11 files, 35 used, 160963 free (27 frags, 20117 blocks, 0.0% fragmentation)

      ***** FILE SYSTEM MARKED CLEAN *****
      Creating initial configuration...mgd: commit complete
      Setting initial options:  debugger_on_panic=NO debugger_on_break=NO.
      Starting optional daemons: .
      Doing initial network setup:.
      Initial interface configuration:
      additional daemons:.
      Additional routing options:kern.module_path: /boot//kernel;/boot/modules -> /boot//kernel;/boot/modules;/modules
       grat_arp_on_ifup=YES: net.link.ether.inet.grat_arp_on_ifup: 1 -> 1
      kld netpfe drv: ifpfed_etLoading the NETPFE ethernet module
      h ifpfed_ml_cmnLoading common multilink module.
      kld platform: ex_ifpfeLoading the EX-series platform NETPFE module
       if_vcpkern.module_path: /boot//kernel;/boot/modules;/modules -> /boot//kernel;/boot/modules;/modules;/modules/peertype
      kld peertype: peertype_hcm peertype_pfem peertype_sfi peertype_slavere ipsec kld.
      Doing additional network setup:.
      Starting final network daemons:.
      setting ldconfig path: /usr/lib /opt/lib
      starting standard daemons: cron.
      Local package initialization:.
      kern.securelevel: -1 -> 1
      starting local daemons:.
      Sun Apr 28 19:03:18 UTC 2013

      ex4200 (ttyu0)

      login:
      
- Lenovo G7052 booting output

      U-Boot 2009.06 (Apr 17 2015 - 07:32:53)

      CPU:   P1010, Version: 1.0, (0x80f10010)
      Core:  E500, Version: 5.1, (0x80212151)
      Clock Configuration:
             CPU0:800  MHz,
             CCB:400  MHz,
             DDR:400  MHz (800 MT/s data rate) (Asynchronous), IFC:100  MHz
      L1:    D-cache 32 kB enabled
             I-cache 32 kB enabled
      Board: Rack Switch G7052, CPLD: V5, PCBA: V2
      I2C:   ready
      DRAM:  Configuring DDR for 800 MT/s data rate
      512 MB

      Memory Test ..........

      Production Mode

      FLASH: 32 MB
      L2:    256 KB enabled
      NAND:  256 MiB
      PCIe1: Root Complex of mini PCIe Slot, x1, regs @ 0xffe0a000
      PCIe1: Bus 00 - 01
      PCIe2: Root Complex of PCIe Slot, x1, regs @ 0xffe09000
      PCIe2: Bus 02 - 03
      Net:   eTSEC2: No support for PHY id 0; assuming generic
      eTSEC1: No support for PHY id ffffffff; assuming generic
      eTSEC2 [PRIME], eTSEC1

      Booting OS


      Wed Dec  2 12:53:16 UTC 2020
      Booting image '1'

      Initializing Production Mode .....
      Early Initialization Done. (4 secs)
      256MB w/2048-128KB sectors
      PCI unit 0: Dev 0xb321, Rev 0x12, Chip BCM56321_B1, Driver BCM56334_B0
      PCI unit 1: Dev 0xb321, Rev 0x12, Chip BCM56321_B1, Driver BCM56334_B0


      Rebooted because of power cycle.

      Dec  2 12:53:38 0.0.0.0 CRIT    system: Redundant Power Supply not Installed
      Booting complete 12:53:50 Wed Dec  2, 2020:
        Version 8.1.1 from FLASH image1, factory default configuration.

      Lenovo RackSwitch G7052.


      Enter  password:
      Dec  2 12:53:50 192.168.49.50 INFO    system: booted version 8.1.1 from FLASH image1, factory default configuration
      Dec  2 12:53:50 192.168.49.50 INFO    system: Assigned default IP 192.168.49.50 to data interface
      Dec  2 12:53:50 192.168.49.50 INFO    system: Assigned default IP 192.168.50.50 to mgmt interface
      Dec  2 12:54:19 192.168.49.50 ALERT   system: coldStart
      
- IBM 2498-24E(Brocade type) booting output

      The system is coming up, please wait...


      U-Boot 1.1.3 (Oct 31 2012 - 09:39:36)

      CPU:   AMCC PowerPC 440EPx Rev. A at 660 MHz (PLB=132, OPB=66, EBC=33 MHz)
             No Security/Kasumi support
             I2C boot EEPROM enabled
             Bootstrap Option G - Boot ROM Location I2C (Addr 0x54)
             Internal PCI arbiter enabled, PCI async ext clock used
             32 kB I-Cache 32 kB D-Cache
      Board: Tomahawk
      I2C:   ready
      DRAM:  Initializing DDRSDRAM
      DDR Init: speed = 132000002
      DQS calibration - Window detected:
      max_passing_cases = 52
      wr_dqs_shift      = 95
      dll_dqs_delay_X   = 29
      dll_dqs_delay_X window = 4 - 55
      DDR0_09=0x00011d5f
      DDR0_22=0x00267f0b
      DDR0_17=0x1d006100
      DDR0_18=0x1d1d1d1d
      DDR0_19=0x1d1d1d1d
      ECC Enabling   DONE
      512 MB
      POST RAM test disabled.
      Now running in RAM - U-Boot at: 1fb30000
      trap_init : 0x0
      system inventory subsystem initialized
      FLASH: Tomahawk CFI Flash Driver Initialized
       4 MB
      ATA interface setup: io_base=0xf8f00000, port=0x3e0, ctl=0x7ec
      PCI:
      Skip our host bridge
              00  07  1657  0011  0280  18
      In:    serial
      Out:   serial
      Err:   serial
      Net:

      Checking system RAM - press any key to stop test

      Checking memory address: 00100000

      System RAM test using Default POST RAM Test succeeded.

      set_bootstatus: BS_LOAD_OS, platform_idx = 1

      Type run flash_nfs to mount root filesystem over NFS

      Hit ESC to stop autoboot:  0
      Map file at LBA sector 0xaab47
      ## Booting image at 00400000 ...
         Image Name:   Linux-2.6.14.2
         Image Type:   PowerPC Linux Multi-File Image (uncompressed)
         Data Size:    4269595 Bytes =  4.1 MB
         Load Address: 00000000
         Entry Point:  00000000
         Contents:
         Image 0:  3190918 Bytes =  3 MB
         Image 1:  1078663 Bytes =  1 MB
      ## Current stack ends at 0x1FAEEBE0 => set upper limit to 0x00C00000
      ## initrd at 0x0070B0D4 ... 0x0081265A (len=1078663=0x107587)
         Loading Ramdisk to 1f9e6000, end 1faed587 ... OK
       initrd_start = 1f9e6000, initrd_end = 1faed587
      ## Transferring control to Linux (at address 00000000) ...
      Installing Linux 2.6 Kernel
      Attempting to find a root file system on hda1...
      INIT: version 2.78 booting
      Bypassing firmware validation.
      INIT: Entering runlevel: 3
      Starting Fabric OS Services...
      FIPS off, RRD...
      Found 1(threshold 5) abnormal reboots within 3000 seconds window(threshold)
      rscmon_service: started thread!
      sys_chip_init: class 0 max 24 Done
      ethsw_ha_enabled = 1
      uptime: 4293888817; sup_qid: 0
      2020/12/17-11:59:21, [HAM-1004], 10532, CHASSIS, INFO, IBM_2498_24E, Processor rebooted - Reset.


      Fabric OS (Rack33_FC_1)

      Rack33_FC_1 console login: SNMP Research EMANATE/Lite Agent Version 16.2.0.9
      Copyright 1989-2006 SNMP Research, Inc.
      ki_state_set: redundant ACTIVATE for instance 1
      2020/12/17-11:59:46, [ZONE-1022], 10533, FID 128, INFO, Rack33_FC_1, The effective configuration has changed to snap20_1.
      All service instances on Active
      POST1: Started running Thu Dec 17 11:59:51 GMT 2020
      POST1: Test #1 - Running turboramtest

      *** CORE FILES WARNING (12/17/20 - 12:00:03 ) ***
      2 KBytes in  file(s)
      use "supportsave" command to upload


      *** FFDC FILES WARNING (12/17/20 - 12:00:03 ) ***
      4 KBytes in  file(s)
      use "supportSave" command to upload

      POST1: Script PASSED with exit status of 0 Thu Dec 17 12:00:04 GMT 2020 took (0:0:13)
      POST2: Started running Thu Dec 17 12:00:05 GMT 2020
      POST2: Test #1 - Running portloopbacktest (SERDES)
      POST2: Running diagshow
      POST2: Script PASSED with exit status of 0 Thu Dec 17 12:00:12 GMT 2020 took (0:0:7)
      2020/12/17-12:00:13, [BL-1000], 10534, CHASSIS, INFO, IBM_2498_24E, Initializing ports...
      2020/12/17-12:00:13, [BL-1001], 10535, CHASSIS, INFO, IBM_2498_24E, Port initialization completed.
      2020/12/17-12:00:30, [SNMP-1007], 10536, FID 128, INFO, Rack33_FC_1, The last fabric change happened at: Thu Dec 17 12:00:24 2020
      
- Brocade VDX6730(3759-C32) booting output

      The system is coming up, please wait...


      U-Boot 1.1.3 (Apr  2 2012 - 10:49:05)

      CPU:   8548, Version: 2.1, (0x80310021)
      Core:  E500, Version: 2.2, (0x80210022)
      Clock Configuration:
             CPU:1199 MHz, CCB: 399 MHz,
             DDR: 199 MHz, LBC:  49 MHz
      L1:    D-cache 32 kB enabled
             I-cache 32 kB enabled
      Board: Elara2F
      CPU Board Revision 255.198 (0xffc6)
          PCI2: disabled
      I2C:   ready
      DRAM:  initdram: Initializing DDRSDRAM
      initdram: Normal boot in progress
      dimm_spd_sdram 2001: memsize = 800
      initdram: Call ddr_enable_ecc dram size -2147483648
          DDR: 2048 MB
      POST RAM test disabled.
      Adjusted memory 0x7d000000 2097152000
      Now running in RAM - U-Boot at: 78f7e000
      trap_init : 0x0
      system inventory subsystem initialized
      FLASH:  4 MB
      L2 cache 512KB: enabled
      CPLD: Init complete
      ATA interface setup upm 256: ffdfc04, fffffc01, fffffc00, fffffc00
      PCI: gd->brcd_flags = 0, PCI init
      Skip our host bridge
              00  10  1657  0010  0280  1a
              00  14  1033  0035  0c03  1a
              00  14  1033  00e0  0c03  1a
              83  00  1657  001a  0280  3c
              82  01  10b5  8508  0604  1a
              82  02  10b5  8508  0604  1a
              82  03  10b5  8508  0604  1a
              82  04  10b5  8508  0604  1a
              81  00  10b5  8508  0604  1a
      In:    serial
      Out:   serial
      Err:   serial
      Net:
      ENET0: PHY is Broadcom BCM5461S 10/100/1000 BaseT PHY (2060c1)
      ENET1: PHY is Broadcom BCM5241 10/100 BaseT PHY (143bc31)
      
      board_init_r: Adjusted ram 2097152000 0x7d000000 2048000k
      Checking system RAM - press any key to stop test
      
      Checking memory address: 00100000
      
      System RAM test using Default POST RAM Test succeeded.
      
      set_bootstatus: BS_LOAD_OS, platform_idx = 13
      Hit ESC to stop autoboot:  0
      Map file at LBA sector 0x36803f
      do_bootm 171: load_addr 0x02000000 addr 0x00400000
      do_bootm 174: ## Booting image at 00400000 load_addr 0x02000000 ...
         Image Name:   Linux-2.6.14.2
         Image Type:   PowerPC Linux Multi-File Image (gzip compressed)
         Data Size:    2681131 Bytes =  2.6 MB
         Load Address: 00000000
         Entry Point:  00000000
         Contents:
         Image 0:  1604615 Bytes =  1.5 MB
         Image 1:  1076503 Bytes =  1 MB
         Uncompressing Multi-File Image ... do_bootm_linux 565: ## Current stack ends at 0x78F5CB30 => set upper limit to 0x00800000
      ## initrd at 0x00587C54 ... 0x0068E96A (len=1076503=0x106D17)
         Loading Ramdisk to 1fef9000, end 1ffffd17 ... OK
       initrd_start = 1fef9000, initrd_end = 1ffffd17
      ## Transferring control to Linux (at address 00000000) ...
      tlbcam_index=11
      mpc85xx_setup: Doing Pcie bridge setup
      Installing Linux 2.6 Kernel
      Attempting to find a root file system on hda1...
      INIT: version 2.78 booting
      Bypassing firmware validation.
      INIT: Entering runlevel: 3
      (none)
      Starting Fabric OS Services...
      Found 1(threshold 5) abnormal reboots within 3000 seconds window(threshold)
      CPLD INIT DONE
      elara2f: setting to MASTER as HA bit is not set
      elara2f: after reboot, reset enables are  pci reset en = 0x9 , sw reset en = 0x9,  pu reset en = 0x9 ha_assist 1 is 0x0
      sys_chip_init: class 0 max 24 Done
      sys_chip_init: class 1 max 24 Done
      sysdrv_config = 0x2
      VCS mode is ENABLED (VCS ID 1)
      VCS cluster mode is DISABLED
      eAnvil rev B found
      Starting Dcmd ...
      Running as a Daemon : 1
      Type                : 0
      Host Name           : VDX6730
      Host IP Address     :
           Port           : 3016
      Starting Services now...
      Running as a Daemon : 1
      Type                : 0
      Host Name           : VDX6730
      Host IP Address     :
           Port           : 5016
      Starting Services now...
      Dcmd is ready for HASM to be launched
      Ccmd is ready for HASM to be launched
      Waiting for HASM to be launched and get to a stage where DCMD can continue ...
       Global Fan Direction is 0
      HASM has been launched and now DCMD can continue processing ...
      HASM has been launched and now CCMD can continue processing ...
      max chip 1
      Thu Dec 17 12:25:10 GMT 2020 :: Triggering confd load phase1 after waiting for 5 secs
      licd_ctor
      : chassis(0)-dcm: Msg input: msg = RECOVER, size = 0
      : chassis(0)-dcm: recovery stage
      : chassis(0)-dcm: stage 1002
      : chassis(0)-dcm: message to be delivered to component
      : chassis(0)-ccm: Msg input: msg = RECOVER, size = 0
      : chassis(0)-ccm: recovery stage
      : chassis(0)-ccm: stage 1002
      : chassis(0)-ccm: message to be delivered to component
      : chassis(0)-dcm: Next recover (idx) stage: 1
      : chassis(0)-ccm: Next recover (idx) stage: 1
      : chassis(0)-dcm: Msg input: msg = RECOVER, size = 0
      : chassis(0)-dcm: recovery stage
      : chassis(0)-dcm: stage 1
      : chassis(0)-dcm: message to be delivered to component
      : chassis(0)-ccm: Msg input: msg = RECOVER, size = 0
      : chassis(0)-ccm: recovery stage
      : chassis(0)-ccm: stage 1
      : chassis(0)-ccm: message to be delivered to component
      : chassis(0)-dcm: Next recover (idx) stage: 2
      : chassis(0)-ccm: Next recover (idx) stage: 2
      : chassis(0)-dcm: Msg input: msg = RECOVER, size = 0
      : chassis(0)-dcm: recovery stage
      : chassis(0)-dcm: stage 2
      : chassis(0)-dcm: message to be delivered to component
      : chassis(0)-ccm: Msg input: msg = RECOVER, size = 0
      : chassis(0)-ccm: recovery stage
      : chassis(0)-ccm: stage 1001
      : chassis(0)-ccm: message to be delivered to component
      : chassis(0)-dcm: Next recover (idx) stage: 3
      : chassis(0)-ccm: Next recover (idx) stage: 6
      : chassis(0)-dcm: Msg input: msg = RECOVER, size = 0
      : chassis(0)-dcm: recovery stage
      : chassis(0)-dcm: stage 3
      : chassis(0)-dcm: message to be delivered to component
      : chassis(0)-ccm: Msg input: msg = RECOVER, size = 0
      : chassis(0)-ccm: recovery stage
      : chassis(0)-ccm: no stage 3
      : chassis(0)-dcm: Next recover (idx) stage: 4
      : chassis(0)-dcm: Msg input: msg = RECOVER, size = 0
      : chassis(0)-dcm: recovery stage
      : chassis(0)-dcm: stage 4
      : chassis(0)-dcm: message to be delivered to component
      : chassis(0)-ccm: Msg input: msg = RECOVER, size = 0
      : chassis(0)-ccm: recovery stage
      : chassis(0)-ccm: no stage 4
      : chassis(0)-dcm: Next recover (idx) stage: 5
      : chassis(0)-dcm: Msg input: msg = RECOVER, size = 0
      : chassis(0)-dcm: recovery stage
      : chassis(0)-dcm: stage 1001
      : chassis(0)-dcm: message to be delivered to component
      : chassis(0)-ccm: Msg input: msg = RECOVER, size = 0
      : chassis(0)-ccm: recovery stage
      : chassis(0)-ccm: no stage 5
      : chassis(0)-dcm: Next recover (idx) stage: 6
      Create ethernet switch instance 0
      HSL module initialization: VCS enabled, VCS ID 1
      trill_create: Create completed handle 0x50000000
      .
      SNMP Research EMANATE/Lite Agent Version 16.2.0.9
      Copyright 1989-2006 SNMP Research, Inc.
      Eld_trace_init success
      2020/12/17-12:25:05, [HAM-1004], 2,, INFO, chassis, Processor rebooted - Reset
      SFLOW successfully initialized as wave client************
      PCAP successfully initialized as wave client************
      MCAST_SS successfully initialized as wave client************
      Passed :: Dcm-Client Initialization passed from eld client...
      
       eld_master_init(): eld_sock_init() sock 14 success
      Passed :: Dcm-Client Initialization passed from mstp client...
      INFO   : Thu Dec 17 12:25:37 2020 : DcmClientInterface dcm_initialize_client_for_dual_daemons : cluster type = 1
      Initialized DCMD Client Successfully
      
      Network OS (VDX6730)
      
      
      VDX6730 console login: SSM successfully initialized as wave client************
      Rmon successfully initialized as wave client************
      IGMP successfully initialized as wave client************
      chmod: too few arguments
      Try `chmod --help' for more information.
      Passed :: Dcm-Client Initializat: ethsw(2)-dcm: ion passed from Msg input: msg = RECOVER, size = 0
      : ethsw(2)-dcm:
                      recovery stage
      : ethsw(2)-dcm: stage 1002
      : ethsw(2)-dcm: message to be delivered to component
      : ethsw(2)-ccm: Msg input: msg = RECOVER, size = 0
      : ethsw(2)-ccm: recovery stage
      : ethsw(2)-ccm: stage 1002
      : ethsw(2)-ccm: message to be delivered to component
      Passed :: Dcm-Cl: ethsw(2)-dcm: ient InitializatNext recover (idx) stage: 1
      ion passed from : ethsw(2)-ccm: l2sys client... Next recover (idx) stage: 1
      
      Passed :: Dcm-Client Initialization passed from dauthd client...
      NSM VCS mode is enabled domain id = 0
      MCT_TABLE AVL CREATEION FAILED
      MCT_TABLE AVL CREATEI: ethsw(2)-dcm: ON SUCCESS
      Msg input: msg = RECOVER, size = 0
      : ethsw(2)-dcm: recovery stage
      : ethsw(2)-dcm: stage 1
      : ethsw(2)-dcm: message to be delivered to component
      : ethsw(2)-ccm: Msg input: msg = RECOVER, size = 0
      : ethsw(2)-ccm: recovery stage
      : ethsw(2)-ccm: stage 1
      : ethsw(2)-ccm: message to be delivered to component
      : ethsw(2)-dcm: Next recover (idx) stage: 2
      : ethsw(2)-ccm: Next recover (idx) stage: 2
      : ethsw(2)-dcm: Msg input: msg = RECOVER, size = 0
      : ethsw(2)-dcm: recovery stage
      : ethsw(2)-dcm: stage 2
      : ethsw(2)-dcm: message to be delivered to component
      : ethsw(2)-ccm: Msg input: msg = RECOVER, size = 0
      : ethsw(2)-ccm: recovery stage
      : ethsw(2)-ccm: stage 1001
      : ethsw(2)-ccm: message to be delivered to component
      : ethsw(2)-dcm: Next recover (idx) stage: 5
      : ethsw(2)-ccm: Next recover (idx) stage: 8
      : ethsw(2)-dcm: Msg input: msg = RECOVER, size = 0
      : ethsw(2)-dcm: recovery stage
      : ethsw(2)-dcm: no stage 3
      : ethsw(2)-ccm: Msg input: msg = RECOVER, size = 0
      : ethsw(2)-ccm: recovery stage
      : ethsw(2)-ccm: no stage 3
      ki_state_set: re: ethsw(2)-dcm: dundant ACTIVATEMsg input: msg = RECOVER, size = 0
      : ethsw(2)-dcm:
                      2020/12/17-12:2recovery stage
      5:43, [SSMD-1300: ethsw(2)-dcm: ], 3,, INFO, VDXno stage 4
      6730,  CEEMap _v: ethsw(2)-ccm: cs_cee is createMsg input: msg = RECOVER, size = 0
      d with precedenc: ethsw(2)-ccm: e 100.
      recovery stage
      : ethsw(2)-ccm: no stage 4
      2020/12/17-12:25:43, [SSMD-1303], 4,, INFO, VDX6: ethsw(2)-dcm: 730,  CEEMap _vcMsg input: msg = RECOVER, size = 0
      s_cee priority g: ethsw(2)-dcm: roup 15.0 with wrecovery stage
      eight 0 is creat: ethsw(2)-dcm: ed with PFC enabstage 3
      led.
      : ethsw(2)-dcm: message to be delivered to component
      : ethsw(2)-ccm: Msg input: msg = RECOVER, size = 0
      : ethsw(2)-ccm: recovery stage
      : ethsw(2)-ccm: no stage 5
      : ethsw(2)-dcm: Next recover (idx) stage: 6
      : ethsw(2)-dcm: Msg input: msg = RECOVER, size = 0
      : ethsw(2)-dcm: recovery stage
      : ethsw(2)-dcm: stage 1001
      : ethsw(2)-dcm: message to be delivered to component
      : ethsw(2)-ccm: Msg input: msg = RECOVER, size = 0
      : ethsw(2)-ccm: recovery stage
      : ethsw(2)-ccm: no stage 6
      : ethsw(2)-dcm: Next recover (idx) stage: 8
      : ethsw(2)-dcm: Msg input: msg = RECOVER, size = 0
      : ethsw(2)-dcm: recovery stage
      : ethsw(2)-dcm: no stage 7
      : ethsw(2)-ccm: Msg input: msg = RECOVER, size = 0
      : ethsw(2)-ccm: recovery stage
      : ethsw(2)-ccm: no stage 7
      : fcsw(4)-dcm: Msg input: msg = RECOVER, size = 0
      : fcsw(4)-dcm: recovery stage
      : fcsw(4)-dcm: stage 1002
      : fcsw(4)-dcm: message to be delivered to component
      : fcsw(4)-ccm: Msg input: msg = RECOVER, size = 0
      : fcsw(4)-ccm: recovery stage
      : fcsw(4)-ccm: stage 1002
      : fcsw(4)-ccm: message to be delivered to component
      : fcsw(4)-dcm: Next recover (idx) stage: 1
      : fcsw(4)-ccm: Next recover (idx) stage: 1
      : fcsw(4)-dcm: Msg input: msg = RECOVER, size = 0
      : fcsw(4)-dcm: recovery stage
      : fcsw(4)-dcm: stage 1
      : fcsw(4)-dcm: message to be delivered to component
      : fcsw(4)-ccm: Msg input: msg = RECOVER, size = 0
      : fcsw(4)-ccm: recovery stage
      : fcsw(4)-ccm: stage 1
      : fcsw(4)-ccm: message to be delivered to component
      ki_state_set: re: fcsw(4)-dcm: dundant ACTIVATENext recover (idx) stage: 2
      : fcsw(4)-ccm:
                     ki_state_set: rNext recover (idx) stage: 6
      edundant ACTIVATE for instance 1
      : fcsw(4)-dcm: Msg input: msg = RECOVER, size = 0
      : fcsw(4)-dcm: recovery stage
      : fcsw(4)-dcm: stage 2
      : fcsw(4)-dcm: message to be delivered to component
      : fcsw(4)-ccm: Msg input: msg = RECOVER, size = 0
      : fcsw(4)-ccm: recovery stage
      : fcsw(4)-ccm: no stage 2
      : fcsw(4)-dcm: Next recover (idx) stage: 7
      : fcsw(4)-dcm: Msg input: msg = RECOVER, size = 0
      : fcsw(4)-dcm: recovery stage
      : fcsw(4)-dcm: no stage 3
      : fcsw(4)-ccm: Msg input: msg = RECOVER, size = 0
      : fcsw(4)-ccm: recovery stage
      : fcsw(4)-ccm: no stage 3
      : fcsw(4)-dcm: Msg input: msg = RECOVER, size = 0
      : fcsw(4)-dcm: recovery stage
      : fcsw(4)-dcm: no stage 4
      : fcsw(4)-ccm: Msg input: msg = RECOVER, size = 0
      : fcsw(4)-ccm: recovery stage
      : fcsw(4)-ccm: no stage 4
      : fcsw(4)-dcm: Msg input: msg = RECOVER, size = 0
      : fcsw(4)-dcm: recovery stage
      : fcsw(4)-dcm: no stage 5
      : fcsw(4)-ccm: Msg input: msg = RECOVER, size = 0
      : fcsw(4)-ccm: recovery stage
      : fcsw(4)-ccm: no stage 5
      : fcsw(4)-dcm: Msg input: msg = RECOVER, size = 0
      : fcsw(4)-dcm: recovery stage
      : fcsw(4)-dcm: no stage 6
      : fcsw(4)-ccm: Msg input: msg = RECOVER, size = 0
      : fcsw(4)-ccm: recovery stage
      : fcsw(4)-ccm: stage 1001
      : fcsw(4)-ccm: message to be delivered to component
      : fcsw(4)-ccm: Next recover (idx) stage: 16
      : fcsw(4)-dcm: Msg input: msg = RECOVER, size = 0
      : fcsw(4)-dcm: recovery stage
      : fcsw(4)-dcm: stage 3
      : fcsw(4)-dcm: message to be delivered to component
      : fcsw(4)-ccm: Msg input: msg = RECOVER, size = 0
      : fcsw(4)-ccm: recovery stage
      : fcsw(4)-ccm: no stage 7
      : fcsw(4)-dcm: Next recover (idx) stage: 8
      : fcsw(4)-dcm: Msg input: msg = RECOVER, size = 0
      : fcsw(4)-dcm: recovery stage
      : fcsw(4)-dcm: stage 4
      : fcsw(4)-dcm: message to be delivered to component
      : fcsw(4)-ccm: Msg input: msg = RECOVER, size = 0
      : fcsw(4)-ccm: recovery stage
      : fcsw(4)-ccm: no stage 8
      : fcsw(4)-dcm: Next recover (idx) stage: 9
      : fcsw(4)-dcm: Msg input: msg = RECOVER, size = 0
      : fcsw(4)-dcm: recovery stage
      : fcsw(4)-dcm: stage 5
      : fcsw(4)-dcm: message to be delivered to component
      : fcsw(4)-ccm: Msg input: msg = RECOVER, size = 0
      : fcsw(4)-ccm: recovery stage
      : fcsw(4)-ccm: no stage 9
      : fcsw(4)-dcm: Next recover (idx) stage: 10
      : fcsw(4)-dcm: Msg input: msg = RECOVER, size = 0
      : fcsw(4)-dcm: recovery stage
      : fcsw(4)-dcm: stage 6
      : fcsw(4)-dcm: message to be delivered to component
      : fcsw(4)-ccm: Msg input: msg = RECOVER, size = 0
      : fcsw(4)-ccm: recovery stage
      : fcsw(4)-ccm: no stage 10
      : fcsw(4)-dcm: Next recover (idx) stage: 11
      : fcsw(4)-dcm: Msg input: msg = RECOVER, size = 0
      : fcsw(4)-dcm: recovery stage
      : fcsw(4)-dcm: stage 7
      : fcsw(4)-dcm: message to be delivered to component
      : fcsw(4)-ccm: Msg input: msg = RECOVER, size = 0
      : fcsw(4)-ccm: recovery stage
      : fcsw(4)-ccm: no stage 11
      : fcsw(4)-dcm: Next recover (idx) stage: 12
      : fcsw(4)-dcm: Msg input: msg = RECOVER, size = 0
      : fcsw(4)-dcm: recovery stage
      : fcsw(4)-dcm: stage 8
      : fcsw(4)-dcm: message to be delivered to component
      : fcsw(4)-ccm: Msg input: msg = RECOVER, size = 0
      : fcsw(4)-ccm: recovery stage
      : fcsw(4)-ccm: no stage 12
      : fcsw(4)-dcm: Next recover (idx) stage: 13
      : fcsw(4)-dcm: Msg input: msg = RECOVER, size = 0
      : fcsw(4)-dcm: recovery stage
      : fcsw(4)-dcm: stage 9
      : fcsw(4)-dcm: message to be delivered to component
      : fcsw(4)-ccm: Msg input: msg = RECOVER, size = 0
      : fcsw(4)-ccm: recovery stage
      : fcsw(4)-ccm: no stage 13
      : fcsw(4)-dcm: Next recover (idx) stage: 14
      : fcsw(4)-dcm: Msg input: msg = RECOVER, size = 0
      : fcsw(4)-dcm: recovery stage
      : fcsw(4)-dcm: stage 10
      : fcsw(4)-dcm: message to be delivered to component
      : fcsw(4)-ccm: Msg input: msg = RECOVER, size = 0
      : fcsw(4)-ccm: recovery stage
      : fcsw(4)-ccm: no stage 14
      : fcsw(4)-dcm: Next recover (idx) stage: 15
      : fcsw(4)-dcm: Msg input: msg = RECOVER, size = 0
      : fcsw(4)-dcm: recovery stage
      : fcsw(4)-dcm: stage 1001
      : fcsw(4)-dcm: message to be delivered to component
      : fcsw(4)-ccm: Msg input: msg = RECOVER, size = 0
      : fcsw(4)-ccm: recovery stage
      : fcsw(4)-ccm: no stage 15
      : fcsw(4)-dcm: Next recover (idx) stage: 16
      All service instances on Active
      2020/12/17-12:26:10, [EM-1034], 5,, ERROR, chassis, PS 1 is set to faulty, rc=2000e.
      POST1: Started running Thu Dec 17 12:26:12 GMT 2020
      POST1: Test #1 - Running ceeturboramtest
      POST1: Script PASSED with exit status of 0 Thu Dec 17 12:26:30 GMT 2020 took (0:0:18)
      POST2: Started running Thu Dec 17 12:26:33 GMT 2020
      POST2: Test #1 - Running ceeportloopbacktest (SERDES)
      POST2: Test #2 - Running ceeportloopbacktest (BACKPLANE)
    
      
      Network OS (VDX6730)    
      
      VDX6730 console login: 

> **Copyright Notice**: Original work by this blog’s author, licensed under <a href="https://creativecommons.org/licenses/by-nc-nd/4.0/" target="_blank">CC BY-NC-ND 4.0</a>. When reposting, please include a link to the original source and this copyright notice.