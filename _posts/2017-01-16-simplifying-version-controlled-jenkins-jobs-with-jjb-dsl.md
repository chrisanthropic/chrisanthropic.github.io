---
layout: post
title: "Simplifying version controlled Jenkins jobs with Jenkins Job Builder and Jobs DSL"
sitemap:
  priority: 0.5
  exclude: 'no'
permalink: simplifying-version-controlled-jenkins-jobs-with-jjb-dsl
---
**Table of Contents**

- [Intro](#intro)
- [Notes](#notes)
- [Install](#install)
- [Configure](#configure)
- [Create the Seed job](#create-the-seed-job)
- [Create the Test job](#create-the-test-job)
- [Creating New Jobs](#creating-new-jobs)
- [Migrating Jobs](#migrating-jobs)
- [Wrap up](#wrap-up)
- [Repos](#repos)

----------------

## **INTRO**
I've been doing a lot of Jenkins research for work and one of the things I always miss the most when using Jenkins is the ease of job creation that similar tools like Travis, GitLab, etc. use. These tools let you define your 'job' in a simple Yaml file at the root of the project and when pushed the build tool will do as instructed.

Compare that to Jenkins' default of requiring you to use the GUI for most setup. Now, things have been changing (for the better in my opinion) for Jenkins lately and the recommend using the Jobs DSL to programatically define jobs, but why did they choose Groovy DSL over YAML?

Jenkins Job Builder and Jobs DSL are both cool, but neither can automatically create a job when a repo is pushed. Let's change that.

**In short, we'll configure Jenkins so that you can use YAML to define your Jenkins job, save it in the root of your project, and when deployed it will _automatically_ create and run the job for you.**

## **NOTES**
My research led me to: [Jenkins à la Travis](https://medium.com/mindera/jenkins-a-la-travis-6c5a8debbb5b). This post and the repos explain the general gist of his setup but the documentation was a little light, so consider this an extension of that original post.

HUGE thanks to the original author [João Cravo](https://twitter.com/joaogbcravo) for his hard work and inspiration!

## **INSTALL**
I'm using Ubuntu 16.04 for this example.
Install Jenkins Job Builder (JJB) and dependencies.

- NOTE: I run the following commands as my `jenkins` user to keep permissions issues to a minimum.

```
    apt-get install -y python-pip
    su jenkins
    pip install jenkins-job-builder
    pip install PyYAML python-jenkins
```
- Copy the binary to /usr/bin
  - `cp -R .local/bin/jenkins-jobs /usr/bin/jenkins-jobs`

## **CONFIGURE**
(Can be run as root or default user, doesn't have to be run as Jenkins user)

- Create the default config
  - `vi /home/jenkins/.jenkins/jenkins_jobs.ini`

    ```
        [job_builder]/jobs.yml
        ignore_cache=true
        keep_descriptons=false
        recursive=true
        allow_duplicates=false

        [jenkins]
        user=$USERNAME
        password=$YOUR_JENKINS_API_TOKEN
        url=$http://YOUR-JENKINS-URL:P/jobs.ymlORT
        query_plugins_info=False
    ```
- Install virtualenv
    - `pip install virtualenv`
- Ensure permissions
    - `chown -R jenkins:jenkins /etc/jenkins_jobs`
- Create symlink to the binary
    - `sudo ln -s /usr/bin/jenkins-jobs /usr/local/bin/jenkins-jobs`

## **CREATE THE SEED JOB**
We're going to be using [João Cravo's](https://twitter.com/joaogbcravo) meta_repositories_jobscreator.py Python script to create our 'meta' Seed job. He should get full credit for this magic.

- Git clone the JJB-Seed repo
    - `git clone https://github.com/chrisanthropic/jenkins-job-creator-seed.git ~/jjb-seed/`
- Ensure permissions
    - `chown -R jenkins:jenkins /home/jenkins/jjb`

Now we'll run the setup job for the first time which will create a job to monitor repos for .jenkins/jobs.yml files and create the Jenkins jobs defined by them. It's a bit confusing but will make more sense later.

- ```
jenkins-jobs -l DEBUG --ignore-cache --conf .jenkins/jenkins_jobs.ini update -r globals:.jenkins/jobs.yml
  ```

## **CREATE THE TEST JOB**
Let's pretend you just created a new Hello World job for testing purposes and you want to see how this setup works. I've created the repo for you to use, but you could just as easily test your own.

Once you have a repo containing a .jenkins/jobs.yml file;

- Add the Git URL of the target repo to `~/jjb-seed/repository_list.yml`.
- Manually build the Seed job, or wait until the scheduled build triggers (every minute).
    - Once the Seed job sees the new repo added to the `repository_list` it will do the following FOR EVERY REPO IN THE LIST:
        - Check the repo for a .jenkins/jobs.yml file
        - create a JJB job that will continually check the .jenkins/jobs.yml file for changes and create the job defnined.
        - create a JVB job that will continually check the .jenkins/views.yml file for changes and create the views defined.
    - Yes, this means that for every job you create this way you'll end up with:
        - jjb job that creates the job defined in the repo
        - jvb job that creates the views defined in the repo
        - the actual job you defined
        - the views you defined

## **CREATING NEW JOBS**
Creating new jobs is simple.

- Create a new project
- Add a .jenkins directory in the root of the project
- Add a jobs.yml file inside the .jenkins directory.
    - define your job using the [Jenkins Job Builder](http://jenkins-job-builder.readthedocs.io/en/latest/index.html) yaml syntax.
- Push your project to Github
- Add your project's URL to ~/jjb-seed/repository_list.yml
- Wait for JJB seed job to build and create:
    - YOUR-JOB-jjb
        - this job is automatically generated and watches your repo for changes to jobs.yml. If it finds changes then it will rebuild YOUR-JOB accordingly.
    - YOUR-JOB-jvb
        - this job is automatically generated and watches your repo for changes to views.yml. If it finds changes then it will create YOUR-VIEWS as defined.
    - YOUR-JOB
        - Then whole reason for doing this.

## **MIGRATING JOBS**
Most of us already have at least some existing Jenkins jobs and the reason you may be reading this is because setting them up (again) is a pain.

That's where this wonderful tool comes in - [Jenkins Jobs Wrecker](https://github.com/ktdreyer/jenkins-jobs-wrecker)!
It will pull an existing Jenkins job and convert it to Jenkins Job Builder YAML syntax without touching the original!

- Install Jenkins Jobs Wrecker
  - `pip install jenkins-job-wrecker`
- Copy the binary for easy access
  - `sudo cp -R .local/bin/jjwrecker /usr/bin/jjwrecker`
- Convert a job!
  - `JJW_USERNAME=$jenkins-login-username JJW_PASSWORD=$jenkins_API_token` jjwrecker -s http:YOUR-JENKINS-URL:PORT/ -N 'NAME-OF-AN-EXISTING-JENKINS-JOB-TO-CONVERT'
- Check it out
  - `ls $PWD/output`

## **WRAP-UP**
If you're interested in seeing a more interesting JJB job, I've got my Jenkins backup job [here](https://github.com/chrisanthropic/jenkins-backup-job).

If you look at the jobs.yml in my jjb-test repo, you'll see a commented out job at the bottom. It's an example of JJB _using_ the Jobs DSL. That's right, you can use this setup to call Jobs DSL defined jobs as well!

## **REPOS**
For your convenience here's a list of the repos created for this tutorial.

- [jenkins-job-creator-seed](https://github.com/chrisanthropic/jenkins-job-creator-seed)
- [jjb-test](https://github.com/chrisanthropic/jjb-test)
- [jenkins-backup-job](https://github.com/chrisanthropic/jenkins-backup-job)
