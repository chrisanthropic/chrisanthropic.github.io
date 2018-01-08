---
layout: post
title: "Ansible Loops and AWS EC2, Now With Working Tags."
keywords:
description:
thumbnail:
facebook_type:
facebook_image:
---

I've been playing with Ansible and have been working on using their awesome 'loop' feature. What this allows me to do is define a task such as creating an EC2 instance, and then loop over that task multiple times, using a file of specified variables as the input.

This allows me to simply specify the variables I want for as many VMs as I want, without having to touch another file.

The problem I ran into though, is a quirk with how Ansible collects the data from a loop when registering it as a variable.

My tasks look like this:
- Loop through my variables and create the specified Security Groups
- Loop through my variables and create the specified EC2 instances, placing them into the previously created Security Groups. Tag the instances.
- Loop through my variables and tag the Security Groups with identical tags as the EC2 instances.

The problem? I couldn't find a way to successfully capture the IDs of the security groups as they were created, and I *need* to have that ID in order to successfully tag it.

After far too much time tinkering around, I finally came up with a working solution. 
First, I run a task with a shell command to hit the AWS API and return the security IDs of all instances, filtering them by tags to only include the ones I just created.

Then I register that output as a new variable.

Lastly, I loop my 'tag' task for every security group in the variable.

The main bit of code is here:

{% raw %}
```
- shell: aws ec2 describe-instances --query 'Reservations[*].Instances[*].SecurityGroups[*].GroupId' --filter Name=tag:Name,Values="{{ stack }}"_VM_*_"{{ env }}" --output text | uniq
  with_items: VMs
  register: groupIDs
```
{% endraw %}

And then my tag task is here (with the last line being the magic):

{% raw %}
```
# Add tags to the Security Group
- ec2_tag:
      resource: "{{ item }}"
      region: "{{ region }}"
      state: present
      tags:
        Name: "sg_{{ stack }}_VM_{{ env }}"
        env: "{{ env }}"
        event: "{{ stack }}"
        owner: "{{ owner }}"
  with_items: groupIDs.results[0].stdout_lines
```
{% endraw %}

You can see the whole thing in action and the entire Ansible setup in the repo I created here:[https://github.com/chrisanthropic/ansible-aws-template](https://github.com/chrisanthropic/ansible-aws-template)

And that's how I finally managed to use Ansible to loop a task, register the output as a variable, and then loop that output. Do you have a better way to accomplish the same thing? Let me know about it.
