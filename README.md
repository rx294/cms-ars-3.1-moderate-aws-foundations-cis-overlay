# cms-ars3.1-cis-aws-foundations-baseline
CMS ARS 3.1 Overlay InSpec Validation Profile for CIS AWS Foundations Benchmark

Based on Inspec profile for CIS Amazon Web Services Foundations Benchmark v1.1.0 - 11-29-2016

## Description

This [InSpec](https://github.com/chef/inspec) compliance profile implements the [CIS AWS Foundations Benchmark](https://github.cms.gov/ispg-review/cis-aws-foundations-baseline) in an automated way to provide security best-practice tests in an AWS environment.

InSpec is an open-source run-time framework and rule language used to specify compliance, security, and policy requirements for testing any node in your infrastructure.

## Requirements

- [InSpec](http://inspec.io/) at least version 2.1
- [AWS CLI](https://aws.amazon.com/cli/) at least version 2.x

### Tested Platforms

This profile is being developed and tested along side a `hardening` recipe implemented in Terraform. The [cis-aws-foundations-hardening](https://github.com/aaronlippold/cis-aws-foundations-hardening) will help you configure and deploy your AWS environment to meet the requirements of the security baseline.

## Get started

### Installing InSpec 

If needed - install inspec on your 'runner' system - i.e. your orchestration server, your workstation, your bastion host or your instance you wish to evlauate.

  a. InSpec has prepackaged installers for all platforms here: https://www.inspec.io/downloads, or 
  
  b. If you already have a ruby environment (`2.4.x`) installed on your 'runner' system - you can just do a simple `gem install inspec`, or 
  
  c. You can use the AWS SSM suite to run InSpec on your AWS assets - see the InSpec + SSM documation here: https://aws.amazon.com/blogs/mt/using-aws-systems-manager-to-run-compliance-scans-using-inspec-by-chef/
  
### Get the CMS AWS Foundations Baseline

You will need to download the InSpec Profile to your `runner` system. You can do this via `git` or the GitHub Web interface, etc.

  a. `git clone https://github.cms.gov/ispg-review/cms-ars3.1-cis-aws-foundations-baseline`, or 
  
  b. Save a Zip or tar.gz copy of the master branch from the `Clone or Download` button of this project

### Setting up dependencies in your Ruby and InSpec Environments

The profile uses Bundler to manage needed dependencies - so you will need to installed the needed gems via bundler before you run the profile. Change directories to your your cloned inspec profile then do a `bundle install`. 

  a. `cd cms-ars3.1-cis-aws-foundations-baseline` 
  
  b. `bundle install`

### Getting MFA Aware AWS Access, Secret and Session Tokens

You will need to ensure your AWS CLI environment has the right system environment variables set with your AWS region and credentials and session key to use the AWS CLI and InSpec resources in the CMS AWS environment.  InSpec supports the following standard AWS variables:

- `AWS_REGION`
- `AWS_ACCESS_KEY_ID`
- `AWS_SECRET_ACCESS_KEY`
- `AWS_SESSION_TOKEN`

In the CMS AWS environments - and any AWS MFA enabled enviroment - you need to use `derived credentials` to use the CLI. Your default `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` will not satisfy the MFA Policies in the CMS AWS environments. 

- The AWS documentation is here: https://docs.aws.amazon.com/cli/latest/reference/sts/get-session-token.html
- The AWS profile documentation is here: https://docs.aws.amazon.com/cli/latest/userguide/cli-multiple-profiles.html

A useful bash script for automating this is here: https://gist.github.com/dinvlad/d1bc0a45419abc277eb86f2d1ce70625

To generate credentials using an AWS Profile you will need to use the following AWS CLI commands.

  a. `aws sts get-session-token --serial-number arn:aws:iam::<$YOUR-MFA-SERIAL> --token-code <$YOUR-CURRENT-MFA-TOKEN> --profile=<$YOUR-AWS-PROFILE>` 

  b. Then export the `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY` and `AWS_SESSION_TOKEN` that was generated by the above command.

### Building your `attributes.yml` file

We use a yml attribute file to steer the configuration, the following options are available:  

The followiing attributes must be set to accepted/documented values which is 
then verified by the applicable controls.

These attributes are generated if the profile is used with the Terraform hardening receipe (https://github.com/aaronlippold/cis-aws-foundations-hardening) with kitchen-terraform.

- default aws key age (1.4), <br>
`aws_key_age: 60`

- Make the password length (1.9), <br>
`pwd_length: 12`

- make the aws_cred_age an attribute (1.11), <br>
`aws_cred_age: 60`

- description: 'default aws region', <br>
`default_aws_region: 'us-east-1'`

- description: 'default aws region', <br>
`aws_region: 'us-east-1'`

- description: 'iam manager role name',<br>
`iam_manager_role_name: "iam_manager_role_name"`

- description: 'iam master role name',<br>
`iam_master_role_name: "iam_master_role_name"`

- description: 'iam manager user name',<br>
`iam_manager_user_name: "iam_manager_user_name"`

- description: 'iam master user name',<br>
`iam_master_user_name: "iam_master_user_name"`

- description: 'iam manager policy name',<br>
`iam_manager_policy_name: "iam_manager_policy"`

- description: 'iam master policy name',<br>
`iam_master_policy_name: "iam_master_policy"`

- description: 'list of instances that have specific roles',<br>
`aws_actions_performing_instance_ids: ["aws_access_instance_id"]`

- description: 'Config service list and settings in all relevant regions',<br>
```
config_service:
    us-east-1: 
      s3_bucket_name: "s3_bucket_name_value"
      sns_topic_arn: "sns_topic_arn_value"
    us-east-2: 
      s3_bucket_name:  "s3_bucket_name_value"
      sns_topic_arn: "sns_topic_arn_value"
    us-west-1: 
      s3_bucket_name:  "s3_bucket_name_value"
      sns_topic_arn: "sns_topic_arn_value"
    us-west-2: 
      s3_bucket_name:  "s3_bucket_name_value"
      sns_topic_arn: "sns_topic_arn_value"

```
- description: 'SNS topics list and details in all relevant regions',<br>
```
sns_topics: 
    topic_arn1 : 
      owner : "owner_value"
      region : "region_value"
    topic_arn2 :
      owner : "owner_value"
      region : "region_value"`
```
- description: 'SNS subscription list and details in all relevant regions', <br>
```
sns_subscriptions: 
    subscription_arn1: 
      endpoint: "endpoint_value"
      owner: "owner_value"
      protocol: "protocol_value"
    subscription_arn2: 
      endpoint: "endpoint_value"
      owner: "owner_value"
      protocol: "protocol_value"`
```

## Generate Attributes

The repo includes a script : generate_attributes.rb to generate part of the attributes required for the profile.
The script will inspect aws regions: us-east-1, us-east-2, us-west-1, us-west-2 to generate the following attributes to STDOUT.

```
- config_delivery_channels
- sns_topics
- sns_subscriptions
```
The generated attributes __must be reviewed carefully__. 
Only __valid__ channels and sns items should be placed in the attributes.yml file.

Usage:
```
  ruby generate_attributes.rb
```
## Usage

InSpec makes it easy to run your tests wherever you need. More options listed here: [InSpec cli](http://inspec.io/docs/reference/cli/)

```
# Clone Inspec Profile
$ git clone https://github.cms.gov/ispg-review/cms-ars3.1-cis-aws-foundations-baseline

# Install Gems
$ bundle install

# Set required ENV variables
$ export AWS_ACCESS_KEY_ID=key-id
$ export AWS_SECRET_ACCESS_KEY=access-key
$ export AWS_SESSION_TOKEN=session_token

# Run the `generate_attributes.rb` 
$ ruby generate_attributes.rb
# The generated attributes __must be reviewed carefully__. 
# Only __valid__ channels and sns items should be placed in the attributes.yml file.

# To run profile locally and directly from Github
$ inspec exec /path/to/profile -t aws:// --attrs=attributes.yml

# To run profile locally and directly from Github with cli & json output 
$ inspec exec /path/to/profile -t aws:// --attrs=attributes.yml --reporter cli json:aws-results.json

# To run profile locally and directly from Github with cli & json output, in a specific region with a specific AWS profile
$ inspec exec /path/to/profile -t aws://us-east-1/<mycreds-profile> --attrs=attributes.yml --reporter cli json:aws-results.json
```

### Run individual controls

In order to verify individual controls, just provide the control ids to InSpec:

```
$ inspec exec /path/to/profile --attrs=attributes.yml --controls cis-aws-foundations-1.10
```

## Contributors + Kudos

- Rony Xavier [rx294](https://github.com/rx294)
- Aaron Lippold [aaronlippold](https://github.com/aaronlippold)

## License and Author

### Authors

- Author:: Rony Xaiver [rx294@gmail.com](mailto:rx294@gmail.com)
- Author:: Aaron Lippold [lippold@gmail.com](mailto:lippold@gmail.com)

### License 

* This project is dual-licensed under the terms of the Apache license 2.0 (apache-2.0)
* This project is dual-licensed under the terms of the Creative Commons Attribution Share Alike 4.0 (cc-by-sa-4.0)
