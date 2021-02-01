---
layout: post
section-type: post
title: Learning the Meraki Dashboard API in Python
category: Guide
tags: [ 'meraki', 'api', 'python' ]
---

[Part 01 --- Setting Up Our Environment](/guide/2021/02/01/lmap-setting-up.html)     
[Part 02 --- Working with the Sites variable](/guide/2021/02/01/lmap-working-with-sites.html)      

# Setting Up Our Environment

## Summary

In this series I will be documenting a majority of the things that I have learned about Meraki's Dashboard API. I started using this API because I wanted to learn Python and because I needed to do audits on my Meraki networks. This series will be written assuming you already have some fundamental knowledge of what Meraki is and how their networks work. While I will not be assuming you know anything about Python, I will assume that you are, at the very minimum, familiar with programming. Lastly, I will also be assuming that we are working in a single Meraki Organization. Everything I will be sharing can be scripted across multiple organizations but since I only have a single Organization to work with, we shall just stick to that. Let's get started with setting our environment.

## Goals

  * Generate our API key
  * Install Python 3
  * Install PyCharm IDE
  * Setup Meraki API Script Starter in PyCharm
  * Generate `sites.pkl`
  * Reading suggestions

## Generating an API Key

The first thing we are going to need to do is generate ourselves an API key. After logging into your Meraki Dashboard, go to your profile by selecting your email in the top right and select `My profile`.

![](/img/2021-02-01-lmap-setting-up-81bf2.png)

Scroll down to the API access section and select Generate new API key.

![](/img/2021-02-01-lmap-setting-up-3a4ea.png)

Copy the generated API key to a secure location such as a password manager. Please understand that your API key shares your permission levels. If this key falls into the wrong hands, depending on our permission level, it can do severe damage to any organization and their networks that you have access to. Please keep it safe.

## Installing Python 3

