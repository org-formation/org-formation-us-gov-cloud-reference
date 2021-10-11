# IMPORTANT NOTE

This reference architecture and also the steps outlined in this review are still under development and wont work as-is.

Please do
 - watch this project if you would like to get updates on the process.
 - contribute best practices / ideas / documentation / code through issues or PRs <3

thanks!

# Introduction
This is a reference architecture to get started quickly with using `org-formation` in the GovCloud (US) partition as well as provide examples of best practices and demonstrates the capabilities. The reference architecture is built with the following core principles:
* _Batteries included:_ it should deploy with as little dependencies as possible
* _Always relevant:_ every organization will get value out of every part of the reference architecture. This means two things
  * it should not contain services that might not be used. Examples: **AWS ECS**, **AWS Step Functions**
  * it should not be too opinionated about choosing an AWS service as the implementation for a function that can also be sourced externally. Examples: **AWS SecurityHub**, **AWS CloudWatch** logs and metrics

To use this reference architecture, follow the [Getting Started](#getting-started). If you want to deploy additional stacks using org-formation on top of this reference architecture, then it is advised to do that using _delegated builds_. Within the org-formation GitHub there will be example stacks configured as delegated builds that are usable, but are not eligeble to be part of the reference architecture because they either depend on an external system (not batteries included) or not always relevent (uses an AWS service that does something that is not always relevant). Examples are:
* Break glass notifications based on events
* DNS and Domain management

# Getting Started

To create an AWS Organization based on this reference architecture, managed by org-formation follow these steps end to end.

1. [Create the AWS Management Account](#1-create-the-aws-management-account)
2. [Create the GovCloud (US) Management Account](#2-create-the-govcloud-us-management-account)
3. [Create the AWS Organization](#3-create-the-aws-organization)
4. [Configure AWS SSO](#4-configure-aws-sso)
5. [Clone and modify this repository](#5-clone-and-modify-this-repository)
6. [Initialize `org-formation`](#6-initialize-org-formation)

## Prerequisites
1. A valid credit card
2. A working phone number
3. Four unique email addresses within your domain (management-root, compliance-root and orgbuild-root)

> _Hint: if you are using Google as an email provider, you can use team+something@domain.com to create unique addresses that all arrive in the same email box_

## 1. Create the AWS Management Account
Create an AWS Account. This will be the management account of your commercial AWS Organization

1. Navigate [here](https://portal.aws.amazon.com/billing/signup?nc2=h_ct&src=default&redirect_url=https%3A%2F%2Faws.amazon.com%2Fregistration-confirmation#/start) to sign up to an AWS account. Or sign-up on the page [here](https://aws.amazon.com/)
2. Gather the following data that you will need in step [Clone and modify this repository](#4.-clone-and-modify-this-repository)

| Parameter | Where to find it | Example
| ----- | ----------- | ---
| {{management-account-id}} | Top right -> your account -> My Account | 341696816352

## 2. Create the GovCloud (US) Management Account
GovCloud (US) accounts are tied to your commercial account. This will be the management account of your GovCloud (US) AWS Organization.

1. While authenticated into your commercial account from step 1 navigate [here](https://portal.aws.amazon.com/billing/signup#/govcloud) to sign up for a GovCloud (US) account.  Once requested you will have to wait until it is approved.
2. Once approved follow the steps to create your `Administrator` user to access your GovCloud (US) account. Once you set your password you can sign into your account [here](https://signin.amazonaws-us-gov.com/)
2. Gather the following data that you will need in step [Clone and modify this repository](#4.-clone-and-modify-this-repository)

| Parameter | Where to find it | Example
| ----- | ----------- | ---
| {{gov-cloud-management-account-id}} | Top right -> your account -> My Account | 082782365842

## 3. Create the AWS Organization in the commercial and govcloud (us) partitions

1. [Create an AWS Organization](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_create.html)
2. [Enable all features](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_org_support-all-features.html)
3. [Enable all policy types](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_enable-disable.html)

## 4. Configure AWS SSO
You will use AWS SSO to obtain access to the AWS Accounts after we are done with the initial setup. IAM Users including the root user should not be used as a security best practice for human access.

1. [Enable AWS SSO](https://docs.aws.amazon.com/singlesignon/latest/userguide/step1.html)
2. [Create 4 groups](https://docs.aws.amazon.com/singlesignon/latest/userguide/addgroups.html)
   - Admin
   - Developer
   - Auditor
   - Supporter
3. [Create 1 user](https://docs.aws.amazon.com/singlesignon/latest/userguide/addusers.html)
   - Set a password
   - Enroll 2FA
   - Add user to the Administrator group
4. Gather the following data you will need in steps [Clone and modify this repository](#4.-clone-and-modify-this-repository) and [Clone your new `org-formation` repository](#6.-clone-your-new-org-formation-repository)

> _Note: you will need to run these steps in both the commercial and govcloud (US) partitions.  Make sure to obtain the following information._

| Parameter | Description | Example
| ----- | ----------- | ---
| SSO start URL | Go to AWS SSO -> Settings -> User Portal -> User Portal URL | https://dgega332fa.awsapps.com/start
| {{commercial-sso-instance-arn}} | Go to AWS SSO -> Settings -> ARN | arn:aws:sso:::instance/ssoins-6987b39db64e1ecd
| {{sso-admin-group-id}} | Go to AWS SSO -> Groups -> Administrator -> Details -> Group ID | 99672ac0cf-8495fd69-c57e-4214-88a4-b9f41eed0d32
| {{commercial-sso-auditor-group-id}} | Go to AWS SSO -> Groups -> Auditor -> Details -> Group ID | 99672ac0cf-8495fd69-c57e-4214-88a4-b9f41eed0d32
| {{commercial-sso-developer-group-id}} | Go to AWS SSO -> Groups -> Developer -> Details -> Group ID | 99672ac0cf-8495fd69-c57e-4214-88a4-b9f41eed0d32
| {{commercial-sso-supporter-group-id}} | Go to AWS SSO -> Groups -> Supporter -> Details -> Group ID | 99672ac0cf-8495fd69-c57e-4214-88a4-b9f41eed0d32
| | | 
| SSO start URL | Go to AWS SSO -> Settings -> User Portal -> User Portal URL | https://dgega332fa.awsapps.com/start
| {{govcloud-sso-instance-arn}} | Go to AWS SSO -> Settings -> ARN | arn:aws:sso:::instance/ssoins-6987b39db64e1ecd
| {{govcloud-sso-admin-group-id}} | Go to AWS SSO -> Groups -> Administrator -> Details -> Group ID | 99672ac0cf-8495fd69-c57e-4214-88a4-b9f41eed0d32
| {{govcloud-sso-auditor-group-id}} | Go to AWS SSO -> Groups -> Auditor -> Details -> Group ID | 99672ac0cf-8495fd69-c57e-4214-88a4-b9f41eed0d32
| {{govcloud-sso-developer-group-id}} | Go to AWS SSO -> Groups -> Developer -> Details -> Group ID | 99672ac0cf-8495fd69-c57e-4214-88a4-b9f41eed0d32
| {{govcloud-sso-supporter-group-id}} | Go to AWS SSO -> Groups -> Supporter -> Details -> Group ID | 99672ac0cf-8495fd69-c57e-4214-88a4-b9f41eed0d32

Configuring AWS SSO manually using AWS SSO itself as an identity provider is the quickest and easiest way to get started (and is batteries included). At the time of writing AWS SSO does not support automating this via an API, as soon as it does, this guide will be updated accordingly. If you want to configure an external Identity Provider, then start [here](https://docs.aws.amazon.com/singlesignon/latest/userguide/supported-idps.html).

## 5. Clone and modify this repository
1. Clone this repository locally
2. Search and replace the following values globally in all files

| Parameter | Description | Source | Example
| --- | --- | --- | ---
| {{management-account-id}} | 12 digit identifier of the management account | [Create the AWS Management Account](#1-create-the-aws-management-account) | 341696816352
| {{gov-cloud-management-account-id}} | 12 digit identifier of the management account | [Create the AWS Management Account](#1-create-the-aws-management-account) | 082782365842
| {{organization-name}} | Alias of the management account | choose | bee-awesome
| {{primary-commercial-aws-region}} | The primary AWS commercial region to deploy to | choose | us-east-1
| {{primary-govcloud-aws-region}} | The primary AWS govcloud region to deploy to | choose | us-gov-west-1
| {{management-root-email-address}} | Email address used to register the management account | [Create the AWS Management Account](#1-create-the-aws-management-account) | platform.team@bee.awesome
| {{security-root-email-address}} | Email address for the security account | [Prerequisites](#prerequisites) | platform.team+security@bee.awesome
| {{log-archive-root-email-address}} | Email address for the log archive account | [Prerequisites](#prerequisites) | platform.team+logs@bee.awesome
| {{commercial-sso-instance-arn}} | AWS SSO instance ARN | [Configure AWS SSO](#3-configure-aws-sso) | arn:aws:sso:::instance/ssoins-6987b39db64e1ecd
| {{commercial-sso-admin-group-id}} | Principal ID from Identity Provider's group used by administrators | [Configure AWS SSO](#3-configure-aws-sso) |99672ac0cf-8495fd69-c57e-4214-88a4-b9f41eed0d32
| {{commercial-sso-auditor-group-id}} | Principal ID from Identity Provider's group used by auditors | [Configure AWS SSO](#3-configure-aws-sso) | 99672ac0cf-8495fd69-c57e-4214-88a4-b9f41eed0d32
| {{commercial-sso-developer-group-id}} | Principal ID from Identity Provider's group used by auditors | [Configure AWS SSO](#3-configure-aws-sso) | 99672ac0cf-8495fd69-c57e-4214-88a4-b9f41eed0d32
| {{commercial-sso-supporter-group-id}} | Principal ID from Identity Provider's group used by supporters | [Configure AWS SSO](#3-configure-aws-sso) | 99672ac0cf-8495fd69-c57e-4214-88a4-b9f41eed0d32
| {{govcloud-sso-instance-arn}} | AWS SSO instance ARN | [Configure AWS SSO](#3-configure-aws-sso) | arn:aws:sso:::instance/ssoins-6987b39db64e1ecd
| {{govcloud-sso-admin-group-id}} | Principal ID from Identity Provider's group used by administrators | [Configure AWS SSO](#3-configure-aws-sso) |99672ac0cf-8495fd69-c57e-4214-88a4-b9f41eed0d32
| {{govcloud-sso-auditor-group-id}} | Principal ID from Identity Provider's group used by auditors | [Configure AWS SSO](#3-configure-aws-sso) | 99672ac0cf-8495fd69-c57e-4214-88a4-b9f41eed0d32
| {{govcloud-sso-developer-group-id}} | Principal ID from Identity Provider's group used by auditors | [Configure AWS SSO](#3-configure-aws-sso) | 99672ac0cf-8495fd69-c57e-4214-88a4-b9f41eed0d32
| {{govcloud-sso-supporter-group-id}} | Principal ID from Identity Provider's group used by supporters | [Configure AWS SSO](#3-configure-aws-sso) | 99672ac0cf-8495fd69-c57e-4214-88a4-b9f41eed0d32

## 6. Initialize `org-formation`

In this step, you run OrgFormation locally using the credentials of the root user of the management account. This is the only time you will use the root user of any account for any purpose and the root user will be closed off as the last step in this guide.

> _Note: you might need to run the OrgFormation commands more than once._

<br>

1. Install OrgFormation on your local machine
```
npm install -g aws-organization-formation
```

2. Apply the organization.yml file to AWS Organizations. This means creating accounts and OUs and ordering them. This will also create the OrgBuild account where the org-formation build pipeline will reside.
```
npx org-formation update ./src/organization.yml --partition-region us-gov-west-1 --partition-profile myGovProfile --verbose
```

3. Run tasks on either the commercial or govcloud partition.
```
npx org-formation perform-tasks ./src/_commercial-tasks.yml
```
**or**  
```
npx org-formation perform-tasks ./src/_govcloud-tasks.yml --is-partition --partition-region us-gov-west-1 --partition-profile myGovProfile
```