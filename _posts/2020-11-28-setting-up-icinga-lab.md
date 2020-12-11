---
layout: post
section-type: post
title: Distributed Monitoring with Icinga2 - Part 1
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

# Setting up our Icinga2 Lab

## Introduction
I was recently tasked with building out distributed monitoring for our infrastructure that spanned two physical sites and AWS. We decided to go with Icinga2 since that seemed to be the agreed upon best choice. The documentation felt more like a reminder or reference guide for those that have already had experience with Icinga2. Hopefully this guide will help others like me who are brand new to Icinga2 and need help getting started. In this first part we will setup our simulated multi-site environment. I am doing all of this off of my home lab R720 server. I gave each host, except the master, 50gb of storage, 2gb of RAM, and 2 CPUs. I gave the master 8gb of RAM and 4 CPUs since it has to handle a backend database. We will be placing the master in Site-B to simulate it being in AWS. Since the master server does not do any checks itself, we will also need a satellite in Site-B to monitor our host.

## Goal
During this journey I will working towards getting distributed monitoring. Specifically, I will be trying to setup what the documentation calls [Top Down Command Endpoint](https://icinga.com/docs/icinga-2/latest/doc/06-distributed-monitoring/#top-down-command-endpoint). I will have one master server that will be run the web frontend and three satellites, one for each site, that the master will communicate with. I want the satellites to run remote HTTP, WinRM, SSH, and ping checks on the hosts that are in their respective zones and then report back to the master which will display the results on the frontend.

## Terminology
Don't worry if these terms do not make sense right away. I will add context to them as we go through this journey.
* Agent
  * This is a confusing term since it can refer to the Icinga2 service running on an end host or it can refer to the hosts being monitored. For the context of this guide, agent will refer to the Icinga2 service.
* Endpoint
  * This is any server, desktop, device, etc in your infrastructure
* Master
  * The endpoint that has IcingaWeb2 and Director installed on it. Controls and configures the satellites
* Satellite
  * Endpoints that are children of the Master. They are the hosts that do the actual checks on the client endpoints that are in their respective zones. Client endpoints are the devices in your infrastructure that are being monitored by your satellite endpoints and are the children of these satellites.

## Lab Environment
Its important to draw out your environment and plan out how you want to setup your master and satellites. I will be using my home lab to simulate a multi-site setup by using VLANs and firewall rules.
* All VLANs can talk to the internet but they can't talk to each other. The master endpoint is only allowed to talk to the satellite endpoints while the satellite endpoints will only be allowed to talk to the Ubuntu endpoints in their respective VLAN. This will ensure that only the satellites in each zone are doing the monitoring checks.
* VLAN 400
  * Site-A  
  * Satellite
    * Site-A-Satellite
    * 10.40.0.10
    * Ubuntu 18.04 LTS
  * Test hosts
    * Site-A-Host-01
      * 10.40.0.20
      * Ubuntu 18.04 LTS
    * Site-A-Host-02
      * 10.40.0.30
      * Ubuntu 18.04 LTS
* VLAN 500
  * Site-B  
  * Master
    * IcingaMaster
    * 10.50.0.10
    * Ubuntu 18.04 LTS
    * `iptables -A INPUT -s IP-ADDRESS -j DROP`
      * We will need this since the traffic is inner-vlan and thus will not be impacted by the PFSense firewall rules
  * Satellite
    * Site-B-Satellite
    * 10.50.0.20
    * Ubuntu 18.04 LTS
  * Test hosts
    * Site-B-Host-01
      * 10.50.0.30
      * Ubuntu 18.04 LTS
* VLAN 600
  * Site-C
  * Satellite
    * Site-C-Satellite
    * 10.60.0.10
    * Ubuntu 18.04 LTS
  * Test hosts
    * Site-C-Host-01
      * 10.60.0.20
      * Ubuntu 18.04 LTS
    * Site-C-Host-02
      * 10.60.0.30
      * Ubuntu 18.04 LTS

## Diagram
![](/img/2020-11-28-setting-up-icinga-lab-c164a.png)

## Firewall rules
#### Site A
![](/img/2020-11-28-setting-up-naigos-5c27c.png)
#### Site B
![](/img/2020-11-28-setting-up-naigos-6d89b.png)
#### Site C
![](/img/2020-11-28-setting-up-naigos-989b2.png)

<br>
## Resources Used
ESXI 6.7 Hypervisor

## Special Mentions
Roland Sommer and William van Beek on the Icinga community forms as they provided the critical information I needed to succeeded.
