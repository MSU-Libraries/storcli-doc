StorCLI Documentation
===========================

Table of Contents
---------------------------
* [Introduction](#introduction)
* [Terminology](#terminology)
* [Installation](#installation)
* [Basic Usage](#basic-usage)
* [Updating Firmware](#updating-firmware)
* [Monitoring and Notifications](#monitoring-and-notifications)
* [Silence the Alarm](#silence-the-alarm)
* [Set a Global Hotspare](#set-a-global-hotspare)


Introduction
---------------------------
StorCLI is a utility to monitor and manage LSI/Avago hardware RAID cards.  

For the purposes of this documentation, we will assume you are working on an
Ubuntu 16.04 based server. If you are using a different distribution, your steps
may differ from below. Unless otherwise stated, all commands should be run with
root privileges (e.g. `sudo`).  


Terminology
---------------------------
**RAID**  
RAID stands for Redundant Array of Independent Disks. It is a standards based way of grouping more
than one hard drive together, whether for performance, redundancy, or both. A RAID controller is something
that manages one or more RAIDs. The most common RAID types are:  
 - RAID0 - This is where multiple drives are joined together without any redundancy. If one drive fails, the whole RAID fails. Typically used for performance reasons.
 - RAID1 - This is where two drives are mirrored exactly. If one drives fails, the other has exactly the same information. Smallest possible RAID with redundancy.
 - RAID10 - This is the combination of RAID1 and RAID0, where multiple RAID1 are joined together. Provides increased performance with redundancy.
 - RAID5 - This is where 3 or more drives are joined together. To provide redundancy, parity information is distributed across all drives, allowing any 1 drive to fail. Provides limited redundancy without sacrificing too much space (1 drive worth).
 - RAID6 - This is where 4 or more drives are joined together. Provides redundancy similar to RAID5, only it can survive any 2 drives failing at once. Provides more redundancy than RAID5 while sacrificing a little more space (2 drives worth).

**Controller**  
The RAID card; if you have more than one RAID card installed, you will have more than one controller. Some
servers have a RAID controller integrated right into the motherboard.
Controllers are typically identified by number and start from 0. If you only have one RAID card, it will
likely be referenced as controller 0.  

**Enclosure ID / EID**
The Enclosure ID, or EID, is a grouping of disk in an enclosure. Most small servers only have a single
enclosure. Larger servers, or servers with attached storage, can have multiple enclosures. Each
enclosure will be identifed by a number, but the number does **not** necessarily start a 0. You will
likely have to query the controller to identify the EIDs available.  

**Drive Group / DG**  
A drive group is simply a grouping of drives using RAID. Each RAID you create will be a new Drive Group, typically
starting from 0 and counting up. If you have three RAIDs being managed by your controller, their DG number will
likely be 0, 1 and 2.  

**Virtual Drive / VD**  
A virtual drive is what the controller presents to the operating system as a "drive". While you can split a Drive Group up
into more than one Virtual Drive, you'll typically want a one-to-one mapping of VD to DG. VDs are typically 0 based
like DGs, so if you have 3 Virtual Drives, their number will likely be 0, 1, and 2.  

**Slot Number or Physical Disk / Slt or PD**  
Each enclosure has a number of slots associated with it. Each slot is a phyical disk location where a hard drive can be
placed. If that slot has a hard drive installed, you can use the EID and slot number to make reference to that physical
hard drive.  


Installation
---------------------------
As StorCLI is proprietary, you will need to download the application from the official website. As of the
writing of this document, that would be Broadcom's site. You will need to search for your model of card, and
if StorCLI is compatible with your card, StorCLI will be listed as a download.  
```
https://www.broadcom.com/support/download-search
```

Once downloaded onto the server, unzip the file. If you haven't installed `unzip`, you can do so with `apt-get`.  
```
apt-get install unzip
```

Unzipping the file (`1.21.16_StorCLI.zip` in this example) will create some directories, one of which will
contain another zip file.  
```
unzip 1.21.16_StorCLI.zip 
```

Proceed to unzip the newly extracted zip file.  
```
cd versionChangeSet/univ_viva_cli_rel/
unzip storcli_All_OS.zip
```

This will extract many different types of StorCLI built for various operating systems. Here we'll use
the Ubuntu version. There is also a `read_me.txt` file for each isntance of StorCLI, which you should
look at, in case there are special instructions.  

Now we can install the `.deb` file using `dpkg`.
```
cd storcli_All_OS/Ubuntu/
dpkg -i storcli_1.21.06_all.deb
```

Pay attention for errors, if installation went well, you should have the `storcli64` tool installed
in `/opt/MegaRAID/storcli/`. Test it by using the `show` command.  
```
/opt/MegaRAID/storcli/storcli64 show
```

This should display some basic information about your machine and identify the controller you have
installed. Under the "System Overview" header, you should see your control number (listed under `Ctl`)
and model. Your controller number is very likely to be `0` if you only have a single controller installed.  

You probably don't want to type out `/opt/MegaRAID/storcli/storcli64` every time you want to run StorCLI, so
linking it into `/usr/local/sbin/` will make things easier.  
```
ln -s /opt/MegaRAID/storcli/storcli64 /usr/local/sbin/
```

With the link created, you can just call `storcli64` from the command line from now on. For example, the
following should now work.  
```
storcli64 show
```


Basic Usage
---------------------------
StorCLI accepts a number of switches, such as `/cx`, `/ex`, `/sx`, etc. When these are being referenced
the `x` is a placeholder for a number. For example:  
 - `/c0` means controller 0
 - `/e4` means enclosure 4
 - `/s2` means physical disk slot 2
 - `/v1` means virtual disk 1

When running commands, you may need to combine these switches together. Having spaces between the
switches is allowed. For example:  
 - `/c0/e5/s8` means disk 8 in enclosure 5 on controller 0
 - `/c0 /v1` means virtual disk 1 on controller 0

For example, to show basic information about physical drive 3 in enclosure 4 for controller 0:  
```
storcli64 /c0/e4/s3 show
```

You can also use ranges to represent more than one of something, like `/s0-2` to mean `/s0`, `/s1`,
and `/s2`.  
```
storcli64 /c0/e5/s0-2 show
```

You can also use the `all` keyword to get the entire list of something, like `/sall` means all disk slots.  
```
storcli64 /c0/e5/sall show
```


Updating Firmware
---------------------------
TODO  


Monitoring and Notifications
---------------------------
TODO  


Silence the Alarm
---------------------------
TODO  


Set a Global Hotspare
---------------------------
TODO  


