**Note**: This public repo contains the documentation for the private GitHub repo <https://github.com/gruntwork-io/module-security>.
We publish the documentation publicly so it turns up in online searches, but to see the source code, you must be a Gruntwork customer.
If you're already a Gruntwork customer, the original source for this file is at: <https://github.com/gruntwork-io/module-security/blob/master/modules/aws-config/core-concepts.md>.
If you're not a customer, contact us at <info@gruntwork.io> or <http://www.gruntwork.io> for info on how to get access!

# AWS Config Core Concepts

## Background

### What is AWS Config?
Config monitors your AWS resources (such as EC2 instances, security groups, EBS volumes, CloudFront Distributions, and [a whole lot more](https://docs.aws.amazon.com/config/latest/developerguide/resource-config-reference.html)) for configuration changes. It tracks these changes over time, and can track whether configurations are in compliance with a standard configuration. If the configuration drifts out of compliance, Config can send a notification. You can view and query Config items in the AWS Config console.

### What are Config Rules?
Config rules are expressions of a desired configuration state, written in code and executed as Lambda functions. When a resource configuration changes, AWS Config fires the relevant Lambda functions to evaluate whether the configuration changes the state of compliance with the desired configuration. AWS has developed a set of pre-written rules called [AWS Config Managed Rules](https://docs.aws.amazon.com/config/latest/developerguide/managed-rules-by-aws-config.html), but you can also author your own [custom rules](https://docs.aws.amazon.com/config/latest/developerguide/evaluate-config_develop-rules_nodejs.html).

This module enables AWS Config but does not manage or enable any Config Rules.

## What resources does this module create?

This module creates the requisite elements to enable AWS Config in a given region. The steps include:

1. Create a [Configuration 
   Recorder](https://docs.aws.amazon.com/config/latest/developerguide/config-concepts.html#config-recorder).
1. Create an S3 Bucket and an SNS Topic to be used by AWS Config to deliver configuration 
   [snapshots](https://docs.aws.amazon.com/config/latest/developerguide/config-concepts.html#config-snapshot) and 
   [streams](https://docs.aws.amazon.com/config/latest/developerguide/config-concepts.html#config-stream).
1. Enable the configuration recorder.

To implement these steps, this module creates the following resources:

- **aws_s3_bucket**: An S3 bucket used by AWS Config to store configuration items.
- **aws_sns_topic**: An SNS topic for notifications from AWS Config.
- **aws_iam_role**: An IAM role allowing the Config service to access the supported resources as well as to put S3 objects in the aforementioned bucket and publish notifications to the SNS topic.
- **aws_config_configuration_recorder**: A configuration recorder that records resource configurations.
- **aws_config_delivery_channel**: A delivery channel with the previously noted S3 bucket and SNS destinations.
- **aws_config_configuration_recorder_status**: A resource to enable the configuration recorder.

The module does not create and manage [Config 
Rules](https://docs.aws.amazon.com/config/latest/developerguide/config-concepts.html#aws-config-rules) or 
[Aggregators](https://docs.aws.amazon.com/config/latest/developerguide/config-concepts.html#multi-account-multi-region-data-aggregation).

**Note**: AWS Config must be enabled on a per-region basis. For a complete view of your AWS resources, use this module 
within each region that is enabled in your account.

## Day-to-day operations

### What does a configuration item look like, and how do I view it?
A [configuration item](https://docs.aws.amazon.com/config/latest/developerguide/config-item-table.html) is a JSON-encoded description of configuration change to a resource. Configuration items are delivered by AWS Config each time a resource is created, modified, or deleted. The following snippet is an example of a configuration item (edited for brevity):

```json
{
  "configurationItemDiff": {
    "changedProperties": {
      "Configuration.IpPermissions.1": {
        "updatedValue": {
          "fromPort": 22,
          "ipProtocol": "tcp",
          "toPort": 22,
          "ipv4Ranges": [ { ... } ],
          "ipRanges": [ ... ]
        },
        "changeType": "CREATE"
      },
      "Configuration.IpPermissions.2": {
        "previousValue": null,
        "updatedValue": {
          "fromPort": 80,
          "ipProtocol": "tcp",
          "ipv6Ranges": [],
          "prefixListIds": [],
          "toPort": 80,
          "userIdGroupPairs": [],
          "ipv4Ranges": [ { ... } ],
          "ipRanges": [ ... ]
        },
        "changeType": "CREATE"
      },
      "Configuration.IpPermissions.0": {
        "previousValue": {
          "fromPort": 22,
          "ipProtocol": "tcp",
          "toPort": 22,
          "ipv4Ranges": [ { "cidrIp": "0.0.0.0/0" } ],
          "ipRanges": [ "0.0.0.0/0" ]
        },
        "changeType": "DELETE"
      }
    },
    "changeType": "UPDATE"
  },
  "configurationItem": {
    "relationships": [
      {
        "resourceId": "vpc-09a90003b04281036",
        "resourceName": null,
        "resourceType": "AWS::EC2::VPC",
        "name": "Is contained in Vpc"
      }
    ],
    "configuration": {
      "description": "An Example Security Group",
      "groupName": "ExampleGroup",
      ...
      "groupId": "sg-040febc38b5233298",
      ],
      "vpcId": "vpc-09a90003b04281036"
    },
    "configurationItemVersion": "1.3",
    "configurationItemCaptureTime": "2019-08-22T20:35:49.316Z",
    "configurationStateId": 1566506149316,
    "configurationItemStatus": "OK",
    "resourceType": "AWS::EC2::SecurityGroup",
    "resourceId": "sg-040febc38b5233298",
    "ARN": "arn:aws:ec2:us-east-1::security-group/sg-040febc38b5233298",
    "awsRegion": "us-east-1",
    "configurationStateMd5Hash": "",
  },
  "notificationCreationTime": "2019-08-22T20:35:49.815Z",
  "messageType": "ConfigurationItemChangeNotification",
  "recordVersion": "1.3"
}
```

The example shows crucial information about how the configuration of a security group has changed. It shows the previous ingress rule configuration, new ingress rule configuration, and the relationship of the security group to other AWS resources, along with some metadata and resource attributes.

### How does Config work with multiple AWS accounts and multiple regions?
AWS Config must be enabled on a per-region basis. Once enabled, multiple regions (and accounts) can be combined using the [data aggregation](https://docs.aws.amazon.com/config/latest/developerguide/aggregate-data.html) features. Multi-account/region Config works both with several individual accounts and with AWS Organizations.

This module enables Config for a single region. To enable Config across multiple regions, call this module once for each desired region. This is considerably easier to accomplish with [terragrunt](https://github.com/gruntwork-io/terragrunt).

To consolidate multiple regions and accounts, use the [aws_config_configuration_aggregator](https://www.terraform.io/docs/providers/aws/r/config_configuration_aggregator.html) resource in the desired destination account/region, and use the [aws_config_aggregate_authorization](https://www.terraform.io/docs/providers/aws/r/config_aggregate_authorization.html) resource in the desired source accounts/regions.

For example, if you wish to aggregate regions `us-east-1` and `eu-west-3` from account `012345678901` to region `eu-west-1` in account `123456789012`, you would first run `terraform apply` on account `123456789012` using the following Terraform code:

```
provider "aws" {
  region = "eu-west-1"
}

resource "aws_config_configuration_aggregator" "destination_account" {
  name = "AggregationExample"

  account_aggregation_source {
    account_ids = ["012345678901"]
    regions     = ["us-east-1", "eu-west-3"]
  }
}
```

Then you would run `terraform apply` on account `012345678901` using this Terraform code:

```
resource "aws_config_aggregate_authorization" "source_account" {
  account_id = "123456789012"
  region     = "eu-west-1"
}
```

Once authorized, the resources from the source regions will begin to appear in the AWS Config console at the destination.


### How can I be alerted by AWS Config when the configuration of a resource changes?
Configuration items (e.g. changes in configuration) are sent to the SNS topic associated with the config recorder. You can [subscribe to the SNS topic](https://docs.aws.amazon.com/sns/latest/dg/welcome.html) using the technique of your choice.
