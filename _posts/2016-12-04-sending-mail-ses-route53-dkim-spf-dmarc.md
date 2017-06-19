---
layout: post
title: "Sending mail with AWS SES and Route53 (DKIM, SPF, and DMARC)"
sitemap:
  priority: 0.5
  exclude: 'no'
---
**Table of Contents**

- [Intro](#intro)
- [Notes](#notes)
- [Verify a new Domain](#verify-a-new-domain)
- [DKIM](#dkim)
  - [Generate DKIM DNS Entries](#generate-dkim-dns-entries)
  - [Apply DNS Record Sets](#apply-record-sets)
  - [Verification Emails](#verification-emails)
- [Custom MAIL FROM domain](#custom-mail-from-domain)
- [SPF / DMARC](#spf-dmarc)
- [Wrap up](#wrap-up)

----------------

## **INTRO**
I'm in the process of migrating our newsletter / mailing list from Mailchimp to [EmailOctopus](https://emailoctopus.com/?ali=566327e1-ba66-11e6-90cc-06ead731d453). That'll be a different post, this post is about setting up SES and DNS so that we can send mail from our AWS account that is:

- DKIM signed & verified
- SPF verified
- Valid DMARC

In short, this setup should give you pretty good deliverability results as long as you're not sending spam from a bad domain.

## **NOTES**
- I'm not a DKIM/SPF/DMARC/EMAIL expert. This guide is only to share what works for _me_. There are no guarantees this will work with your set up.
- I use Route53 for my DNS so these instructions will assume the same.
- I use a custom domain, Route53, and Mailgun for [email forwarding](https://www.chrisanthropic.com/blog/2014/mail-forwarding-with-mailgun-and-cloudflare/). Adding this SES stuff doesn't affect it all if done correctly.

## **VERIFY A NEW DOMAIN**
First we'll need to log in to the AWS dashboard and navigate to the SES dash.

![Verify a New Domain](/images/posts/002-verify.jpg)

- Click on the **Verify a New Domain** button.
- Enter your domain name (domain.com)
- Tick **Generate DKIM Settings**
- Click **Verify This Domain**

## **DKIM**

### **Generate DKIM DNS Entries**

- Now you should see the following screen with all of the DKIM DNS entries listed (NOTE, your values will be different from the screenshot).
  ![DKIM DNS Entries](/images/posts/003-verify.jpg)

- Click on **Use Route 53**

### **Apply Record Sets**
- Now you should see the following screen with the DKIM entries again.
  ![DKIM Record Sets](/images/posts/004-verify.jpg)

- Make sure **Domain Verification Record** is TICKED
- Make sure **DKIM Record Set** is TICKED
- Make sure **Email Receiving Record** is NOT TICKED
  - We'll be doing this step next, but we don't want the MX record applied to the naked/root domain, we need to apply it to a new subdomain.
- Make sure **Hosted Zones** is TICKED
- Click **Create Record Sets**
  - **NOTE** This step will modify your Route53 DNS settings, make sure you know what you're doing, don't hold me responsible if something breaks, etc.

### **Verification Emails**

- You should now be redirected to the SES dash and your domain should be listed as **PENDING**.
  ![Verification Status](/images/posts/005-verify.jpg)

- Once Amazon has completed verifying your domain they'll send you an email notifiying you of the success. In my experience this only takes about a minute or two.
  ![Verification Success](/images/posts/006-verify.jpg)

- Once Amazon has completed the domain / DNS settings they'll verify the DKIM settings as well. In my experience this take less than a minute once your site has been verified.
  ![DKIM Verification Success](/images/posts/007-verify.jpg)

## **CUSTOM MAIL FROM DOMAIN**
Now it's time to set up our Custom MAIL FROM Domain. Essentially, this allows SES to mark our emails as "coming from" our domain rather than from Amazon. From the SES dash:

- Click on your domain
- Click on **Set MAIL FROM Domain**
  ![Set MAIL FROM Domain](/images/posts/008-verify.jpg)

- Create a new subdomain to use as your MAIL FROM domain. I'm using `newsletter`.mydomain in this example.
  - I chose to use AWS SES endpoint if another MX record isn't found.
  - Click **Set MAIL FROM Domain**
    ![Create MAIL FROM Domain](/images/posts/009-verify.jpg)

- Click **Publish Records Using Route 53**
  - ![Publish Route53 Records](/images/posts/010-verify.jpg)

- Make sure **MX Record** is TICKED
- Make sure **SPF Record** is TICKED
- Make sure **Hosted Zones** is TICKED
- Click **Create Record Sets**
  - **NOTE** This step will modify your Route53 DNS settings, make sure you know what you're doing, don't hold me responsible if something breaks, etc.
    ![Create Route53 Recordsets](/images/posts/011-verify.jpg)

- Once Amazon is able to verify the DNS settings they will send you an email telling you that it has been successfully verified.
  ![Custom MAIL FROM verification](/images/posts/012-verify.jpg)

## **SPF-DMARC**
Ok, confession time. I've never done a lot of research into DMARC before and this project is no different. I skimmed the official docs as well as AWS docs and have a _working_ solution. If anyone knows of better configs or known issues, I'm all ears.

**UPDATE**
Thanks to user `inopinatus` on [Hacker News](https://news.ycombinator.com/item?id=13101926#13102127) for suggesting [https://dmarc.postmarkapp.com/](https://dmarc.postmarkapp.com/) as a DMARC aggregator to prevent annoyingly noisy DMARC status reports being emailed to you daily from multiple ISPs.

- Go to [https://dmarc.postmarkapp.com/](https://dmarc.postmarkapp.com/) to create your free account
  - Enter any email address to receive your DMARC status reports
  - Enter your new subdomain in the **Send reports about this domain** field. Our example would be **newsletter.chrisanthropic.com**
  ![DMARC](/images/posts/016-verify.jpg)

- Now you should see a screen similar to this:
  ![DMARC](/images/posts/017-verify.jpg)
- Leave this tab/window/screen open and navigate to your AWS dashboard in another tab/window/screen.

- Go to your AWS dash and navigate to the Route 53 dash
- Go to **Hosted Zones** and click on your domain
- Click **Create Record Set**
  ![DMARC](/images/posts/014-verify.jpg)
  - Name: `_dmarc.newsletter`.yourdomain.com
  - Type: `TXT - Text`
  - TTL: 300/default is fine
  - Value: `"COPY AND PASTE THE CONTENTS FROM THE PREVIOUS STEP"`
- Click **Create**

## **WRAP-UP**
- Now you can return to the SES Dash and check the status of your domain
![final verification](/images/posts/013-verify.jpg)
  - **status** Should be _verified_
  - **DKIM Settings Generated** should be _yes_
  - **DKIM Verification Status** should be _verified_
  - **DKIM Signing** should be _enabled_
  - **MAIL FROM Domain** should be _newsletter.yourdomain.com_

- Verify DMARC settings
  - Go to [https://dmarcian.com/dmarc-inspector/](https://dmarcian.com/dmarc-inspector/) and enter your new domain(newsletter.yourdomain.com). You should see something similar to this:
    ![final verification](/images/posts/015-verify.jpg)

That's it for now. I'll go over integrating [EmailOctopus](https://emailoctopus.com/?ali=566327e1-ba66-11e6-90cc-06ead731d453) in the next post.
