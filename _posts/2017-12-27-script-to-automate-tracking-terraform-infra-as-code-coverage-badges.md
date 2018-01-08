---
layout: post
title: "A scripted solution to tracking what percentage of your AWS resources are managed by Terraform (with coverage badges)."
keywords:
description:
thumbnail:
facebook_type:
facebook_image:
---
## What
[https://github.com/chrisanthropic/terraform-infra-as-code-coverage-badges](https://github.com/chrisanthropic/terraform-infra-as-code-coverage-badges)

I recently needed to bring a rather large existing project under control of Terraform. While Terraform does a great of job of simplifying the import process via a cli command, it's still difficult to track exactly which AWS resources are managed by Terraform and which ones aren't. So I wrote a script to analyze and create coverage badges for a single region at a time.

Here's an example of what they look like -

![ec2-instances-coverage](https://s3-us-west-2.amazonaws.com/terraform-infra-as-code-coverage-badges/us-east-1-ec2-instances-current-coverage.svg) ![ec2-sgs-coverage](https://s3-us-west-2.amazonaws.com/terraform-infra-as-code-coverage-badges/us-east-1-ec2-security-groups-current-coverage.svg) ![ec2-ami-coverage](https://s3-us-west-2.amazonaws.com/terraform-infra-as-code-coverage-badges/us-east-1-ec2-ami-current-coverage.svg) ![ec2-volumes-coverage](https://s3-us-west-2.amazonaws.com/terraform-infra-as-code-coverage-badges/us-east-1-ec2-volumes-current-coverage.svg) ![ec2-albs-coverage](https://s3-us-west-2.amazonaws.com/terraform-infra-as-code-coverage-badges/us-east-1-ec2-albs-current-coverage.svg) ![ec2-elbs-coverage](https://s3-us-west-2.amazonaws.com/terraform-infra-as-code-coverage-badges/us-east-1-ec2-elbs-current-coverage.svg) ![lambda-functions-coverage](https://s3-us-west-2.amazonaws.com/terraform-infra-as-code-coverage-badges/us-east-1-lambda-functions-current-coverage.svg) ![rds-instances-coverage](https://s3-us-west-2.amazonaws.com/terraform-infra-as-code-coverage-badges/us-east-1-rds-instances-current-coverage.svg) ![vpcs-coverage](https://s3-us-west-2.amazonaws.com/terraform-infra-as-code-coverage-badges/us-east-1-vpcs-current-coverage.svg) ![subnets-coverage](https://s3-us-west-2.amazonaws.com/terraform-infra-as-code-coverage-badges/us-east-1-subnets-current-coverage.svg) ![route-tables-coverage](https://s3-us-west-2.amazonaws.com/terraform-infra-as-code-coverage-badges/us-east-1-route-tables-current-coverage.svg) ![internet-gateways-coverage](https://s3-us-west-2.amazonaws.com/terraform-infra-as-code-coverage-badges/us-east-1-internet-gateways-current-coverage.svg) ![dhcp-option-sets-coverage](https://s3-us-west-2.amazonaws.com/terraform-infra-as-code-coverage-badges/us-east-1-dhcp-opts-current-coverage.svg) ![network-acls-coverage](https://s3-us-west-2.amazonaws.com/terraform-infra-as-code-coverage-badges/us-east-1-network-acls-current-coverage.svg) ![s3-buckets-coverage](https://s3-us-west-2.amazonaws.com/terraform-infra-as-code-coverage-badges/us-east-1-s3-buckets-current-coverage.svg) 

They're created by a simple bash script that:
- uses the AWS CLI to list all resources of a specific type (ie all EC2 instances)
- uses the AWS CLI to filter by all resources of a specific type that aren't tagged `Terraform: True`
  - (this tag is configurable via ENV VAR at the top of the script)
- does some basic arithmetic to calculate what percentage of that resource is tagged (and therefore managed by Terraform)
- uses the [http://shields.io/](http://shields.io/) API to create coverage badges
- writes the coverage badges to an S3 bucket you specify

The script expects that all of the resources managed by Terraform are tagged in a standard way; ie our example uses a `Terraform` tag, written by Terraform when it creates a resource that supports tags. This is how we ensure that all Terraform managed resources are tagged uniformly. The tag is configurable via ENV VARS at the top of the script.

For now it checks the following AWS resources:
- EC2 Instances
- EC2 Security Groups
- EC2 AMIs
- EC2 Volumes
- EC2 ALBs
- EC2 ELBs
- Lambda Functions
- RDS Instances
- VPCs
- VPC Subnets
- VPC Route Tables
- VPC IGWs
- VPC DHCP Options
- VPC Network ACLs
- S3 Buckets

## How to use the script
### Requirements
- An existing AWS account.
  - With permissions able to query all of the above listed resources
  - With permissions to read/write to S3 or a specified S3 bucket
- Locally configured [AWS profile](http://docs.aws.amazon.com/cli/latest/userguide/cli-multiple-profiles.html) with AWS credentials
- AWS resources that are consistently identified via a single tag
    - tag is configurable. Our example is "Terraform: True"
    - Any resource containing this tag is assumed to be managed via Terraform
- jq

### Do it
- `git clone` this repo
- configure the variables at the top of the script
  - AWS_PROFILE=""           `# The name of the AWS profile you configured in the requirements step above`
  - AWS_REGION="us-east-1"   `# The AWS region you want to test`
  - MANAGED_TAG="Terraform"  `# The tag you use to identify resources managed by Terraform`
  - BADGES_S3_BUCKET_NAME="" `# Name of a new or existing S3 bucket you want to store the bages in`
- run the script
  - it will make the AWS API calls, checking all AWS resources in the specified region of your specified account for the existence of the specified tag.
  - it will calculate the total number of resources vs the total number of tagged resources
  - it will use the output of the above function as the input for the badges.io API to create coverage badges
  - it will write the badges to the specified S3 bucket
- you can point to the URL of the S3 badges in order to embed anywhere you want, see above Demo for an example.
- Optionally configure your CI tool to run the script on every PR to ensure the badges are always up to date.

## Badges
### Colors
Badges are color coded based on the coverage percentage:
- Green: 100% coverage
- Yellow: 67-99%
- Orange: 34-66%
- Red: 0-33%

### S3
The script uses the AWS CLI to transfer the badges to an S3 bucket of your choice, with the following settings:
- `public-read` allows anyone to read the badge files.
- `--cache-control max-age=60` tells browsers not to cache the file for longer than 60 seconds. This is to get around Github caching the badge images which prevents them from updating.

### CI
I set up Travis-CI to run this script every time the repo containing the terraform code is merged. This triggers the badges to be rebuilt so they're always up to date.

## Wrap-up
I plan to support all AWS resources that Terraform has tag support for. I also have an idea on how to expand it to AWS resources that don't have tag support so that we could eventually cover all AWS resources.

My next post will include information about my steps in automating the Terraform import process while importing an existing Github organization to a fully Terraform managed Github org.
