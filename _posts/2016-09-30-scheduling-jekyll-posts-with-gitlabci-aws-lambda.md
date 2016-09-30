---
layout: post
title: "Scheduling Jekyll Posts with GitlabCI and AWS Lambda"
sitemap:
  priority: 0.5
  exclude: 'no'
permalink: scheduling-jekyll-posts-gitlabci-aws-lambda
---
**Table of Contents**

- [Setting Up GitlabCI and Jekyll](#setting-up-gitlabci-and-jekyll)
- [A Note About Pricing](#a-note-about-pricing)
- [Create a Trigger](#create-a-trigger)
- [AWS Lambda](#aws-lambda)
  - [Create a New Lambda Function](#create-a-new-lambda-function)
  - [Configure Triggers](#configure-triggers)
  - [Configure Function](#configure-function)
  - [Lambda IAM Role](#lambda-function-handler-and-role)
  - [Advanced Settings](#advanced-settings)
  - [Create Your Function](#create-your-function)
  - [Test Your Function](#test-your-function)
- [Verify Jekyll Build](#verify-jekyll-build)
- [Wrap up](#wrap-up)

----------------

## Setting Up GitLabCI and Jekyll
_This is the second post of a series about how I'm using GitlabCI to build and deploy my wife's weekly webcomic [shamseecomic.com](https://www.shamseecomic.com)._

**You can read the first post [here](/using-gitlab-ci-to-build-deploy-jekyll-site-amazon-s3/)**

In my previous post I discussed how to use GitlabCI to build and deploy your Jekyll site. We also created a Gitlab Trigger which when _triggered_ will tell Gitlab to build and deploy our site. To finish this tutorial you'll need to have created a GitlabCI trigger at a minimum, so I'll cover that again.

In our example, the webcomic updates every Wednesday at 8am PST. I've already created the posts and named them with the date they should be posted to the site. Then, by setting `future: false` in your `_config.yml` file you tell Jekyll not to show that post until the date in it's title.

We'll set up a simple Lambda task to run every Wednesday at 8am which will curl the Gitlab trigger and build/deploy our updated site!

So, here's what our flow of operations will look like:

- Wednesday, 8AM PST, Lambda scheduled task is performed; curls GitlabCI trigger.
- GitlabCI triggers build tasks
  - jekyll build task
    - post is dated 9-21-2016 and today is 9-21-2016 so this will be a new page added
  - jekyll deploy task

Because of the `future: false` setting, we could accidentally build and deploy the site anytime before 9-21-2016 and Jekyll would never add that page to the live site. So, by simply triggering a jekyll build on the date we specified for a post to go live we can have scheduled Jekyll posts! You could have your site rebuilt hourly, daily, weekly, every 3rd wednesday, etc. Whatever works for your update schedule.


## A Note About Pricing
[AWS Lambda](https://aws.amazon.com/lambda/pricing/) is not a free service. It is however free for the first 1 Million requests per month (and then twenty cents per million after that). 1 scheduled trigger = 1 request. So if you're running scheduled triggers every hour of every day that would be 24x7 or 168 requests. Out of your first free 1 million. In fact, you could run a trigger every 3 seconds for the entire month and still come under your free number of requests. Still, it's not a free service so you've been warned.

## Create a Trigger
If you haven't already created a trigger, go to your project and click on the settings gear and then click on **Triggers**:
![Click on Settings](/images/posts/gitlabci-step006.jpg)

Click on **add trigger** 
_(the token in the screenshot was revoked before this was posted)_
![Click on Settings](/images/posts/gitlabci-step007.jpg)

That's it, now test it out. Replace TOKEN with your actual trigger token and PROJECTID with your Gitlab project's id and then Copy and paste this curl command to your trigger and watch your site build and deploy.

```
curl -X POST \
     -F token=TOKEN \
     -F ref=master \
     https://gitlab.com/api/v3/projects/PROJECTID/trigger/builds
```

## AWS Lambda
_Since this is an AWS service, you will require an AWS account with permissions to access the Lambda service._
I'm not going to go into explaining what all AWS Lambda is, suffice it to say that for our needs it will issue a command on a specified schedule. We're going to tell it to hit our GitlabCI trigger.

Log in to your AWS account and the Lambda dashboard.
![Log in to AWS](/images/posts/lambda-001.jpg)

### **Create a new Lambda Function**.
![Create new Lambda function](/images/posts/lambda-002.jpg)

Select **Skip** blueprint.
Log in to your AWS account and the Lambda dashboard.
![Skip Lambda Blueprint](/images/posts/lambda-003.jpg)

### **CONFIGURE TRIGGERS**
Select **Configure Triggers** and select **Cloudwatch Events - Schedule**
_We're selecting a scheduled trigger since our example requires the site to be updated every Wednesday morning._
Log in to your AWS account and the Lambda dashboard.
![Select 'cloudwatch events - schedule' trigger](/images/posts/lambda-004.jpg)

Enter a **Rule name** and **Rule Description**. Any name/description is up to you.
Log in to your AWS account and the Lambda dashboard.
![Function name and description](/images/posts/lambda-005.jpg)
Enter a **Schedule expression** and tick **Enable Trigger**. We're using the Cron expression for our example:
```
cron(0 15 ? * WED *)
```
![Schedule expression](/images/posts/lambda-006.jpg)
If you're unfamiliar with the Cron format, each 'place' represents a measurement, like so: MINUTE HOUR DAY MONTH WEEKDAY YEAR. And times are in UTC. So our example says;

```
MINUTE HOUR DAY MONTH WEEKDAY YEAR
   0    15   ?    *     WED    *
```
So, schedule says to run at 15:00 UTC every Wednesday regardless of the (number) day of the month, the month, or the year. You could just as easily adjust it to build on a different day of the week, or specify to only build during a specific month or even year.

### **CONFIGURE FUNCTION**
Enter a **Name** and **Description**. Any name/description is up to you.
![Name and describe your function](/images/posts/lambda-007.jpg)

Choose **Node.js** as your **Runtime** and paste the following code:
What we're doing is telling Lambda that when it executes (on our schedule) to execute the following Javascript function, which is simply a curl command to our GitlabCI trigger.

```
var child_process = require('child_process');

exports.handler = function(event, context) {
  var child = child_process.spawn('/bin/bash', [ '-c', 'curl --silent --output - --max-time 1 curl -X POST -F token=TOKEN -F ref=master PROJECT || true' ], { stdio: 'inherit' });
  child.on('close', function(code) {
    if(code !== 0) {
      return context.done(new Error("non zero process exit code"));
    }

    context.done(null);
  });
}
```
![Your function](/images/posts/lambda-008.jpg)
Replace **TOKEN** with the token of your GitlabCI trigger that you created earlier.
Replace **PROJECT** with the project ID number listed on the GitlabCI triggers page.

- Example: `https://gitlab.com/api/v3/projects/1XXXXXX4/trigger/builds`

### **Lambda Function Handler and Role**
Leave the 'Handler' default and choose to "Create a custom role".
![Create Custom Role](/images/posts/lambda-009.jpg)

- **Handler**: Leave default - `index.handler`
- **Role**: Create a Custom Role

This opens up the AWS IAM dash in a new tab with a new custom role already set with our Lambda requirements. Simply push the **Allow** button to create the role.
![Allow Custom Role](/images/posts/lambda-010.jpg)

### **Advanced Settings**
Leave these default. Our Lambda function is just a curl command so we don't need to get into fiddling with the advanced bits for now.
- **Memory**: 128
- **Timeout**: 0min 3sec
- **VPC**: No VPC
![Advanced Settings](/images/posts/lambda-011.jpg)
<hr />

### **Create Your Function**

![Create Your function](/images/posts/lambda-012.jpg)
<hr />


### **Test Your Function**
Now let's test it out. Go back to the Lambda Dashboard, go to **Functions**, and click on the function you just created.

![Select Your function](/images/posts/lambda-013.jpg)
<hr />


Click on the **Test** button.

![Test Your function](/images/posts/lambda-014.jpg)
<hr />


When it's finished you should see an **execution log** similar to this:
![Execution Log](/images/posts/lambda-015.jpg)

If you see the green checkmark and "Execution Result: Success" then congratulations, your Lambda function works. Now let's make sure it triggered your Jekyll build.

## Verify Jekyll Build
- Log back into your Gitlab account and go to your project page (the project that you created the trigger for).
- Go to **Pipelines** and if you've followed along you should see a successful build (or in progress).

![Gitlab Pipeline Log](/images/posts/lambda-016.jpg)

## Wrap Up
If you've followed along at home you now have:

- a Lambda function that is scheduled to execute at 8am PST every Wednesday.
- This function executes a curl command to your GitlabCI trigger
- Your GitLabCI trigger builds and deploys your Jekyll site when triggered
- Your Jekyll site uses the Jekyll `future: false` setting to not publish posts with dates in the future of today's date. This allows you to date your posts for the day you want them published so that triggering a build on that day will published your 'scheduled' post!
