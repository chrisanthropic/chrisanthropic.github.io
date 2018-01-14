---
layout: post
title: "How I added NetifyCMS to my Jekyll site and secured access to it with Netlify Identity and Git Gateway."
keywords: "Netlify, Jekyll, static-site, front-end, admin panel, dashboard, authorization"
description: "How to add NetlifyCMS to an existing Jekyll site and use Netlify Identity with Git Gatway to secure access to it."
thumbnail:
facebook_type:
facebook_image:
---
**Table of Contents**

- [Intro](#intro)
  - [Requirements](#requirements)
- [Add NetlifyCMS to Your Site](#add-netlifycms-to-your-site)
  - [Create index.html](#create-index-page)
  - [Create config](#create-config)
  - [Config explained](#config-explained)
- [Authentication](#authentication)
  - [Enable Netlify Identity](#enable-netlify-identity)
  - [Enable Git Gateway](#enable-git-gateway)
  - [Add The Netlify Identity Widget](#add-the-netlify-identity-widget)
  - [Add The Netlify Identity Widget Redirect](#add-the-netlify-identity-widget-redirect)
- [Test it out](#test-it-out)
  - [Approve Account Creation](#approve-account-creation)
  - [Login Via Email](#login-via-email)
- [Next Steps](#next-steps)


----------------
This is the first part of a planned 2 part series.
- Part 1: [How I migrated my blog from an AWS stack to Netlify](/blog/2018/migrating-blog-from-aws-stack-to-netlify/)
- Part 2: How I added NetlifyCMS and secured it with Netlify Identity and Git Gateway


## Intro
----------------
Now that I've migrated my Jekyll site from AWS to Netlify (see [previous post](/blog/2018/migrating-blog-from-aws-stack-to-netlify/)) it's time to add [NetlifyCMS](https://www.netlifycms.org/) to my site and secure access to it via Netlify's [Identity](https://www.netlify.com/docs/identity/) Service and [Git Gateway](https://www.netlify.com/docs/git-gateway/). This will add an admin dashboard to our site where we can add/modify/delete posts and pages. The identity and Gateway stuff will make sure that only we can access it.


### Requirements
This guide is written assuming you are hosting a Jekyll based static-site:
- on netlify
- you have a github account
- github is hosting the source of your Jekyll site

## Add NetlifyCMS to Your Site
----------------
The NetlifyCMS crew did a great job making it simple to [integrate](https://www.netlifycms.org/docs/add-to-your-site/) into your site. 

- Create a new directory called `admin` in the root of your Jekyll project

### Create index page
This is the page that will be our admin panel, simply include the couple of `<scripts>` provided by netlify.

- create a new file inside the admin directory called `index.html` containing the following:

```
<!doctype html>
<html>
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Content Manager</title>

  <!-- Include the styles for the Netlify CMS UI, after your own styles -->
  <link rel="stylesheet" href="https://unpkg.com/netlify-cms@^1.0.3/dist/cms.css" />
</head>
<body>
  <!-- Include the script that builds the page and powers Netlify CMS -->
  <script src="https://unpkg.com/netlify-cms@^1.0.3/dist/cms.js"></script>
</body>
</html>
```

### Create config
This controls the settings for our admin panel.

- Create a new file inside the admin directory called `config.yml` containing the following:

```
backend:
  name: git-gateway
  branch: source

publish_mode: editorial_workflow

media_folder: "images"

collections:
  - name: "post"
    label: "Post"
    folder: "_posts"
    create: true
    slug: "{{year}}-{{month}}-{{day}}-{{slug}}"
    fields:
      - {label: "Layout", name: "layout", widget: "hidden", default: "post"}
      - {label: "Title", name: "title", widget: "string"}
      - {label: "Keywords", name: "keywords", widget: "string", required: false}
      - {label: "Description", name: "description", widget: "string", required: false}
      - {label: "Thumbnail", name: "thumbnail", widget: "image", required: false}
      - {label: "Body", name: "body", widget: "markdown", required: false}

  - name: "page"
    label: "Page"
    folder: "_pages"
    create: true
    slug: "{{slug}}.md"
    fields:
      - {label: "Layout", name: "layout", widget: "select", options: ["about", "blog", "contact", "gallery", "page"]}
      - {label: "Title", name: "title", widget: "string"}
      - {label: "Permalink", name: "permalink", widget: "hidden", default: "/{{slug}}/"}
      - {label: "Body", name: "body", widget: "markdown", required: false}

  - name: "layouts"
    label: "Layouts"
    folder: "_layouts"
    extension: "html"
    fields:
      - {label: "Layout", name: "layout", widget: "hidden", default: "default"}
      - {label: "Title", name: "title", widget: "string", required: false}
      - {label: "Body", name: "body", widget: "markdown", required: false}
```

### Config explained
I started writing this section a few times but every time realized that NetlifyCMS has a great explanation of what the config does [here](https://www.netlifycms.org/docs/configuration-options/) and anything I wrote was just a worse version of that. I'll just point you there instead. You may need to slightly modify the fields in the collections to match the YAML front matter of your posts and pages but this should work with most standard Jekyll sites.

## Authentication
----------------
Netlify offers an _Identity_ service that allows you to manage authenticated users with only an email or optional SSO with GitHub, Google, GitLab, and BitBucket.

### Enable Netlify Identity
- Log in to Netlify, Domain Settings >> Identity >> Enable Identity Service
- Leave registration open for now
- Tick the box of any External Providers you want to support
- Leave all the other defaults as-is for now

### Enable Git Gateway
This step requires a GitHub account.

- Scroll to the bottom of the page
- Click on "Enable Git Gateway"
- Log in to your GitHub account when prompted

### Add The Netlify Identity Widget
Now we need to add a small "Identity Widget" script to the admin page and main page of the site. Rather than modify my Jekyll template I'm going to hightlight another cool feature Netlify provides - [Script Injection](https://www.netlify.com/docs/inject-analytics-snippets/). In short we can tell Netlify to inject JavaScript snippets into the `</head>` or before the end of the `</body>` tags on every page of the site.

- Log in to Netlify, Site Settings >> Build & Deploy >> Post Processing >> Snippet Injection
  - Insert before: select `</head>`
  - Script name: Netlify Identity Widget
  - HTML: `<script src="https://identity.netlify.com/v1/netlify-identity-widget.js"></script>`

### Add The Netlify Identity Widget Redirect
Now that we have code to handle the login, let's make sure we get redirected to the amin page after logging in.

- Log in to Netlify, Site Settings >> Build & Deploy >> Post Processing >> Snippet Injection
  - Insert before: select `</head>`
  - Script name: Netlify Identity Redirect
  - HTML: 

```
<script>
  if (window.netlifyIdentity) {
    window.netlifyIdentity.on("init", user => {
      if (!user) {
        window.netlifyIdentity.on("login", () => {
          document.location.href = "/admin/";
        });
      }
    });
  }
</script>
```

Now Netlify will inject that code into every page on your site.

## Test it out
----------------

Push your changes and watch Netlify rebuild your site with a new admin dashboard!

### Approve Account Creation
You should have recieved an email by now from Netlify (at whatever email you use for your Netlify account) that you were added as a user to the site and that you need to click the link to confirm. This is the Netlify Identity service at work! It automatically invites you since you're the Netlify admin.

- Open the Netlify email and click "Accept Invitation"
- Create a password when prompted
- Turn off open registration! (Now that we've successfully registered there's no need to keep it open)
  - Log in to Netlify, Domain Settings >> Identity >> Registration Preferences >> Invite Only

### Login Via Email
I always login via email (username/password) first to test that it's working. Then I test Google/GitHub/etc if it was enabled.

- Navigate to `YOURDOMAIN/admin`
- Login with your email and password
- You should be redirected to `YOURDOMAIN/admin`, if not, navigate there yourself and behold the beauty of a CMS dashboard for your static site!

## Next Steps
----------------
I'd highly recommend reading through the [NetlifyCMS documentation](https://www.netlifycms.org/docs/). It's pretty short but very clear and concise and will get you going on any further customizations you might want to make to your admin page.

