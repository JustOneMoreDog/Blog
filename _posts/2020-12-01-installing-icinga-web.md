---
layout: post
section-type: post
title: Distributed Monitoring with Icinga2 - Part 3
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

# Installing IcingaWeb2

## Introduction
Now that we have Icinga2 installed on our Master and Satellites we setup IcingaWeb2 on Master. Since the Satellites exist so as to do remote checks on our client endpoints, they will not need the web server installed on them.

**Since this guide will be just as much a journey, I will be showing any mistakes I made and how I troubleshooted them.**

## Installation Process
The first thing that we are going to want to do is to take a snapshot of our IcingaMaster VM, as per tradition. In case anything goes wrong we are going to want a fallback.

#### Installing MySQL

IcingaWeb2 uses MySQL database for its backend. You can install the database on a separate host, or if you are using AWS, in an RDS. However, for the scope of this guide, we will be installing the MySQL database on the IcingaMaster VM along side Icinga2.

```
apt install mysql-server mysql-client -y
systemctl enable --now mysql
```

Now we go through the secure installation and configuration with `mysql_secure_installation`

![](/img/2020-12-01-installing-icinga-web-11e92.png)
We used `MySQLRoot123!` for MySQL root password here. Make sure that your passwords are much more secure than this and stored in a safe place. I am noting the password here so that you know which passwords end up getting used where.

Now that MySQL has been given a secure baseline we can move on to setting up Icinga2 to use the MySQL database. To do this we need to install the 'IDO Module' for Icinga2.

```
apt install icinga2-ido-mysql -y
```

![](/img/2020-12-01-installing-icinga-web-58508.png)
![](/img/2020-12-01-installing-icinga-web-d7810.png)
We will be using `Icinga2IDODB!` for the password here. Make sure that your passwords are much more secure than this and stored in a safe place. I am noting the password here so that you know which passwords end up getting used where.
![](/img/2020-12-01-installing-icinga-web-676ac.png)
![](/img/2020-12-01-installing-icinga-web-88d3d.png)

Now we enable the feature and restart Icinga2
```
icinga2 feature enable ido-mysql
systemctl restart icinga2
```

#### Installing IcingaWeb2

Installing the web interface is straight forward. First thing we will do is install Icinga CLI, IcingaWeb2, and PHP GD packages

```
apt install icingaweb2 icingacli php-gd -y
```

Next we set the time zone in `/etc/php/7.2/apache2/php.ini`
```
root@IcingaMaster:/etc/php/7.2/apache2# sed 's/;date.timezone =/date.timezone = America\/New_York/g' php.ini | grep "date.timezone ="
date.timezone = America/New_York

sed -i 's/;date.timezone =/date.timezone = America\/New_York/g' php.ini
systemctl restart apache2
systemctl status apache2
```
You can find a list of PHP's supported Timezones [here](https://www.php.net/manual/en/timezones.php)

Next we will configure a new database for IcingaWeb2 called `icingaweb2` and give the user `icingaweb2` with password `IcingaWeb2DB!` permissions for the database

![](/img/2020-12-01-installing-icinga-web-5323d.png)

```
mysql -u root -p
create database icingaweb2;
grant all privileges on icingaweb2.* to icingaweb2@localhost identified by 'IcingaWeb2DB!';
flush privileges;
quit;
```

Next we generate a setup token
```
icingacli setup token create
The newly generated setup token is: f4f51f23c8d3777b
```

Now it is time to open a browser and configure the WebUI.

```
http://10.50.0.10/icingaweb2/setup
```

![](/img/2020-12-01-installing-icinga-web-a5bff.png)

Paste in the setup token you generated moments ago

![](/img/2020-12-01-installing-icinga-web-83b29.png)

![](/img/2020-12-01-installing-icinga-web-016ce.png)

Make sure everything except the PostgreSQL is green before continuing. If anything is not green then make sure to go back over the previous steps and ensure you did not skip anything

![](/img/2020-12-01-installing-icinga-web-57012.png) 2560 1440p

We will be using the database for authentication here. Later on down the road we can set it up so that we use Active Directory (or LDAP) for authentication. However, since I am not a cool kid with AD in his homelab, database authentication will have to do.

![](/img/2020-12-01-installing-icinga-web-07b4d.png)

We will be using the `icingaweb2`/`IcingaWeb2DB!` credentials here. After validating our configuration we can go on to the next step


![](/img/2020-12-01-installing-icinga-web-aa1ad.png)
![](/img/2020-12-01-installing-icinga-web-d63ea.png)
![](/img/2020-12-01-installing-icinga-web-25cb8.png)
![](/img/2020-12-01-installing-icinga-web-12f59.png)
![](/img/2020-12-01-installing-icinga-web-6a9f7.png)
![](/img/2020-12-01-installing-icinga-web-714b5.png)
![](/img/2020-12-01-installing-icinga-web-b6506.png)
Icinga2IDODB!
![](/img/2020-12-01-installing-icinga-web-ba795.png)
![](/img/2020-12-01-installing-icinga-web-40aa7.png)
![](/img/2020-12-01-installing-icinga-web-e9d81.png)
![](/img/2020-12-01-installing-icinga-web-1c42e.png)
![](/img/2020-12-01-installing-icinga-web-2a7cd.png)
IcingaAdminPassword123!
![](/img/2020-12-01-installing-icinga-web-7aa5e.png)


#### Enabling SSL for IcingaWeb2
_to do_

<br>
## Resources Used
(https://icinga.com/docs/icinga2/latest/doc/02-installation/)[https://icinga.com/docs/icinga2/latest/doc/02-installation/]  
(https://computingforgeeks.com/how-to-install-icinga2-monitoring-tool-on-ubuntu-18-04-lts/)[https://computingforgeeks.com/how-to-install-icinga2-monitoring-tool-on-ubuntu-18-04-lts/]  
(https://www.howtoforge.com/how-to-install-icinga-2-monitoring-on-ubuntu-1804/)[https://www.howtoforge.com/how-to-install-icinga-2-monitoring-on-ubuntu-1804/]  
(https://www.php.net/manual/en/timezones.php/)[https://www.php.net/manual/en/timezones.php/]  

## Special Mentions