There are several ways to install Python 3 onto your computer. In this guide I will be using Windows 10 and [Chocolatey](https://chocolatey.org/). Feel free to use whatever you would like to use to get Python 3 onto your computer though. To install Chocolatey we just need to follow the instructions on their [install page](https://chocolatey.org/install). Once it is installed, we can then install Python by simply running `choco install python3`.  

```
PS C:\Windows\system32> choco install python3
Chocolatey v0.10.15
Installing the following packages:
python3
By installing you accept licenses for the packages.
python3 v3.7.4 already installed.
 Use --force to reinstall, specify a version to install, or try upgrade.

Chocolatey installed 0/1 packages.
 See the log for details (C:\ProgramData\chocolatey\logs\chocolatey.log).

Warnings:
 - python3 - python3 v3.7.4 already installed.
 Use --force to reinstall, specify a version to install, or try upgrade.
 ```

Once Python is installed startup a new PowerShell session to ensure your PATH variables are set properly. We can verify that Python is installed by doing the following

```
PS C:\Windows\system32> python3 --version
Python was not found; run without arguments to install from the Microsoft Store, or disable this shortcut from Settings > Manage App Execution Aliases.
PS C:\Windows\system32> python
Python 3.7.4 (tags/v3.7.4:e09359112e, Jul  8 2019, 20:34:20) [MSC v.1916 64 bit (AMD64)] on win32
Type "help", "copyright", "credits" or "license" for more information.
>>> print("Hello World")
Hello World
>>> exit()
```

## Installing PyCharm IDE

I personally use [PyCharm](https://www.jetbrains.com/pycharm/) because I found it to be the most beginner friendly IDE for Python out there. It has both a paid professional and free community version. When you are using the Meraki Python library and make a method call, you can have PyCharm take you to the source code for that method. I find this to be incredibly useful because it will show you exactly what is going on in the background and potentially even show you hidden debugging information. There are of course other features I love about PyCharm but that is the most relevant one I can think of. As always, feel free to use whatever you want. To install PyCharm we will once again use Chocolatey.

```
PS C:\Windows\system32> choco install pycharm-community
Chocolatey v0.10.15
Installing the following packages:
pycharm-community
By installing you accept licenses for the packages.
pycharm-community v2019.2.2 already installed.
 Use --force to reinstall, specify a version to install, or try upgrade.

Chocolatey installed 0/1 packages.
 See the log for details (C:\ProgramData\chocolatey\logs\chocolatey.log).

Warnings:
 - pycharm-community - pycharm-community v2019.2.2 already installed.
 Use --force to reinstall, specify a version to install, or try upgrade.
 ```

## Setting up Meraki API Script Starter in PyCharm

When you open PyCharm for the first time you will be shown the following window.

![](/img/2021-02-01-lmap-setting-up-578ee.png)

Select `Get from VCS`. For the URL we will use `https://github.com/picnicsecurity/Meraki-Dashboard-API-Script-Starter.git` which comes from my [Meraki Dashboard API Script Starter](https://github.com/picnicsecurity/Meraki-Dashboard-API-Script-Starter) repository. This is a script that I wrote specifically for learning Merake Dashboard API and Python. After you put in the above URL hit clone and it will begin setting itself up.

![](/img/2021-02-01-lmap-setting-up-591bd.png)

Once it is done setting up it will automatically open up the Readme file for you.

![](/img/2021-02-01-lmap-setting-up-3ade7.png)

You should read through this file as it contains a lot of important information. Even if you do not fully understand it now, it will serve as your reference guide throughout this series. The most important sections is the Sites Variable Structure since that is what we will be working with.

The next step is get your Python virtual environment and dependencies taken care of. We do this by going to `File -> Settings -> Project: Meraki-Dashboard-API-... -> Python Interpreter`. Then click on the gear icon in the top right and select `Add`.

![](/img/2021-02-01-lmap-setting-up-e5157.png)  

It should default to choosing a new environment with whatever version of Python you installed earlier.

![](/img/2021-02-01-lmap-setting-up-78802.png)

Hit ok and let it go through its process of creating the environment.

![](/img/2021-02-01-lmap-setting-up-ac939.png)

Now that the environment is setup, we will need to install the dependencies. This repository comes with a `requirements.txt` file that will tell pip everything that we will need. In the top left of the PyCharm window you should see a folder icon labeled `Project`. Click on that and it will expand out your project directory structure. Then expand out your `Meraki-Dashboard-API-Script-Starter` folder and you will see your newly created `venv` folder as well as the `requirements.txt` file.

![](/img/2021-02-01-lmap-setting-up-2af01.png)

Open up the requirements file and PyCharm will ask you if you want to install the requirements.

![](/img/2021-02-01-lmap-setting-up-8840d.png)

The `meraki` requirement is the Meraki Dashboard API Python Library that we use to make all of our API calls. The `netaddr` library is what we will use to handle IP Addresses and Networks. Lastly, `tqdm` is used to create progress bars. These progress bars are used when first generating our `sites.pkl` file. Let's generate that file now.

In order to generate our `sites.pkl` file, we will first need to put our API key from earlier into a file. Right click on your project root folder, select New, and then select file.

![](/img/2021-02-01-lmap-setting-up-2c2b2.png)

Name this file `apikey` and put your apikey in it. Save the file and close it. We are now ready to generate our `sites.pkl` file.

## Generating sites.pkl

To generate this file we will need to open up `starter.py`. Then in that window, right click and select `Run file in Python console`.

![](/img/2021-02-01-lmap-setting-up-2c9b3.png)

Now while the script runs let's talk about what exactly is going on here. When I was learning the API, I (expectedly) had to keep making API calls. This meant that I needed to always have an internet connection and that I was always wasting time waiting for the API calls to return the data. This script eliminates all of this. It will take a snapshot of each of our Meraki networks and generate a single variable, called `sites`, containing almost all the data you will need from each Meraki network (each site). The downsides of doing this are that you will not be working with live data. However, that is not the point of this script. This script is for learning and thus it sets you up to work in a testing environment. The `sites` variable is just formatted and organized data from a bunch of API calls. In part two I will go over a bunch of example usages and after each example I will show how to translate the data back to live API calls.

After it has finished running we will see the following

![](/img/2021-02-01-lmap-setting-up-ef5f1.png)

We are now fully setup and ready to move on to [Part 02 --- Working with the Sites variable](/guide/2020/11/30/installing-icinga.html).

## Suggested Reading

In the previous section I made mention that this `sites` variable contains _almost_ all the data you will need. This is because I did not want to include the return of every API call in that variable. I decided to only use the ones that I myself have used the most and are not too specific in their use case. I would **_highly_** suggest that you take a look at the [Meraki API Docs](https://developer.cisco.com/meraki/api-v1/#!overview/interactive-api-docs) as they will list all the the API calls you can make currently. I am making this suggestion because if you read through the documentation, then you will have it in your mind of what you can and cannot do with the API. Some of those features are not going to be in my script for aformentioned reasons. For example, I am not going to include if a switch has warm spare configuration in place as that is highly specific but it could be important to someone. The other reason why you should check out these docs is because it will let you run the API call directly from the site and even tell you the Python code you need in order to make that call on your own. If you know that you will be writing a lot of Meraki scripts, then it is in your best interest to read this all the way through.

[https://developer.cisco.com/meraki/api-v1/#!overview/interactive-api-docs](https://developer.cisco.com/meraki/api-v1/#!overview/interactive-api-docs)

## Resources Used
https://developer.cisco.com/meraki/build/meraki-postman-collection-getting-started/

https://chocolatey.org/install

https://www.jetbrains.com/pycharm/download/#section=windows

https://chocolatey.org/packages?q=pycharm

https://developer.cisco.com/meraki/api-v1/

https://developer.cisco.com/meraki/api-v1/#!overview/interactive-api-docs
