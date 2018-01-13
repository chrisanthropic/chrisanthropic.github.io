---
layout: post
title: "How I migrated my blog from an AWS stack to Netlify."
keywords: "Netlify, Jekyll, static-site, hosting, CDN, AWS"
description: "How to migrate a Jekyll static-site blog from AWS to Netlify. Including DNS, CDN, SSL, "
thumbnail:
facebook_type:
facebook_image:
---
**Table of Contents**

- [Intro](#intro)
- [Create a free Netlify account](#create-free-netlify-account)
- [Import your existing site](#import-your-existing-site)
- [Site Settings](#site-settings)
  - [General](#general)
  - [Build & Deploy](#bild-and-deploy)
- [DNS](#dns)
  - [AWS](#aws--change-your-ttl)
  - [Add Your Custom Domain to Netlify](#add-your-custom-domain-to-netlify)
  - [Set up Netlify DNS](#set-up-netlify-dns)
  - [Update Your AWS DNS](#update-your-aws-dns)
- [Enable HTTPS](#enable-https)
- [AWS Cleanup](#aws-cleanup)
- [Post-Processing](#post--processing)
- [Next Steps](#nest-steps)


----------------
This is the first part of a planned 3 part series.
- Part 1: How I migrated my blog from an AWS stack to Netlify
- Part 2: How I added NetlifyCMS and secured it with Netlify Identity and Git Gateway
- Part 3: How I migrated from formspree.io to Netlify Forms


## Intro
----------------

Up until very recently this site had been hosted on an AWS stack. AWS was my registrar, host, and CDN. I'd manually build the site locally using Jekyll and a Rakefile to handle tasks like minification, lossless image compression, deploying the site to S3, updating the CloudFront distribution, and notifying google & bing about the new sitemap/update. It's solid, cheap setup and has worked great for years. But I manage Jekyll sites for a few other folks and it's not so easy for them to grasp. After spending far too long evaluating different options, I finally decided to migrate my site from an AWS stack to Netlify. 

Netlify has a great free tier that offers things like github integration, automated builds, automated deployment, hosting, one-click ssl, CDN, and DNS. They even offer an open source admin web UI and authentication solutions. 

My goal is to have a fast, secure site that can be managed from a web UI. Netlify offers this for FREE with a decent entry-level paid tier for those managing sites for multiple clients.

## Create free Netlify account
----------------
- [https://www.netlify.com/signup](https://www.netlify.com/signup)
  - You can sign up via GitHub, GitLab, Bitbucket, or an email address.

## Import your existing site
----------------
Since I keep the source code of my blog stored in Github it's simple to import it into Netlify without affecting my live site at all. Importing it allows Netlify to build and deploy the site, but they create a new hostname for me at `RANDOMWORD1-RANDOMWORD2-RANDOMSTRING.netlify.com` meaning it's completely seperate from my live blog but is built from the same source.

- [log in](https://app.netlify.com/) to Netlify
- click on "New site from Git"

Netlify can integrate with GitHub, GitLab, and Bitbucket to build your site anytime a change is pushed. 
- Select your provider from the list.
- Sign in to GitHub/GitLab/Bitbucket in the pop-up window that pops-up
- Select the repo that hosts the source code for your website
- **Branch to deploy**: Select whichever branch is your production branch, mine is `source`
- **Build Command**: Netlify detected that my site was a Jekyll site and autopopulated this with `jekyll build`
- **Publish Directory:** Again, Netlify detected this for me. `_site`
- Select "Deploy Site"
- When it's finished building and deploying your site you'll be redirected to the `overview` page where at the top you'll see the random title given to your site as well as a thumbnail and a link to the live site.

## Site Settings
----------------
I'm not going to cover every setting here, only the important ones I had to change to get my existing site migrated over.

### General
- Change site name (optional): Click the "Change site name" button and change it.
  - You can optionally change the randomly generated site name they gave you. I chose to so I could easily remember it while testing.

### Build and Deploy
- Disable public logs. I don't really see any benefit to them being open by default...
  - Continuous Deployment >> Deploy Settings >> Edit Settings >> Public Deploy Logs >> Disable

## DNS
----------------

### AWS - Change your TTL
Now that I've got Netlify set up and serving my site, it's time to start thinking about how to migrate from AWS. Netlify is already hosting an identical version of the site (replacing S3) behind their free CDN (replacing CloudFront) so we only really need to transfer over the DNS. I used AWS Route53 for my DNS so that's what I'll show in the example.

First we're going to WRITE DOWN our existing TTL settings then we're going to change them to 60 seconds. This is so our DNS update propogates in a timely fashion. We'll revert them once we're finished.

- Log in to AWS and go to the Route53 dashboard, select [Hosted Zones](https://console.aws.amazon.com/route53/home)
- Click on the zone/domain you're switching over to Netlify
- Make note of the TTL settings for each DNS entry
- Change each TTL entry to 60

### Add Your Custom Domain to Netlify
Now that we've got the TTL changed in AWS it's time to head back to Netlify to set up our custom domain.

- Log in to Netlify, Domain Settings >> Add custom domain
- type www.YOURDOMAIN.xxx in the popup 

### Set Up Netlify DNS
Now you should see both of your domains (www.YOURDOMAIN.xxx & YOURDOMAIN.xxx) greyed out with a warning to the right that says **"Check DNS configuration"**.

- Click on that warning
- Select "Set up Netlify DNS"
- Select "Create DNS zone"
- Open your Route53 records for this domain
- Add all DNS records except for `A`, `AAAA`, and `CNAME` from AWS to Netlify
  - You can use the original TTL when adding them to Netlify now that the DNS changes have been made
- Once you've added all of your DNS records into Netlify select "Continue"
- Make note of the four NS hostnames 

### Update Your AWS DNS
Netlify provides a free CDN if you use their DNS so let's use it.

Now that you've set up DNS for your domain on Netlify it's time to tell AWS (also the Registrar for my domain name) to use the Netlify DNS instead. I performed this on multiple live sites and saw no downtime, but this hasn't been tested on high-traffic monetized production sites so there's that.

- Log in to AWS and go to the Route53 dashboard, select [Hosted Zones](https://console.aws.amazon.com/route53/home)
- Click on the zone/domain you're switching over to Netlify
- Replace the existing NS Value with the four NS hostnames from Netlify
  - Make sure to add the trailing `.` to each entry, following AWS formattging.
    - EXAMPLE: `dns1.p01.nsone.net` should be written into AWS as `dns1.p01/nsone.net.` <<- note the trailing `.`
- Update the `SOA` record (if you have one)
  - Change the first 3 sectoins: DNS server, Email, and version number.
    - `ns-whatever-yours-is.org.` should become the first DNS server Netlify listed `dns1.p01.nsone.net.`
    - `whatever.hostmaster.com` should become `hostmaster@nsone.net.`
    - whatevever number follows the hostmaster email (likely a 1) should be incremented by 1
    - EXAMPLE: `ns-1211.awsdns-23.org. awsdns-hostmaster.amazon.com. 1 7200 900 1209600 86400` would become `dns1.p04.nsone.net. hostmaster@nsone.net. 2 7200 900 1209600 86400`
- (IF AWS is your registrar you'll also need to modify the DNS in the following step)
  - Route53 >> registered domains >> YOURDOMAIN >> edit name servers
  - Click on "Add or edit name servers"
  - Replace the existing domains with the four Netlify provided
  - Wait for AWS to email you about the successful DNS change. It typically took less than 5 minutes for me.
- Test your DNS here: [http://dnscheck.pingdom.com/](http://dnscheck.pingdom.com/). All tests should come back Green / "Everything is Fine". 

## Enable HTTPS
Netlify provides free SSL certificate handling via Let'sEncrypt. Let's use it.

**NOTE** 
This step requires a 1 hour waiting period from when the DNS zoe was created on Netlify and when SSL is enabled. So if you're moving quickly like I did you'll get a notification saying:

```
The DNS zone for YOURDOMAIN was created less than 1 hour ago. Please allow at least 1 hour for the DNS changes to propagate.
```

- Log in to Netlify, Domain Settings >> Domain Management >> HTTPS
- Click "Verify DNS configuration"
- Click "Let's Encrypt certificate"
- Click "Provision certificate"
- Wait while it provisions
- Test your SSL (and enjoy the free "A" rating: [https://www.ssllabs.com/ssltest/](https://www.ssllabs.com/ssltest/)

- (Optionally) Force HTTPS. My site is simple and I know that forcing HTTPS won't break it so I always just select "Force HTTPS".

## AWS Cleanup
Now that I'm just using AWS as a Registrar I'll remove the remnants.
- delete CloudFront distribution
- delete all of the DNS records **except** `SOA` and `NS`
- delete S3 bucket that was hosting the site before

### Post-Processing
We're pretty much wrapped up and your site should now be available at your `https://YOURDOMAIN.com` and managed via Netlify. Awesome. Now let's take advantage of another great feature Netlify offers - post-processing!

Previously I relied on a Rakefile with some custom build tasks that would loslessly compress images, minify static assets, etc. Netlify has this built in and can handle it for us as long as we enable it.

- Log in to Netlify, Build & Deploy >> Post Processing
- Asset Optimization >> Edit Settings
- Untick "Disable asset optimization" 
- I leave all the optimizations enabled by default
- Click save

## Next Steps
In my next post I'll cover adding Netlify's free and open source admin dashboard to our site so we can use it to create/modify/destroy posts and pages from a typical WYSIWYG Web interface! This is the main reason I started this journey - an easy interface that I can add to static-sites that I build for other people who may not want to use Git to manage it.
