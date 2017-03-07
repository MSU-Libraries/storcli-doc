StorCLI Documentation
===========================

Table of Contents
---------------------------
* [Introduction](#introduction)
* [Terminology](#terminology)
* [Installation](#installation)
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
TODO  


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


