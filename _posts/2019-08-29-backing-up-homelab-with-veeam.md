---
layout: post
section-type: post
title: Backing Up the Home Lab with Veeam
category: Homelab
tags: [ homelab', 'veeam', 'backups' ]
---
# Homelab Backups with Veeam, Synology, and Backblaze B2

## My Setup

- Dell R720 running ESXi-6.7.0-8169922-standard
- [Synology DS218j](https://www.amazon.com/Synology-bay-DiskStation-DS218j-Diskless/dp/B076G6YKWZ) with two [WD 3TB Reds](https://www.amazon.com/dp/B008JJLW4M/) in Raid 1

## Downloading Stuff

1. First thing we want to do is download the [Veeam Community Edition ISO](https://www.veeam.com/virtual-machine-backup-solution-free.html) and upload it to our desired datastore on ESXI.
2. Next download [Windows Server 2019 Essentials](https://www.microsoft.com/evalcenter/evaluate-windows-server-2019?filetype=ISO)
    - You get 6 months free and can rearm 5 times which gives you plenty of time to plan the migration to the next server version.  Its literally procrastination as a service.

## Setting up the Veeam VM

- I will be [d](https://www.veeam.com/veeam_backup_free_9_5_user_guide_pg.pdf)oing all of this on the fly and hoping for the best. Buckle up buckeroo.
- Based on the [System Requirements](https://helpcenter.veeam.com/archive/backup/95/vsphere/system_requirements.html#backup_server) I will be using the following arbitrary specs
    - Windows Server 2016 x64
    - 75GB HD
    - 16GB RAM
    - 8 CPU Cores
- Attach the Windows Server 2019 ISO to the VM and install the OS
- When you first login you are going to first want to unpin Internet Explorer from your taskbar
- Next thing you are going to want to do is change the workgroup and computer name of your host

    ![Untitled.png](/img/Untitled.png)

    ![Untitled%201.png](/img/Untitled%201.png)

    ![Untitled%202.png](/img/Untitled%202.png)

    - The default name will be something like **WIN-xxxxxxxxxx** which can cause issues down the road.  Changing the default workgroup is not confirmed necessary but I did it just as a sanity check.  After you do this, restart your host
- After your host has finished restarting, swap out the Windows ISO for the Veeam ISO

    ![Untitled%203.png](/img/Untitled%203.png)

    - Hit Install
        - You can do all the defaults from here on out.  It will want you to install SQL which it will do for you so just let it.  I am including screenshots just because I had them.  I took them before I knew that all the default settings would work.

    ![Untitled%204.png](/img/Untitled%204.png)

    - Accept the EULA without reading because we know you wont.  I did, and it took 3 hours.

    ![Untitled%205.png](/img/Untitled%205.png)

    - Hit next

    ![Untitled%206.png](/img/Untitled%206.png)

    - Hit next

    ![Untitled%207.png](/img/Untitled%207.png)

    - Since this is a fresh install of Windows Server we do not have these roles installed.  I am going to trust that Veeam knows what it wants and select the option to have it install everything for me.

    ![Untitled%208.png](/img/Untitled%208.png)

    - Excellent
    - Just keep hitting next and let it install everything it wants
    - This install is going to take a while so while you wait, go sign up for BackBlaze B2.  Get MFA setup and put a credit card on file.  We will be using BackBlaze B2 later in the guide.
    - Once the install finishes, restart the host and then open up the console.

    ![Untitled%209.png](/img/Untitled%209.png)

    ![Untitled%2010.png](/img/Untitled%2010.png)

    - First time launching it will have it asking for updates
        - Ignore the server name here as this screenshot was taken before I knew that having that name, or more specifically the **-**, could be causing issues.

    ![Untitled%2011.png](/img/Untitled%2011.png)

    - Alright we now have the software up and running.  Time to switch to our Synology NAS and configure it

    ## Configuring Synology

    - When you first boot up your nas it will guide you through the initial setup.  All the default options are acceptable here.  Once you have logged in, the first thing you are going to want to do it open control panel.

        ![Untitled%2012.png](/img/Untitled%2012.png)

        - Make sure to select Advanced Mode in the top right too since having all configuration options is a plus
    - Next click on File Services

        ![Untitled%2013.png](/img/Untitled%2013.png)

        - Turn on SMB service and set the workgroup to the same name you set your Veeam host to.  You will also want to scroll down and enable NFS services.
    - Next go to advanced and enable WS-Discovery

        ![Untitled%2014.png](/img/Untitled%2014.png)

    - Now that we have our services configured we need to fix our disk setup.  By default, Synology will use their own **Synology Hybrid Raid (SHR)** which seems to be in place to take away the stress of having to choose which raid you are going to use.  Since we know that we are going to use Raid 1, we need to reconfigure this.
    - In the top left of the screen is a three square one diamond looking icon.  Click on it and then click on **Storage Manager**.

        ![Untitled%2015.png](/img/Untitled%2015.png)

    - Click on Volume

        ![Untitled%2016.png](/img/Untitled%2016.png)

        - You can see on my Volume I already have Raid 1 setup.  You will not see this by default.  What you have to do is remove the volume entirely.  You can not convert from SHR to Raid 1.  After you remove it, click create and it will guide you through the setup.
    - Now that we have our raid configured, we can move on to our shared folders.  On the desktop will be an icon called File Station.  Open it up

        ![Untitled%2017.png](/img/Untitled%2017.png)

        - This is where we will be creating our shared folders.
            - nassyboi shares a folder called nassybackups that exists on my nassypool
    - After you create your shared folder, right click and select properties

        ![Untitled%2018.png](/img/Untitled%2018.png)

        - From here you are going to want to create a dedicated user that will be used to access this share

        ![Untitled%2019.png](/img/Untitled%2019.png)

    - Now we have our shared folder all configured the next thing we are going to want to do is install Perl and Cloud Sync.

        ![Untitled%2020.png](/img/Untitled%2020.png)

        ![Untitled%2021.png](/img/Untitled%2021.png)

        - Perl is a just in case things go wrong and you need to do plan B with Veeam (connect to the NAS as a Linux Server and use SCP and NFS ( this is rather complicated and not worth your time though so if you are finding yourself having issues with just using SMB then reinstall and start over)).
        - Cloud Sync is what we will be using to connect to BackBlaze B2
    - After both of those have finished installing, restart your Synology and move back to your Veeam VM

    ## Verifying Shit Works

    - Since Veeam will be connecting to our nassyboi via SMB, we are going to want to test that our Windows VM can connect to it first before letting Veeam try.
        - If you haven't already, make sure to update DNS and DHCP accordingly.  We do not want to be using IPs here, only hostnames.
    - Open up file explorer on your Windows host and type in your SMB path

        ![Untitled%2022.png](/img/Untitled%2022.png)

        - For me this was **\\nassyboi\nassybackups**
            - You may see me switch this later on to **\\nassboi\veeam**.  While troubleshooting I ended up switching the folder name.  These are the exact same though so don't get confused.
    - It will ask you for a username and password.  Give it the one you setup earlier and if all is well, you should see your share.

        ![Untitled%2023.png](/img/Untitled%2023.png)

    - Inside this share we are going to make a file called **tester**.  Go over to your NAS and confirm it was made.  Now open tester and write, **blessed be sampai for his ~~ego~~ wisdom knows no bounds**, and save it.  Confirm the changes can be seen on your NAS.  Now delete the file.  Again, confirm your changes can be seen on your NAS.  This may seem tedious and over the top but it is important to double check you have full Read Write permissions.

    ## Adding NAS as a Backup Repository

    - Open up the console and launch Veeam.  Once inside on the bottom left you will see something called Backup Infrastructure.  Click on it.

        ![Untitled%2024.png](/img/Untitled%2024.png)

        - Side Note, you may not have Home, Inventory, etc listed right away but as we setup things, they will start to show up naturally.  You can also find them if you click on the right arrow on the bottom left of the window.
    - Now click on Backup Repositories and underneath the Default Backup Repository right click and select Add Repository

        ![Untitled%2025.png](/img/Untitled%2025.png)

    - Select Network Attach Storage and then Shared Folder

        ![Untitled%2026.png](/img/Untitled%2026.png)

    - Now give it a name and then move on to share

        ![Untitled%2027.png](/img/Untitled%2027.png)

        - Here we use the same UNC path that we used before.  Also we want to pass in the credentials that are needed to access the share.  I am not on any domain so I did not need to do anything special with the username like **nassyboi\sandwich** or **homelabdomain\sandwich**.
    - I knew ahead of time that the total size of my VMs is about 400GB.  Since I have 2.7TB to work with, I decided to only keep 4 copies.  This will allow me the room to grow.

        ![Untitled%2028.png](/img/Untitled%2028.png)

    - Now hit next through the rest of the menus leaving everything as default.  Once finished you should now see your SMB share listed.

    ![Untitled%2029.png](/img/Untitled%2029.png)

    ## Adding ESXI to Veeam

    - On the bottom left of the window click on Inventory and then click on Add Server.  This part is rather straight forward so I will show it just with screenshots

        ![Untitled%2030.png](/img/Untitled%2030.png)

        ![Untitled%2031.png](/img/Untitled%2031.png)

        ![Untitled%2032.png](/img/Untitled%2032.png)

        ![Untitled%2033.png](/img/Untitled%2033.png)

        ![Untitled%2034.png](/img/Untitled%2034.png)

        ![Untitled%2035.png](/img/Untitled%2035.png)

        ![Untitled%2036.png](/img/Untitled%2036.png)

    - Now we just hit finish and our hypervisor will be added and then it will automatically bring up the next window and our next task

    ## Adding a Backup Job

    - First thing we do is of course give it a name and description

        ![Untitled%2037.png](/img/Untitled%2037.png)

        ![Untitled%2038.png](/img/Untitled%2038.png)

    - Now I selected a bunch of VMs here but it is a much better option to instead just select one of your smaller VMs like pfsense and only back that up.  After you confirm everything is working correctly, you can go back and select the rest of the VMs you want to backup.  I ended up doing this as you will see in the next screenshot
    - Next you select your repository that you just made and hit next

        ![Untitled%2039.png](/img/Untitled%2039.png)

        ![Untitled%2040.png](/img/Untitled%2040.png)

    - Now set a schedule for your backup.

        ![Untitled%2041.png](/img/Untitled%2041.png)

        ![Untitled%2042.png](/img/Untitled%2042.png)

    - Excellent!  Now run your backup and ensure that it works

        ![Untitled%2043.png](/img/Untitled%2043.png)

    ## Syncing Up With BackBlaze B2

    - Going back to our Synology, lets open up that Cloud Sync that we installed earlier

        ![Untitled%2044.png](/img/Untitled%2044.png)

    - Select BackBlaze B2
    - Now you will need to generate some keys for Cloud Sync to use.  Go to your BackBlaze B2 Account and select App Keys

        ![Untitled%2045.png](/img/Untitled%2045.png)

    - Now create a new Application Key

        ![Untitled%2046.png](/img/Untitled%2046.png)

    - Now they keys will appear one time for you.  Save them in a secure location and then copy them into the fields accordingly.  You will know you got it right if you can select your bucket

        ![Untitled%2047.png](/img/Untitled%2047.png)

    - Now you set your Sync options.  Give it the local path you want to sync.  This will be the folder containing your backups.  I also chose to encrypt my backups since they are leaving my network.

        ![Untitled%2048.png](/img/Untitled%2048.png)

        - I am fairly confident that the 1-23 are hours.  I am setting mine to run at 7am every day.
    - Hit next and then apply and it will finish the setup for you.  It will download the key for you to store locally.  Store that securely.  You will be left with the following screen confirming what I expected earlier about the scheduling time.  I did not see a way to force a sync right away so I am going to wait until to morrow and verify everything then

        ![Untitled%2049.png](/img/Untitled%2049.png)

Boom.  Backups Complete
