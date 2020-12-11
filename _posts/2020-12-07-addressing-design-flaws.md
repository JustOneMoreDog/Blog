---
layout: post
section-type: post
title: Distributed Monitoring with Icinga2 - Part 5
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

# Addressing Design Flaws

## Introduction
Before we move on to installing Director I wanted to take a moment to address some design flaws in our setup. Our current setup assumes that each Site will only have one satellite. This is almost always not going to be the case. From a security standpoint you are not going to want a host that can access every corner of your network. You wouldn't want a host that is accessible via the Wi-Fi VLAN to also have access to your VLAN with PCI data on it. You also do not want to allow all traffic between your Master and Satellites. Based on what I know now, all you should need for the communication between a Master and a Satellite is ICMP and port 5665 for the API. The problem with adding more Satellites to a site is that the Master will still need a way to communicate with them. This does not eliminate the security concern but rather just moves it up one level to the Master. Now the Master is the golden host with access into a large majority of the network. Granted pivoting off that host would require you to perform some form of exploit through the Icinga API. I also need to address the scenario of adding a Satellite to our setup. I need to know if I can perform the configuration steps I have outlined in previous parts with Director also installed. It wouldn't be a solution if it was not scalable. Lastly, we need to address Icinga Web. It currently does not have SSL and the default Apache page is accessible. Given that my goal is to get distributed monitoring up and working, regardless of prettiness and security, I am not going to address these issues currently.

**Since this guide will be just as much a journey, I will be showing any mistakes I made and how I troubleshooted them.**

## Locking Down the Network

## Setting the Default Page to be /icingaweb2

## Adding SSL to Icinga Web

## Adding a New Satellite post Director Install

## Resources Used


## Special Mentions
Roland Sommer and William van Beek on the Icinga community forms as they provided the critical information I needed to succeeded.
