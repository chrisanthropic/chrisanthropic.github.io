---
layout: post
title: "Recovering data from a synology NAS disk drive."
sitemap:
  priority: 0.5
  exclude: 'no'
permalink: recovering-data-from-synology-nas-disk-drive
---
I recently had a hard drive in my Synology NAS crash. The SMART scan found something like 8 bad sectors and the volume crashed but the data was still available since the NAS kept it mounted read-only. Contacting Synology support didn't work for me since they said I'd need to install a new drive and create a new volume.

I had the drive created as a single volume in Raid1. I spent quite a bit of time looking online for ways to successfully mount the drive on my linux machine, but all of the `mdadm` based stuff failed me.

That's when I found [this](http://serverfault.com/a/811415) wonderful post and was able to quickly and easily access my drive and copy all of the files. Here's exactly what I did:

1. I removed the hard drive from my Synology.
    - my model supports hot-swapping so I didn't bother powering it off.
2. I turned off my desktop and plugged in the hard drive before turning it back on.
3. Once logged in to my desktop I opened a terminal and:
4. Make sure you can see the drive
    - `lsblk`
        - You should see something similar to this.
    ![sample lsblk output](/images/posts/synology-recovery001.jpg)
        - The largest partition is your data partition, in my case it was sdb3.
5. Use a loop mountpoint to mount our data partition as a device. 
    - `sudo losetup /dev/loop0 /dev/sdb3 -o 1048576`
6. Mount the loop
    - `sudo /dev/loop0 /mnt`
7. Browse your data!
    - `ls -al /mnt`
8. Then I just rsynced the contents to another drive.

It's definitely worth reading the original post for a more detailed explanation of what's going on, but here's the quick and dirty of how I easily got data from an old Synology drive that was part of a Raid1 setup.
