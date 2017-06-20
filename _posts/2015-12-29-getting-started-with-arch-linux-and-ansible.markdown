---
layout: post
title: "Getting started with Arch Linux and Ansible"
date: 2015-12-29T10:55:35-08:00
sitemap:
  lastmod: 2015-12-29T10:55:35-08:00
  priority: 0.5
  changefreq: monthly
  exclude: 'no'
---

**UPDATE**
I created a tiny repo that expands on the basics of this post. You can check it out here: [https://github.com/chrisanthropic/ansible-aws-template](https://github.com/chrisanthropic/ansible-aws-template).
**/UPDATE**

I've been asked to play around with Ansible a bit for work. My first impressions have ranged from "it looks good so far" to "what the fuck does that mean?" I much prefer Chef's documentation.

Here's a real quick-and-dirty "getting started" post mostly for my own memory. Chances are good that this isn't what you want, whoever you are, but if you're looking at using Ansible on an Arch Linux machine to manage a bunch of AWS resources, cool, let's do it.

First I installed Ansible, Python2, python2-boto, and python-boto (not sure why python-boto is needed as well as python2-boto, but the script complained about it missing even with the later environmental variable set, whatever, I'll figure it out later)
    `# pacman -S ansible python2 python2-boto python-boto`

Next, since Ansible uses Python2 and Arch ships with Python3, we need to tell Ansible where to find Python2. The Ansible documentation (and Arch wiki) mention this but don't really make it clear enough for me since there's roughly 6 billion fucking optional files to set this. I plan on sharing my Ansible stuff with others I don't want to set it there since setting it in the Ansible configs could *potentially* result in it breaking for non-Arch users, so I set it locally on my Arch machine.
    `echo ansible_python_interpreter=/usr/bin/python2 >> ~/.ansible.cfg`
    
That takes care of the bare minimum to get Ansible working on Arch. 

Next I needed to set up what Ansible calls my 'Inventory' - a list of machines that I want Ansible to talk to. Creating a manual file would be a pain for AWS/EC2 stuff especially if using Auto Scaling Groups. Luckily Ansible provides a script that creates what they call a 'Dynamic Inventory'. So here's how I got that working.

First, create a directory for your new Ansible project
    `mkdir ~/Ansible-Project`
Next, download the script from Ansible
    `wget https://raw.github.com/ansible/ansible/devel/contrib/inventory/ec2.py -P ~/Ansible-Project`
Now make that script executable
    `chmod +x ~/Ansible-Project/ec2.py`
    
After all of this I still ran into issues when trying to run the script and a bit of RTFMing revealed that I also needed to copy the default ec2.ini file into the same directory as the ec2.py script.
    `wget https://raw.githubusercontent.com/ansible/ansible/devel/contrib/inventory/ec2.ini ~/Ansible-Project`
    
Now, before testing it I'll need to supply my AWS credentials. Since I'm just testing for now I'll pass them as environmental variables.
    `export AWS_ACCESS_KEY_ID='your-key-here'`
    `export AWS_SECRET_ACCESS_KEY='your-key-here'`
    
Now, test the script and it should spit out a list of ec2 instances running in your AWS account.
    `cd ~/Ansible-Project`
    `./ec2.py --list`

Boom. Done.
    
