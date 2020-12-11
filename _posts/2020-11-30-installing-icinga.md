---
layout: post
section-type: post
title: Distributed Monitoring with Icinga2 - Part 2
category: Guide
tags: [ 'director', 'icinga2', 'guide', 'distributed', 'monitoring' ]
---
[Part 01 --- Setting up our Icinga2 Lab](/guide/2020/11/28/setting-up-icinga-lab.html)     
[Part 02 --- Installing Icinga2](/guide/2020/11/30/installing-icinga.html)     
[Part 03 --- Installing IcingaWeb2](/guide/2020/12/01/installing-icinga-web.html)      
[Part 04 --- Establishing the Master Satellite Relationship](/guide/2020/12/03/establishing-master-satellite-relationship.html)     
[Part 05 --- Addressing Design Flaws](/guide/2020/12/07/addressing-design-flaws.html)    
[Part 06 --- Installing Icinga2 Director](/guide/2020/12/07/installing-icinga-director.html)  
[Part 07 --- Adding Client Endpoints](guide/2020/12/08/adding-our-client-endpoints.html)    
[Part 23 --- Summary](/guide/2020/12/11/icinga2-journey-summary.html)     

# Installing Icinga2

## Introduction
In part two of our adventure down the Icinga rabbit hole we will now get our Icinga Endpoints installed. For Master this will mean we need to install Icinga and Icinga Web. For the Satellites we will only be installing Icinga. The client endpoints will not need anything further installed on them yet. Later on in this guide we will spin up Apache and a Windows host to test monitoring HTTP and WinRM services.

**Since this guide will be just as much a journey, I will be showing any mistakes I made and how I troubleshooted them.**

## Installation Process
The first thing that we are going to want to do is to take a snapshot of our VM. In case anything goes wrong we are going to want a fallback. Once we get this first Icinga endpoint installed and documented, the rest will be a lot easier. Let's start on IcingaMaster.

Since we are going to be dealing with the configuration files a lot, let's first get `icinga-vim` installed. This will give us syntax highlighting for our Icinga .conf files. I will be using this [handy dandy code](https://gist.github.com/okv/2b0d9b6c1c73c036cabc) from Github user `okv` to get this setup.
```
apt install vim-icinga2 -y
PREFIX=~/.vim && mkdir -p $PREFIX/{syntax,ftdetect} && curl https://raw.githubusercontent.com/Icinga/icinga2/ec75e7dcbbf8c5650197a82107969936220707c8/tools/syntax/vim/syntax/icinga2.vim > $PREFIX/syntax/icinga2.vim && curl https://raw.githubusercontent.com/Icinga/icinga2/ec75e7dcbbf8c5650197a82107969936220707c8/tools/syntax/vim/ftdetect/icinga2.vim > $PREFIX/ftdetect/icinga2.vim
```

According to the official documentation the following code will install Icinga2 onto our endpoint.

```
apt-get update
apt-get -y install apt-transport-https wget gnupg

wget -O - https://packages.icinga.com/icinga.key | apt-key add -

. /etc/os-release; if [ ! -z ${UBUNTU_CODENAME+x} ]; then DIST="${UBUNTU_CODENAME}"; else DIST="$(lsb_release -c| awk '{print $2}')"; fi; \
 echo "deb https://packages.icinga.com/ubuntu icinga-${DIST} main" > \
 /etc/apt/sources.list.d/${DIST}-icinga.list
 echo "deb-src https://packages.icinga.com/ubuntu icinga-${DIST} main" >> \
 /etc/apt/sources.list.d/${DIST}-icinga.list

apt-get update
```

![](/img/2020-11-30-installing-icinga-b95f6.png)
![](/img/2020-11-30-installing-icinga-045c0.png)

`apt -s install icinga2` will do a simulation install of icinga. Checking their [releases page on Github](https://github.com/Icinga/icinga2/releases) confirms that v2.12.2 is indeed the most recent version so let's install it with `apt install icinga2 monitoring-plugins -y`.

![](/img/2020-11-30-installing-icinga-9db39.png)

Looks Good!

To summarize, the complete code to install Icinga is

```
apt-get update -y && apt-get upgrade -y && reboot
apt-get -y install apt-transport-https wget gnupg

wget -O - https://packages.icinga.com/icinga.key | apt-key add -

. /etc/os-release; if [ ! -z ${UBUNTU_CODENAME+x} ]; then DIST="${UBUNTU_CODENAME}"; else DIST="$(lsb_release -c| awk '{print $2}')"; fi; \
 echo "deb https://packages.icinga.com/ubuntu icinga-${DIST} main" > \
 /etc/apt/sources.list.d/${DIST}-icinga.list
 echo "deb-src https://packages.icinga.com/ubuntu icinga-${DIST} main" >> \
 /etc/apt/sources.list.d/${DIST}-icinga.list

apt-get update
apt install icinga2 monitoring-plugins -y
systemctl enable icinga2
systemctl start icinga2
```

Since I setup my hosts to have my SSH keys, I can use my SSH agent to quickly repeat this task on the satellites.

```
[root@picnicbasket ~]# SCRIPT=$(echo 'apt-get -y install apt-transport-https wget gnupg; wget -O - https://packages.icinga.com/icinga.key | apt-key add -; . /etc/os-release; if [ ! -z ${UBUNTU_CODENAME+x} ]; then DIST="${UBUNTU_CODENAME}"; else DIST="$(lsb_release -c| awk '{print $2}')"; fi; echo "deb https://packages.icinga.com/ubuntu icinga-${DIST} main" > /etc/apt/sources.list.d/${DIST}-icinga.list; echo "deb-src https://packages.icinga.com/ubuntu icinga-${DIST} main" >> /etc/apt/sources.list.d/${DIST}-icinga.list; apt-get update -y; apt install icinga2 monitoring-plugins -y; systemctl enable icinga2; systemctl restart icinga2')
[root@picnicbasket ~]# cat icingahosts | grep Satellite
Site-A-Satellite
Site-B-Satellite
Site-C-Satellite
[root@picnicbasket ~]# for host in $(cat icingahosts | grep Satellite); do ssh root@$host "$SCRIPT"; done
```

Confirmed Working!
```
[root@picnicbasket ~]# for host in $(cat icingahosts | grep -iv host); do ssh root@$host "systemctl status icinga2 | grep Active"; done
   Active: active (running) since Thu 2020-12-03 18:25:23 UTC; 54s ago
   Active: active (running) since Thu 2020-12-03 18:25:26 UTC; 53s ago
   Active: active (running) since Thu 2020-12-03 18:25:28 UTC; 52s ago
   Active: active (running) since Thu 2020-12-03 18:25:31 UTC; 50s ago
```

<br>
## Resources Used
https://icinga.com/docs/icinga2/latest/doc/02-installation/  
https://computingforgeeks.com/how-to-install-icinga2-monitoring-tool-on-ubuntu-18-04-lts/  
https://www.howtoforge.com/how-to-install-icinga-2-monitoring-on-ubuntu-1804/  
https://gist.github.com/okv/2b0d9b6c1c73c036cabc  

## Special Mentions
Roland Sommer and William van Beek on the Icinga community forms as they provided the critical information I needed to succeeded.
