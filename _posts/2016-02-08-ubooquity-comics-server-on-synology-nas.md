---
layout: post
title: "Ubooquity Comics Server on Synology NAS"
keywords:
description:
thumbnail:
facebook_type:
facebook_image:
---

Ubooquity is a cool piece of software that acts as a server for your comic book collection. Think of Plex but for comics.

The installation instructions [provided](http://vaemendis.net/ubooquity/misc/synology-tutorial/) for install it on a Synology NAS were borked (at least for my NAS) so I updated them.

I won't go into as much detail as the 'official' instructions, so this assumes you already know how to SSH into your NAS and use Vi to edit files.

**Prerequisits**

* Install Java on your NAS. [This site](http://minimserver.com/install-synology.html) has a good rundown of which NAS models require which Java versions.
* If you don't already have a 'shared folder' for your Comics, [create one](https://www.synology.com/en-global/knowledgebase/DSM/help/DSM/AdminCenter/file_share_create).

**Install Ubooquity**
_All of these commands will be run from your NAS so you'll need to SSH into your NAS as the **root user** (this is important for proper file/folder permissions)._

**Download Ubooquity**

* `cd /var/packages/`
* `mkdir Ubooquity`
* `wget -O ubooquity.zip http://vaemendis.net/ubooquity/service/download.php`
* `unzip -o ubooquity.zip && rm ubooquity.zip`

**Set Ubooquity to start on boot**

* `cd /etc/init`
* `wget https://gist.githubusercontent.com/chrisanthropic/2e0fc77dea449e2841dc/raw/a0741ec0a4144f2c60e140ca3036d720db726cbb/ubooquity.conf`
* `chmod 755 ubooquity.conf`

**Edit ubooquity.conf**

* `sudo vi /etc/init/ubooquity.conf`
* replace `-workdir "/volume1/Comics"` with the location of your comics shared folder
* replace `httpd-user` with the name of the user with r/w permissions to your comics folder
  * If you're unsure who has what permissions on your shared folder, here's some [instructions](https://www.synology.com/en-global/knowledgebase/DSM/help/DSM/AdminCenter/file_share_privilege) to set it up/check.
  * in my case, I have a single user that SickRage, CouchPotato, Mylar, Transmission, NZBget, and Ubooquity all use.


**Start Ubooquity**
OK, now Ubooquity should be installed and configured so let's start it.

* Run `start ubooquity`

**Login to Ubooquity**
Now that Ubooquity is running we need to visit the WebGUI and finish our setup.

* Open a browser and navigate to http://**NAS-IP**:2202/admin
  * **NAS-IP** needs to be the actual IP of your NAS.

**Add a user**
Ubooquity won't do anything until we define a user and a location of our comics.
The following instructions are done from the WebGUI at: http://**NAS-IP**:2202/admin

* scroll down to "Security"
* click on "edit"
* create new user

**Point it to your Comics**

* scroll down to "comics"
* click on "edit"
* shared directory /volumeX/location-of-your-shared-comic-folder-from-step3-of-Edit-ubooquity.conf-above
  * Authorized Usuers: add the name of the user you created in the previous step

**Test it Out**
Now everything should be good. Let's check.

* Navigate to http://NAS-IP:2202/
* Click on "comics"

You should see your comics starting to populate.
Once that's working make sure to reboot your NAS to ensure that Ubooquity is starting at boot like it should.
