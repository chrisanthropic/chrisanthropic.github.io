---
layout: post
title: How to Build a Kali HVM AMI From Scratch
---
![kali linux](/images/posts/kali_linux.png)
### INTRO

At Blackfin Security, we use Kali Linux extensively for multiple projects, and we also use Amazon EC2. However, as of now, the official Kali AMIs in Amazon from Offensive Security are a few revisions behind their latest release and only support the PVM hardware type. We wanted to run Kali inside Amazon VPCs which require the AMI to be built with HVM support.

Our approach was to use the latest official Debian AMI with HVM support and convert it into a working Kali AMI. We are happy with the result and hope that documenting our steps might be useful to the community.

### GETTING STARTED

You’ll need to have a working [Packer](https://packer.io) install as well as an AWS account to make use of this. If you do, then all you need to do is update a [few variables](https://github.com/chrisanthropic/packer-debian2kali-ec2/blob/master/README.md) with your AWS credentials and run ‘packer build kali.json’. Sit back and wait a bit and then you’ll have a fully updated Kali HVM AMI.

You may use our Unofficial Kali AMI here – ami-c45a71ac (debian2kali-1427320319), but you use it at your own risk.

That said, you’ll most likely want to build your own. We’re starting with an official Debian Wheezy AMI. You can see the whole list [here](https://wiki.debian.org/Cloud/AmazonEC2Image/Wheezy).

### REVERSE ENGINEERING THE KALI BUILD PROCESS

Since Offensive Security already offers an old paravirtual Kali AMI we started by looking at that [code](https://github.com/offensive-security/kali-cloud-build) to see what steps they took. Luckily a lot of it was unnecessary since it involved making the VM AWS compliant – something we didn’t need to worry about since we’re starting with a Debian AMI that is already AWS compliant.

Next, we looked at the [kali-linux-preseed](https://github.com/offensive-security/kali-linux-preseed/blob/master/kali-linux-full-unattended.preseed) repo to see what configurations they set when they build a fresh installation.

Between those two repos and the Kali Custom Image forums we’re pretty confident that we’ve successfully ‘ported’ AMI creation over to Packer.

_The short answer we came up with is pretty simple: Debian + Kali Repos + Kali Kernel = Kali._

We’ll go into a bit more detail below, documenting our steps and you can follow along with the code here if you’re interested: [https://github.com/ctarwater/packer-debian2kali-ec2](https://github.com/chrisanthropic/packer-debian2kali-ec2)

### PACKER / AWS SETTINGS
**Mandatory Settings**

By default Packer requires a few things set up in order to successfully build an AMI in AWS – these can be found in the kali.json file here.

These should all be pretty self-explainable, but we’ll cover them real quick just in case. Your access/secret keys are required for authentication so Packer can access the AWS API. The security group id tells the API what actions your user is allowed to take. The subnet id is required for HVM machines since they’re used with VPCs.

- aws_access_key
- aws_secret_key
- security_group_id
- subnet_id (required for HVM only)
- instance_type
- region
- source_ami

**Build Settings**

Next you have the VM settings or “block_device_mappings”. These specify the “hardware” of your target AMI – these shouldn’t be changed unless you know what you’re doing. Basically we’re telling it to use a 20GB SSD mounted at /dev/xvda and that if the instance is terminated then the volume should be as well.

- “volume_size”: 20,
- “volume_type”: “gp2”,
- “device_name”: “/dev/xvda”,
- “delete_on_termination”: “true”

### KALI BASE

Now we get to the main part – installing Kali with the ‘base.sh’ script.

First, it overwrites the default Debian ‘sources.list’ file with Kali’s repos, then it installs Kali’s gpg key and keyring.

Next it pre-configures some software settings so that we can skip any setup during install and fully automate the process. These configs are taken directly from the kali-preseed repo.

Finally it installs the default Kali metapackages as well as the full Kali linux suite and gnome desktop. The resulting packages are exactly the same as you find on the official Kali iso.

Lastly it runs a dist-upgrade to make sure that we end up with the most recent version of Kali and its packages.

### KALI GRUB

The one change we made where we know we’re not 100% identical to a default Kali install is right here. Kali uses the Grub2 bootloader by default but our source AMI uses Syslinux.

What this script does is install Grub2 alongside Syslinux so everyone is happy.

### CLEANUP

Finally, we remove any ssh keys, clear out all of our logs, and clear our bash history – all as suggested by Amazon if you plan on creating a public AMI.

### CONCLUSION

While the process we followed is straightforward and we believe it is complete, please open an issue in the Github repo if you notice anything I’ve overlooked.
