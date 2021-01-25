# Containerised NTRIP Caster
This ntrip caster is designed to slot into a VPS, Raspberry Pi or any other computer running Docker and provide a highly available service consisting of multiple correction sources and rovers. The following tutorial assumes a brief knowledge of the concepts involved in the network transport of GNSS correction data and the setting up of remote machines. This caster is based off the BKG reference caster with improvements made by Baidu to support ntrip v2.0 and Nearest Base functionality.

---
## Pre-Installation
Firstly, clone your repository.
Navigate to `ntripcaster/conf` and copy the `.dist` files to working files
```
cp ntripcaster.conf.dist ntripcaster.conf
cp sourcetable.dat.dist sourcetable.dat
cp mountpos.conf.dist mountpos.conf
```
### edit ntripcaster.conf
Enter your details in the caster metadata
```
location your_location
rp_email your_email@my.domain.com
server_url http://my.domain.com
```
Enter the password ntrip servers will use to push data into the caster. There is no username.
```
encoder_password yourPassword
```
Enter the domain name of the caster, e.g. localhost for testing, or my.domain.com for real-world usage. This must not be an IP address. Choose the port to use - the standard port is 2101 but you may use any port you please.
```
server_name localhost
#port 80
port 2101
```
The mountpos settings are for the Nearest Base functionality. set `auto_mount` to `false` to disable it.
```
mountposfile /usr/local/ntripcaster/conf/mountpos.conf
auto_mount true
read_gpgga_interval 15
```
Mountpoint usernames and passwords follow this format: only user `usera` with password `abc` or `userb` with password `def` may access `test`. Any user could access `pub`. Adjust to taste.
```
/test:usera:abc,userb:def
/pub
```
### edit mountpos.conf
This file stores the locations of mountpoints so that the caster can determine which stream to send to the client. This feature only needs to be edited if you are using the nearest base function. Enter the mountpoint, lat, long, height in the following format
```
/mountpoint:lat, long, height
/test:51.0704, -2.9162, 13
```
### edit sourcetable.dat
You can follow the syntax guidelines in NtripSourcetable.doc. It is recommended to include the standard caster information in the first line of the file.
```
CAS;rtcm-ntrip.org;2101;NtripInfoCaster;BKG;0;DEU;50.12;8.69;http://www.rtcm-ntrip.org/home
STR;test;BurrowMump;RTCM 2.0;;2;GPS+GLONASS+GALILEO+BEIDOU;QNET;GBR;51.07;-2.91;1;0;Ublox F9P;none;B;N;520;
```
## Installation

Assuming you have a working docker instance, simply navigate to the top of the directory and run
```
docker build .
```
You may include a -t tag to give it a friendly name
```
docker build -t friendlyNtripCaster .
```
If all goes well you will see something like this
```
Successfully built 68b1841290ef
```
then run it, using the -p flag to map the appropriate port. 
```
docker run -p2101:2101 68b1841290ef
```
If you used the -t flag earlier you can call it by its friendly name instead. You can add some further arguments so that you get your terminal back, and the caster runs silently in the background.
```
docker run -p2101:2101 friendlyNtripCaster >/dev/null 2>&1 &
```
Unless you used the silent example above, you should see the caster start up and begin giving status messages. Test by retreiving the sourcetable
```
curl http://my.domain.com:2101/
```
You could also use RTKlib strsvr to grab a stream from a public source such as RTK2go and send it into the caster. Or, if you already have a base station, point that to the caster's address.
You should also be able to pull data using an ntrip client.

Congratulations on your shiny new ntrip caster! 

---

## Credits and changelog
This version is based on the following two repos. Please check out the well written and in-depth readme's on both.
[Goblimey's containerised version](https://github.com/goblimey/ntripcaster)
[Baidu's version](https://github.com/baidu/ntripcaster)

in `ntripcaster/conf`, `Makefile.in` was edited in the following manner:
```
etc_DATA = ntripcaster.conf.dist sourcetable.dat.dist mountpos.conf.dist
```
becomes
```
etc_DATA = ntripcaster.conf.dist sourcetable.dat.dist mountpos.conf.dist mountpos.conf sourcetable.dat ntripcaster.conf
```
`client.c` was adjusted to properly find and read the sourcetable

several config tweaks were made so this should work out of the box.

---

## Original readme

* Standard NtripCaster *


#### New Feature
in this new version, some new functions are supported. It includes
supporting Ntrip V2.0 and changing mountpoint for client automatically.


#### Introduction


The Standard NtripCaster is a software written in C Programming
Language for disseminating GNSS real-time data streams via Internet.
For details of Ntrip (Networked Transport of RTCM via Internet
Protocol) see its documentation available from
http://igs.ifag.de/index_ntrip.htm. You should understand the Ntrip
data dissemination technique when considering an installation of
the software.

The Standard NtripCaster software has been developed within the
framework of the EUREF-IP project,
see http://www.epncb.oma.be/euref_IP. It is derived from the ICECAST
Internet Radio as written for Linux platforms under GNU General
Public License (GPL). Please note that whenever you make software
available that contains or is based on your copy of the Standard
NtripCaster, you must also make your source code available - at
least on request.

The Standard NtripCaster software has been tested so far on various
Suse, Debian, Gentoo, and Redhat (up to Enterprise 5) Linux
distributions. Note that the software may not run today on some
other Linux distributions of recent date. Version 0.1.5 of the
Standard NtripCaster supports a maximum of 50 NtripServers and
100 NtripClients simultaneously,
see http://igs.ifag.de/pdf/NtripImplementation.pdf for technical
details.
 
Ntrip Version 1.0 is an RTCM standard for streaming GNSS data over
the Internet. Offering the Standard NtripCaster Version 0.1.5 is
part of BKG’s policy to help distributing this standard. RTCM may
decide to issue further Ntrip versions as the need arises. Thus,
it might be necessary to modify the Standard NtripCaster
Version 0.1.5 in the future. Ntrip is already part of some GNSS
equipment available today. You may like to ask your vendor about
the Ntrip capability of your GNSS hardware or software.

Although Standard NtripCaster Version 0.1.5 should satisfy most
needs, we continue to work on a high-performance version with
enhanced functionality. This Professional NtripCaster Version is
meant for professional/commercial service providers.

Following your installation, we would appreciate if you could
inform us about the IP address of your Standard NtripCaster. We
intend to keep track of the upcoming global NtripCaster network,
which allows linking them through appropriate entries in the
corresponding configuration files. 

Note that the BKG does not give any warranty regarding the
function of the Standard NtripCaster Version 0.1.5. Moreover,
the BKG disclaims any liability nor responsibility to any person
or entity with respect to any loss or damage caused, or alleged
to be caused, directly or indirectly by the use of Standard
NtripCaster Version 0.1.5.

Please note that due to limited resource we are not able to
give any support concerning installation and maintanance of the
software. 



#### License

NtripCaster, a GNSS real-time data server
Copyright (C) 2004-2008 

This program is free software; you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation; either version 2 of the License, or
(at your option) any later version.

This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU General Public License for more details.

You should have received a copy of the GNU General Public License
along with this program; if not, write to the Free Software
Foundation, Inc., 59 Temple Place - Suite 330, Boston, MA 02111-1307, USA


#### Contact and webpage

The main webpage for Ntripcaster is "http://igs.bkg.bund.de/index_ntrip.htm".

27 February 2008, "euref-ip@bkg.bund.de".

