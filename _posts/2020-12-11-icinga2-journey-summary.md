---
layout: post
section-type: post
title: Distributed Monitoring with Icinga2 - Part 23
category: Guide
tags: [ 'director', 'icinga2', 'guide', 'distributed', 'monitoring' ]
---
[Part 01 --- Setting up our Icinga2 Lab]()  
[Part 02 --- Installing Icinga2]()  
[Part 03 --- Installing IcingaWeb2]()  
[Part 04 --- Establishing the Master Satellite Relationship]()  
[Part 05 --- Addressing Design Flaws]()  
[Part 06 --- Installing Icinga2 Director]()
[Part 07 --- Adding Client Endpoints]()  
[Part 23 --- Summary]()  

# Summary of the Icinga2 Journey

## Introduction
Towards the end of this journey I decided that it might be helpful to have a global summary page. Some of the aforementioned parts of this journey have their own summary but I do not have a summary for the journey as a whole. This summary will be more of a reference guide focusing less on the details and more on a high level overview. You can find more detail on any given part of this summary in its respective part.

## Table of Contents
- [Summaries](#summaries)
  * [Goal](#goal)
  * [Lab Environment](#lab-environment)
  * [Installing Icinga2](#installing-icinga2)
  * [Installing IcingaWeb2 on IcingaMaster](#installing-icingaweb2-on-icingamaster)
  * [Establishing the Master Satellite Relationship](#establishing-the-master-satellite-relationship)
  * [Locking Down Firewall Rules](#locking-down-firewall-rules)
  * [Setting Default Page to /icingaweb2/](#setting-default-page-to--icingaweb2-)
  * [Adding SSL to IcingaWeb2](#adding-ssl-to-icingaweb2)
  * [Adding New Satellites post Director Install](#adding-new-satellites-post-director-install)
  * [Installing Director](#installing-director)
  * [Adding Client Endpoints](#adding-client-endpoints)

## Summaries

### Goal
During this journey I will working towards getting distributed monitoring. Specifically, I will be trying to setup what the documentation calls [Top Down Command Endpoint](https://icinga.com/docs/icinga-2/latest/doc/06-distributed-monitoring/#top-down-command-endpoint). I will have one master server that will be run the web frontend and three satellites, one for each site, that the master will communicate with. I want the satellites to run remote HTTP, WinRM, SSH, and ping checks on the hosts that are in their respective zones and then report back to the master which will display the results on the frontend.

### Lab Environment
![](/img/2020-11-28-setting-up-icinga-lab-c164a.png)

### Installing Icinga2
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

### Installing IcingaWeb2 on IcingaMaster
```
apt install mysql-server mysql-client -y
systemctl enable --now mysql
mysql_secure_installation
apt install icinga2-ido-mysql -y
icinga2 feature enable ido-mysql
systemctl restart icinga2
apt install icingaweb2 icingacli php-gd -y
sed -i 's/;date.timezone =/date.timezone = America\/New_York/g' php.ini
systemctl enable apache2
systemctl restart apache2
```
```
mysql -u root -p
create database icingaweb2;
grant all privileges on icingaweb2.* to icingaweb2@localhost identified by 'IcingaWeb2DB!';
flush privileges;
quit;
```
```
icingacli setup token create
# Open browser and go through the configuration
```

### Establishing the Master Satellite Relationship
```
root@IcingaMaster:~# icinga2 node wizard
Welcome to the Icinga 2 Setup Wizard!

We will guide you through all required configuration details.

Please specify if this is an agent/satellite setup ('n' installs a master setup) [Y/n]: n

Starting the Master setup routine...

Please specify the common name (CN) [IcingaMaster.picnicsecurity.com]:
Reconfiguring Icinga...
Checking for existing certificates for common name 'IcingaMaster.picnicsecurity.com'...
Certificates not yet generated. Running 'api setup' now.
Generating master configuration for Icinga 2.
Enabling feature api. Make sure to restart Icinga 2 for these changes to take effect.

Master zone name [master]:

Default global zones: global-templates director-global
Do you want to specify additional global zones? [y/N]:
Please specify the API bind host/port (optional):
Bind Host []:
Bind Port []:

Do you want to disable the inclusion of the conf.d directory [Y/n]: Y
Disabling the inclusion of the conf.d directory...
Checking if the api-users.conf file exists...

Done.

Now restart your Icinga 2 daemon to finish the installation!
```
```
icinga2 pki ticket --cn Site-B-Satellite.picnicsecurity.com
044bcbf89a25a322f7a41c0eb5a43f1691e7d8d6
```
```
root@Site-A-Satellite:~# icinga2 node wizard
Welcome to the Icinga 2 Setup Wizard!

We will guide you through all required configuration details.

Please specify if this is an agent/satellite setup ('n' installs a master setup) [Y/n]: Y

Starting the Agent/Satellite setup routine...

Please specify the common name (CN) [Site-A-Satellite]: Site-A-Satellite.picnicsecurity.com

Please specify the parent endpoint(s) (master or satellite) where this node should connect to:
Master/Satellite Common Name (CN from your master/satellite node): IcingaMaster.picnicsecurity.com

Do you want to establish a connection to the parent node from this node? [Y/n]: Y
Please specify the master/satellite connection information:
Master/Satellite endpoint host (IP address or FQDN): 10.50.0.10
Master/Satellite endpoint port [5665]: 5665

Add more master/satellite endpoints? [y/N]: N
Parent certificate information:

 Version:             3
 Subject:             CN = IcingaMaster.picnicsecurity.com
 Issuer:              CN = Icinga CA
 Valid From:          Dec  4 18:19:54 2020 GMT
 Valid Until:         Dec  1 18:19:54 2035 GMT
 Serial:              53:f0:24:3b:fd:e1:f4:be:fa:a4:5f:13:e3:31:22:c3:19:13:9e:99

 Signature Algorithm: sha256WithRSAEncryption
 Subject Alt Names:   IcingaMaster.picnicsecurity.com
 Fingerprint:         1B C8 13 00 BE 5F 4C C3 37 8A A5 0D E6 0D B2 BF 70 7B A9 9F D1 5E AE 15 D7 91 C0 B3 F2 29 13 DA

Is this information correct? [y/N]: Y

Please specify the request ticket generated on your Icinga 2 master (optional).
 (Hint: # icinga2 pki ticket --cn 'Site-A-Satellite.picnicsecurity.com'): 044bcbf89a25a322f7a41c0eb5a43f1691e7d8d6
Please specify the API bind host/port (optional):
Bind Host []:
Bind Port []:

Accept config from parent node? [y/N]: Y
Accept commands from parent node? [y/N]: Y

Reconfiguring Icinga...
Disabling feature notification. Make sure to restart Icinga 2 for these changes to take effect.
Enabling feature api. Make sure to restart Icinga 2 for these changes to take effect.

Local zone name [Site-A-Satellite.picnicsecurity.com]: Site-A
Parent zone name [master]: master

Default global zones: global-templates director-global
Do you want to specify additional global zones? [y/N]: N

Do you want to disable the inclusion of the conf.d directory [Y/n]: Y
Disabling the inclusion of the conf.d directory...

Done.

Now restart your Icinga 2 daemon to finish the installation!
```
```
root@IcingaMaster:~# cat /etc/icinga2/zones.conf
/*
 * Generated by Icinga 2 node setup commands
 * on 2020-12-04 18:20:20 +0000
 */

object Endpoint "IcingaMaster.picnicsecurity.com" {
        host = "10.50.0.10"
        log_duration = 172800
}

object Zone "master" {
        endpoints = [ "IcingaMaster.picnicsecurity.com" ]
}

object Zone "global-templates" {
        global = true
}

object Zone "director-global" {
        global = true
}

object Endpoint "Site-A-Satellite.picnicsecurity.com" {
        host = "10.40.0.10"
}

object Zone "Site-A" {
        endpoints = [ "Site-A-Satellite.picnicsecurity.com" ]
        parent = "master"
}

object Endpoint "Site-B-Satellite.picnicsecurity.com" {
        host = "10.50.0.20"
}

object Zone "Site-B" {
        endpoints = [ "Site-B-Satellite.picnicsecurity.com" ]
        parent = "master"
}

object Endpoint "Site-C-Satellite.picnicsecurity.com" {
        host = "10.60.0.10"
}

object Zone "Site-C" {
        endpoints = [ "Site-C-Satellite.picnicsecurity.com" ]
        parent = "master"
}
```
```
root@Site-A-Satellite:/etc/icinga2# cat zones.conf
/*
 * Generated by Icinga 2 node setup commands
 * on 2020-12-04 18:52:21 +0000
 */

object Endpoint "IcingaMaster.picnicsecurity.com" {
        host = "10.50.0.10"
        port = "5665"
}

object Zone "master" {
        endpoints = [ "IcingaMaster.picnicsecurity.com" ]
}

object Endpoint "Site-A-Satellite.picnicsecurity.com" {
        host = "10.40.0.10"
        log_duration = 0
}

object Zone "Site-A" {
        endpoints = [ "Site-A-Satellite.picnicsecurity.com" ]
        parent = "master"
}

object Zone "global-templates" {
        global = true
}

object Zone "director-global" {
        global = true
}
```
```
root@IcingaMaster:/etc/icinga2# cd zones.d
mkdir Site-A
mkdir Site-B
mkdir Site-C
```
```
root@IcingaMaster:/etc/icinga2# cat hosts.conf
// Site-A Satellite
object Host "Site-A-Satellite.picnicsecurity.com" {
        check_command = "hostalive" // check is executed on IcingaMaster
        address = "10.40.0.10"
        vars.agent_endpoint = name // Following the convention that hostname == endpoint name
}
// Site-B Satellite
object Host "Site-B-Satellite.picnicsecurity.com" {
        check_command = "hostalive" // check is executed on IcingaMaster
        address = "10.50.0.20"
        vars.agent_endpoint = name // Following the convention that hostname == endpoint name
}
// Site-C Satellite
object Host "Site-C-Satellite.picnicsecurity.com" {
        check_command = "hostalive" // check is executed on IcingaMaster
        address = "10.60.0.10"
        vars.agent_endpoint = name // Following the convention that hostname == endpoint name
}
```
```
systemctl restart icinga2
systemctl restart apache2
```

### Locking Down Firewall Rules

### Setting Default Page to /icingaweb2/

### Adding SSL to IcingaWeb2

### Adding New Satellites post Director Install

### Installing Director
```
mysql -u root -p
CREATE DATABASE director CHARACTER SET 'utf8';
CREATE USER director@localhost IDENTIFIED BY 'DirectorDBPass!';
CREATE USER director@localhost IDENTIFIED BY 'DirectorDBPass123!';
GRANT ALL ON director.* TO director@localhost;
flush privileges;
exit;
```

Log into the web frontend and create a new database resource that points to the above database

```
apt-get install php7.2-mbstring

MODULE_NAME=reactbundle
MODULE_VERSION=v0.8.0
MODULES_PATH="/usr/share/icingaweb2/modules"
MODULE_PATH="${MODULES_PATH}/${MODULE_NAME}"
RELEASES="https://github.com/Icinga/icingaweb2-module-${MODULE_NAME}/archive"
mkdir "$MODULE_PATH" \
&& wget -q $RELEASES/${MODULE_VERSION}.tar.gz -O - \
   | tar xfz - -C "$MODULE_PATH" --strip-components 1
icingacli module enable "${MODULE_NAME}"

MODULE_NAME=ipl
MODULE_VERSION=v0.5.0
MODULES_PATH="/usr/share/icingaweb2/modules"
MODULE_PATH="${MODULES_PATH}/${MODULE_NAME}"
RELEASES="https://github.com/Icinga/icingaweb2-module-${MODULE_NAME}/archive"
mkdir "$MODULE_PATH" \
&& wget -q $RELEASES/${MODULE_VERSION}.tar.gz -O - \
   | tar xfz - -C "$MODULE_PATH" --strip-components 1
icingacli module enable "${MODULE_NAME}"

MODULE_NAME=incubator
MODULE_VERSION=v0.6.0
MODULES_PATH="/usr/share/icingaweb2/modules"
MODULE_PATH="${MODULES_PATH}/${MODULE_NAME}"
RELEASES="https://github.com/Icinga/icingaweb2-module-${MODULE_NAME}/archive"
mkdir "$MODULE_PATH" \
&& wget -q $RELEASES/${MODULE_VERSION}.tar.gz -O - \
   | tar xfz - -C "$MODULE_PATH" --strip-components 1
icingacli module enable "${MODULE_NAME}"

ICINGAWEB_MODULEPATH="/usr/share/icingaweb2/modules"
REPO_URL="https://github.com/icinga/icingaweb2-module-director"
TARGET_DIR="${ICINGAWEB_MODULEPATH}/director"
MODULE_VERSION=$(curl https://github.com/Icinga/icingaweb2-module-director/releases/latest --silent | cut -d"=" -f 2 | cut -d">" -f 1 | cut -d'"' -f 2 | rev | cut -d"/" -f 1 | rev)
URL="${REPO_URL}/archive/${MODULE_VERSION}.tar.gz"
install -d -m 0755 "${TARGET_DIR}"
wget -q -O - "$URL" | tar xfz - -C "${TARGET_DIR}" --strip-components 1

systemctl restart icinga2
```

Add the following API user on IcingaMaster and Satellites  
```
vim /etc/icinga2/conf.d/api-users.conf
object ApiUser "director" {
  password = "DirectorAPIPassword123!"
  // client_cn = ""
  permissions = [ "*" ]
}
chown nagios:nagios /etc/icinga2/conf.d/api-users.conf
echo 'include "zones.d/api-users.conf"' >> /etc/icinga2/icinga2.conf
```

```
useradd -r -g icingaweb2 -d /var/lib/icingadirector -s /bin/false icingadirector
install -d -o icingadirector -g icingaweb2 -m 0750 /var/lib/icingadirector
MODULE_PATH=/usr/share/icingaweb2/modules/director
cp "${MODULE_PATH}/contrib/systemd/icinga-director.service" /etc/systemd/system/
systemctl daemon-reload
systemctl enable icinga-director.service

# Go through the kickstart wizard
# Deploy any changes
# Make sure the icinga-director.service daemon is running

systemctl restart icinga2

# Add the director API user to each satellite endpoint in the Web UI
```

### Adding Client Endpoints

* Create a service template for each service you want to monitor
  * Leave `Run on agent` and `Cluster Zone` blank
* Make your host templates
  * Leave `Run on agent`, `Cluster Zone`, and `Command Endpoint` blank
* Add your service templates to your host templates
* Make your client endpoints and assign them the appropriate host template
  * Set the `Cluster Zone` based on whichever zone the host is in
  * Set the `Icinga2 Agent` to no if you are doing remote checks
* Deploy your configuration


## Special Mentions
Roland Sommer and William van Beek on the Icinga community forms as they provided the critical information I needed to succeeded.
