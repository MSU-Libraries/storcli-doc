StorCLI Documentation
===========================

This documentation aims to provide some basic overview of use of StorCLI. For
more in-depth detail about use of StorCLI, refer to the official StorCLI
Reference Manual available from Broadcom. Many commands and feature that are
only rarely going to be useful are not covered in this document.

Table of Contents
---------------------------
* [Introduction](#introduction)
* [Terminology](#terminology)
* [Installation](#installation)
  - LSI/Avago using StorCLI
  - Dell using PERCCli
* [Basic Usage](#basic-usage)
* [Getting Statuses](#getting-statuses)
* [Changing Properties](#changing-properties)
* [Virtual Drives](#virtual-drives)
* [Convert Virtual Drive to Another RAID Type](#convert-virtual-drive-to-another-raid-type)
* [Updating Firmware](#updating-firmware)
* [Consistency Check Impact](#consistency-check-impact)
* [Email Notification](#email-notification)
* [Silence the Alarm](#silence-the-alarm)
* [Set a Hotspare Drive](#set-a-hotspare-drive)
* [Other Useful Commands](#other-useful-commands)
  * [Spindown a Good Drive Prior to Removal](#spindown-a-good-drive-prior-to-removal)
  * [Clear JBOD Status](#clear-jbod-status)
  * [Clear Foreign Configurations](#clear-foreign-configurations)
  * [Importing Foreign Configurations](#importing-foreign-configurations)
  * [Show Rebuild and Copyback Status](#show-rebuild-and-copyback-status)
* [Implementation Notes](#implementation-notes)


Introduction
---------------------------
StorCLI is a utility to monitor and manage LSI/Avago hardware RAID cards (e.g. MegaRAID cards).  

Dell PERC RAID controllers are mostly based on the same chipsets, thus the below can also work with PERC cards, although they require PERCCLI from Dell (which is just a rebranded version of StorCLI).  

For the purposes of this documentation, we will assume you are working on an
Ubuntu based server. If you are using a different distribution, your steps
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
 - RAID6 - This is where 4 or more drives are joined together. Provides more redundancy than RAID5; it can survive any 2 drives failing at once. Sacrifices a little more space than RAID5, 2 drives instead of just 1.

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
like DGs, so if you have three Virtual Drives, their number will likely be 0, 1, and 2.  

**Slot Number or Physical Disk / Slt or PD**  
Each enclosure has a number of slots associated with it. Each slot is a phyical disk location where a hard drive can be
placed. If that slot has a hard drive installed, you can use the EID and slot number to make reference to that physical
hard drive.  


Installation
---------------------------
**For LSI/Avago/Broadcom branded cards (for Dell, see further down)**  
As StorCLI is proprietary, you will need to download the application from the official website. As of the
writing of this document, that would be Broadcom's site. You will need to search for your model of card, and
if StorCLI is compatible with your card, StorCLI will be listed as a download.  
```
https://www.broadcom.com/support/download-search
```

From the same site, be sure to download the User Guide for your card. This will include detailed information about
both StorCLI usage as well as other management information related to your card.  

Once StorCLI is downloaded onto the server, unzip the file. If you haven't installed `unzip`, you can do so with `apt-get`.  
```
apt-get install unzip
```

Unzipping the file will create a number of directories in your current directory.
```
unzip 007.2203.0000.0000_Unified_StorCLI-PUL.zip
```

This will extract many different types of StorCLI built for various operating systems. Here we'll use
the Ubuntu `.deb` version, but there are other options, like `.rpm`, also included.

Let's install the `.deb` file using `dpkg` or `apt`.
```
cd Ubuntu/
# Using dpkg
dpkg -i storcli_007.2203.0000.0000_all.deb
# Using apt
apt install ./storcli_007.2203.0000.0000_all.deb
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

**For Dell PERC branded cards**  
Dell's hardware based PERC cards are based on the same chipsets, but require you download PERCCLI instead of StorCLI. You
will want to download the latest "PERCLI Utility" for Linux from Dell's support site. Go to the
[LINUX PERCCLI Utility For All PERC Controllers](https://www.dell.com/support/home/en-us/drivers/driversdetails?driverid=f48c2) page
and using the "Enter Details" button you can paste in the serial number and select the RHEL 7 option to get the latest driver
(select the "A gnu zip file for software installation" package to download). 

Download and extract the contents.  
```
tar -xzvf PERCCLI_N65F1_7.1420.00_A10_Linux.tar.gz
```

If the downloaded file contains only a RPM package. For Ubuntu/Debian, use `alien` to create a `.deb` package.  
```
# Install alien if not already installed
apt install alien

# Convert RPM to DEB (be sure to run with root privileges)
alien perccli-007.1020.0000.0000-1.noarch.rpm
```

Copy the pacakge to the appropriate server and install it:  
```
dpkg -i perccli_007.1420.0000.0000_all.deb
```

The default location PERCCLI installs is `/opt/MegaRAID/perccli/`. For consistency sake, we can link this like a `storcli64`
binary:  
```
ln -s /opt/MegaRAID/perccli/perccli64 /usr/local/sbin/storcli64
```

With the link created, calling `storcli64` from the command line will work just like normal StorCLI.  
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
 - `/d1` means drive group 1
 - `/v1` means virtual drive 1

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

You can also use comma delimited lists to represent more than one of something, like `/s2,5` to mean `/s2`
and `/s5`.  
```
storcli64 /c0/e5/s2,5 show
```

You can also use the `all` keyword to get the entire list of something, like `/sall` means all disk slots.  
```
storcli64 /c0/e5/sall show
```

Most StorCLI commands do not ask for confirmation, so be certain of what command you are entering.  


Getting Statuses
---------------------------
Many things in StorCLI support the `show` command, and frequently `show all` to
get a more detailed report. Again `x` in the examples below each represents a different
ID number to indicates the appropriate item, or `all` to query all items of that type.
```
# Controller
storcli64 /cx show
storcli64 /cx show all      # A lot of detailed info

# Enclosures
storcli64 /cx/ex show
storcli64 /cx/ex show all
storcli64 /cx/eall show
storcli64 /cx/eall show all

# Physical Drive Slots
storcli64 /cx/ex/sx show
storcli64 /cx/ex/sx show all
storcli64 /cx/eall/sall show

# Drive Groups
storcli64 /cx/dx show
storcli64 /cx/dx show all
storcli64 /cx/dall show

# Virtual Drives
storcli64 /cx/vx show
storcli64 /cx/vx show all
storcli64 /cx/vall show

# Sound Alarm
storcli64 /cx show alarm

# Controller Properties
storcli64 /cx show PROPERTY
storcli64 /cx show rebuildrate

# VD Properties
storcli64 /cx/vx show PROPERTY
storcli64 /cx/vx show name
```


Changing Properties
---------------------------
A number of example of properties that can be changed.
This list is non-exhaustive; there are many other properties available.
See the official StorCLI Reference Manual for more.

```
# Generic Format
storcli64 TARGET_ID set PROPERTY=VALUE

# VD Name String (15 char max)
storcli64 /cx/vx set name=<namestring>
# VD IO Policy
storcli64 /cx/vx set iopolicy=<cached|direct>
# VD Disk Cache Policy
storcli64 /cx/vx set pdcache=<on|off|default>
# VD Read Cache Policy
storcli64 /cx/vx set rdcache=<ra|nora|adra>
# VD Write Cache Policy
storcli64 /cx/vx set wrcache=<wt|wb|awb>
# VD Enabled/disable SSD caching
storcli64 /cx/vx set ssdcaching=<on|off>
# Sound Alarm
storcli64 /cx set alarm=<on|off|silence>
# RAID Rebuild Rate % (performance impact)
storcli64 /cx set rebuildrate=30
```


Virtual Drives
---------------------------
### Creating a Virtual Drive
Creating a virtual drive can require quite a long command, needing all (or at least most)
of the information about the drive to be passed. In its most basic form, the command is:
```
# Note: PROPERTIES must be in order: type, name, drives, ...
storcli64 /cx add vd [PROPERTIES...]
```

Properties that can be set when creating a virtual drive:

| property | values | default | example | description |
| ------ | ------ | ------ | ------- | ------ |
| type | jbod,raid[0,1,5,6,10,50,60] |   | type=raid5 | Sets the RAID type for the new VD |
| name | {character string} |   | mydrive1 | A label for the VD (15 char max length) |
| drives | comma delimited list |   | 4:2-6,5:0 | A list of of `<enclosure id>:<drive_id(s)>` |
|      | direct,cached | direct |   | Set cached IO policy. Only one value may be set |
|      | wt,wb,awb | wb |    | Sets cached write policy. Write-through, write-back, adaptive write-back. Only one value may be set |
|      | ra,nora,adra | ra |     | Sets read-ahead polity. Read-ahead, no read-ahead, adaptive read-ahead. Only one value may be set |
|      | cachevd |    |     |   If set, enables SSD caching for the VD |
| strip | 8,16,32,64,128,256,512,1024 |    | strip=64 | Sets RAID strip size (in KB) for new VD |

Example VD create commands:
```
# Create a RAID6 w/ 64K strip size using drives:
#  - 0 thru 7 in enclosure 4
#  - 0 thru 3 in enclosure 5
# Other properties left at default values.
storcli64 /c0 add vd type=raid6 name=mybrick1 drives=4:0-7,5:0-3 strip=64

# Create a RAID6 w/ 64K strip size and setting a couple custom properties
storcli64 /c0 add vd type=raid6 name=mybrick2 drives=4:0-11 wt strip=64
```

### Initialization
Whenever you create a new VD, you need to initialize it. StorCLI support both full
initialization and fast initialization. Full initialization is strongly
recommended always. This will scan and clear all the drives. _While full initialization is happening,
however, virtual drives may be visible_ (e.g. through `parted`) _will not be accessible_ (cannot
create partition table).
```
# Start Full Initialization (default is fast init, without the 'full' keyword)
storcli /cx/vx start init full
# Display Initialization Status
storcli /cx/vx show init
# Stop Initialization (normally, would never need to do this)
storcli /cx/vx stop init
# Force Full Initialization (force an init on a drive that has already been initialized)
# WARNING: THIS WILL WIPE ALL DRIVE DATA
storcli /cx/vx start init full force
```

### Deleting a Virtual Drive
```
# Delete the VD (will fail if any data exists for VD)
storcli /cx/vx del
# Force delete the VD, destroying any data on it
storcli /cx/vx del force
```

Convert Virtual Drive to Another RAID Type
---------------------------
StorCLI supports converting (i.e. migrating) RAIDs from one type to another. Of course, during
any major disk job, there is the risk of failure and data loss. Always have backups and understand
you may have to recreate your virtual drives from scratch in case of failure.

Note that not all types of RAID can be converted to another RAID type. Neither RAID5 nor RAID6
can be converted to RAID1. Likewise, no conversions to/from multi-level RAID types are
possible (e.g. RAID10, RAID60).
```
# Convert existing RAID5 to RAID6, adding two disks (1 for parity, 1 to expand data)
# Added disks are: enclosure 5, slot 3; enclosure 5, slot 4
storcli64 /c0/v1 start migrate type=r6 option=add disk=e5:s3,e5:s4

# Show status of migration
storcli64 /c0/v1 show migrate
```


Updating Firmware
---------------------------
To view your current make and model of RAID card, you can run the following. Remember to replace the `x`
in `/cx` with your controller number.
```
storcli64 /cx show | grep "^Product"
```

To view your current firmware package and version, you can run the following. Remember to replace the `x`
in `/cx` with your controller number.  
```
storcli64 /cx show | grep "^FW "
```

To update firmware, download the latest firmware from the official site. This will be the same place where
you downloaded StorCLI from. Unzip the file and read over any `.txt` files provided for additional instructions
or precautions. If you were provided with multiple `.rom` files, read the `.txt` file to ensure you use the
correct one.  

Upgrading firmware to a newer version is _typically_ a "live" action, meaning you can do it while the
controller is running. However, there may be exceptions to this; be sure to read the provided `.txt` file for
details. Downgrading firmware to a prior version is very likely **not** a live action, so be cautious.  

To update the firmware for controller `0` using rom file `mr3108fw.rom`:  
```
storcli64 /c0 download file=mr3108fw.rom
```


Consistency Check Impact
---------------------------
By default, the RAID controller will perform a concurrent (that is, all virtual disks at once) data consistency check
every 168 hours (7 days). This check also defaults to allow a performance impact of 30% while it is running.  

These consistency check (CC) settings can impact performance quite significantly, especially with larger virtual
drives using spindle disks that take long time to run the CC.

This consistency check (CC) can be disabled, but this is _not_ recommended unless any performance impact
is unacceptable.  
```
# NOT RECOMMENDED
storcli64 /cx set cc=off
```

### Viewing CC Info
You can use StorCLI to view setting and status of CC:
```
# Show CC operation mode and schedule
storcli64 /cx show cc
# See what the current CC performance impact setting is
storcli64 /cx show ccrate
# See current state of CC on a virtual drive
storcli64 /cx/vx show cc
```

### Reducing the CC Impact
Rather than disabling CC, you can lower the impact it has by:  
 * Less frequent checks.
 * Checking only one virtual disk (e.g. RAID) at a time instead of all of them sequentually.
 * Lower the rate at which it performs its check.
```
# Set the CC to scan virtual disks sequentially once every 30 days
storcli64 /cx set cc=seq delay=720
# Changing CC performance impact to 10%
storcli64 /cx set ccrate=10
```


Email Notification
---------------------------
Provided along with this documentaion is a script named `notify_raid_problem`. This is a simple
Bash script that will send out an email to `root@localhost` in case of a degraded VD or PD.  

Set the script to run periodically, at least once a day, via cron. In the example below, the script
is placed in `/usr/local/sbin/` and set to run twice a day, at 4:30am and 4:30pm.  
```
# Hardware RAID problem notification
30 4,16 * * *   root    /usr/local/sbin/notify_raid_problem
```

By default, the script will run on controller `0`. If you have a specific controller you wish to check,
then specify the controller number to the script.  
```
# Hardware RAID problem notifications for controller 1 and 2
30 4    * * *   root    /usr/local/sbin/notify_raid_problem 1
30 16   * * *   root    /usr/local/sbin/notify_raid_problem 2
```

Additional flags for `notify_raid_problem` are:  
* `-h` Show help
* `-g` Send alert email if no Global Hot Spare drive is detected
* `-p` Print the results of the check to the terminal, even if no issues were detected
* `-e addr@example.com` Override where the email alerts are sent (default is `root@localhost`)


Silence the Alarm
---------------------------
During any failure or recovery, the controller will trigger an audible alarm that will continue to sound until
all aspects of the hardware are returned to an ideal state. The means that even once you replace a failed
hard drive, the alarm will continue to sound until the RAID that was degraded has fully rebuilt back to its
original state. Obviously, this can be an annoyance.  

To silence an alarm means to stop it from sounding due to all existing known problems. This is distinctly different
from disabling an alarm; disabling prevents the alarm from sounding for any future problems, meaning you have to
remember to re-enable it later. As a general rule, do not disable and alarm, just silence it.  

To silence the alarm for controller `0`, you can run:  
```
storcli64 /c0 set alarm=silence
```

Provided along with this documentaion is a script named `silence_raid_alarm`. This script merely calls
the above command. By default, the script will run on controller `0`. If you have a specific controller you
wish to check, then specify the controller number to the script.  
```
silence_raid_alarm
silence_raid_alarm 2
```


Set a Hotspare Drive
---------------------------
A global hotspare drive will be used to rebuild any DG in the event of a drive failure. Once the failed
drive has been replaced, a "copyback" operation will copy data back to the new drive from the hotspare.
Once complete, the drive in the hotspare slot will be available as a hotspare.  

To create global hotspare using StorCLI, you can use the `add hotsparedrive` command.  
```
storcli64 /cx/ex/sx add hotsparedrive
```

Alternatively, you can create a dedicated hotspare which will only be used for certain DGs. For example,
if you only wanted a hotspare to be used for rebuilding DGs 2 and 3, you could set:  
```
storcli64 /cx/ex/sx add hotsparedrive dgs=2,3
```

Note that after a hot spare drive has been used and then the data copyback has happened on a replaced drive,
the former hot spare may contain a foreign configuration that may need to be cleared. This particular
issue only exists on some versions of RAID controller firmware and can be avoided by upgrading firmware.  

Note that replacement drives may not automatically be accepted for copyback or rebuild. If the drive in
question has ever been used (e.g. a drive recieved from warranty replacement), it may show up as foreign
or `JBOD`. The controller will not automatically make use of such as drive.

If you are certain a new drive is intended to be used for rebuilding or as a hot spare, see the
"Other Useful Commands" section for how to clear the `JBOD` and foreign configurations.  



Other Useful Commands
---------------------------

### Spindown a Good Drive Prior to Removal
To reduce any potential for damaging a good drive by removing it while it's spinning, you can
tell an unconfigured good drive to spin down so you can remove it safely.
```
storcli /cx/ex/sx spindown
```

### Clear JBOD Status
To clear the `JBOD` status of a disk inserted to replace a drive, you can force it to reset to an unconfigured good drive. 
Note, this command should only be used on a drive you know is blank or that you are okay to have overwritten.  
```
# Use an appropriate 'show' command first to determine the controller, enclosure, and drive slot.
storcli64 /cx/ex/sx set good force
```

### Clear Foreign Configurations
If you re-use a drive for a different purpose, the previous RAID configuration will still exist on the drive
itself. The RAID card will detect this and refuse to use the drive in question, to protect the data on the drive
in case a mistake happened, or if your intention is to rebuild a RAID after moving disks from a different server.  

However, if you are actually intending to re-purpose a drive, it will be marked as "F" as the DG, meaning the
drive config is foreign. To be able to use the disk again, you will need to clear that foreign config.  

The `storcli` command can clear ALL foreign drive configs with the following command. This will not affect your drives
that have a numeric DG already. To clear the foreign configs from any drives with the "F" DG:  
```
storcli64 /cx/fall del
```

After which, you should be able to assign the drive(s) as normal unconfigured disks.  

### Importing Foreign Configurations
It is possible to import a foreign configuration, such as if you know you drive groups are intact
but the controller could not load them. Perhaps you are attempting to migrate a set of disks to a new
machine. Regardless of the reason, you can query and attempt to import foreign configurations. This
is not a full walkthrough of the process, but here are commands that should help.
```
storcli64 /cx/fall show
storcli64 /cx/fall show all
storcli64 /cx/fall import preview
storcli64 /cx/fall import
```

### Show Rebuild and Copyback Status
To view the status of current RAID rebuild jobs:  
```
storcli64 /cx/eall/sall show rebuild
```

To view the status of current copyback jobs:
```
storcli64 /cx/eall/sall show copyback
```

Implementation Notes
----------------------------
Accessing RAID physical disks is done by Controller/Enclosure/Slot. If you only have one RAID controller,
the control id will always be 0.  

However, you can have multiple enclosures, such as a 36 drive server which has two enclosures: a 24 bay
one the front and a 12 bay one on the rear. Note that enclosure numbering does not start at zero. It
has commonly been the case where in 36 bay servers, the front bay is enclosure 4 and the rear bay is enclosure 5.  

Within each enclosure, the physical slots always start at 0 for StorCLI purposes. In the above 36 bay
server example, the front would have slots 0 - 23 and the rear would have slots 0 - 11.  

So when referencing `/c0/e5/s9`, it might translate to:
- controller 0
- enclosure 5 (rear in this example)
- slot 9 (which is the 10th bay on the rear)

That is, 24 + 9 = bay 33 when looking at physical bay labels, if the labels on the bays are also numbered
starting with 0. That is, because they don't reset back to 0 when going to the rear (like RAID card
slot numbers do when changing enclosures). _Be aware of your numbering,_ or you might end up pulling a
wrong drive.  
