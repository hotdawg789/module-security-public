**Note**: This public repo contains the documentation for the private GitHub repo <https://github.com/gruntwork-io/module-security>.
We publish the documentation publicly so it turns up in online searches, but to see the source code, you must be a Gruntwork customer.
If you're already a Gruntwork customer, the original source for this file is at: <https://github.com/gruntwork-io/module-security/blob/master/modules/cloudtrail/core-concepts.md>.
If you're not a customer, contact us at <info@gruntwork.io> or <http://www.gruntwork.io> for info on how to get access!

# AWS CloudTrail Core Concepts

## Background

### What is CloudTrail?

From the [AWS documentation](http://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-user-guide.html):

> You can use AWS CloudTrail to get a history of AWS API calls and related events for your account. This includes calls
made by using the AWS Management Console, AWS SDKs, command line tools, and higher-level AWS services.

### Why use CloudTrail?

An important element of cloud security is auditing all AWS API calls. This can give you key insight into when a new port
was opened, when a new IAM User account was created, and other security-significant events. While it's trivially easy to
turn on CloudTrail, thinking through where to store CloudTrail logs, how to guarantee CloudTrails logs aren't tampered
with, and getting alerts when certain API events occur is non-trivial.

Note that *any* interaction with AWS ultimately results in an API call. This includes using the AWS Web Console, awscli,
and any of the various AWS SDKs.

### What is a CloudTrail Trail?

To setup CloudTrail, you enable an individual "Trail" which has certain configuration options. For example, a Trail could
be configured to collect logs from just a single AWS Region or all AWS Regions, it could optionally log to CloudWatch Logs,
optionally use a KMS Key, etc.

In practice, most teams will use a single CloudTrail Trail.

### What's the difference between CloudTrail and AWS Config?

CloudTrail records every API call event in log files and provides convenient ways of viewing those logs. It treats the
*API Call* as a first-class entity.

AWS Config tracks changes to individual AWS resources and can alert you when a change is made that violates a defined
policy. It treats the *AWS Resource* as a first-class entity.

Naturally, an API Call typically operates on a particular AWS Resource, so the AWS Web Console includes links to the
corresponding AWS Resource in AWS Config (and vice versa).  

### CloudTrail Threat Model

The following are the various parts of the cloudtrail module threat model:

- **Deleting the S3 Bucket that stores CloudTrail events.** An IAM User who has permissions to delete the S3 Bucket that
  contains all CloudTrail events can delete all historical log data by deleting the S3 Bucket. Interestingly, CloudTrail
  will still show the last 7 days of create, modify, and delete API calls in the AWS Web Console, even after the S3
  Bucket is deleted.

  One way to protect against this is to set an alert if the S3 Bucket is deleted. In addition, an attacker could
  delete the alert (!) so we also need an alert if any alerts are themselves deleted.

  To assist with access auditing, you can also enable S3 Access Logs so that any access or modification of the CloudTrail S3
  bucket will be logged to the separate S3 bucket. Of course, you'll need to ensure that the same user does not have access
  to the access logs bucket.

  Another option is to enable [S3 Cross-Region Replication](http://docs.aws.amazon.com/AmazonS3/latest/dev/crr.html) in
  a cross-account scenario. That is, all S3 data is asynchronously replicated to another S3 Bucket in another AWS Account
  which you limit access to a very small number of trusted parties.

  Yet another option is to enable CloudWatch Logs integration so that CloudTrail events are published to a CloudWatch Logs
  group. The same deletion risk exists for an IAM user who has access to CloudWatch Logs. This is one reason among many
  why it is recommended to limit IAM access and prevent full administrator rights to any IAM user.

## Resources Created

This module creates the following AWS resources:

- **aws_s3_bucket:** An S3 Bucket where all CloudTrail data is stored. You can configure for how long CloudTrail data
  is stored.
- **aws_kms_key:** A KMS Customer Master Key (CMK) that encrypts all CloudTrail data. In order to view any read API log
  entries, or any API log entries older than 7 days, a user must have "decrypt" rights on this Key.
- **aws_cloudtrail:** We create a CloudTrail "Trail" that records log data from all AWS Regions and also writes a hash
  of all log entries to S3 to allow for log file validation.
- **aws_cloudwatch_log_group:** (optional) A CloudWatch Logs group where CloudTrail can publish events.
- **aws_iam_role:** (optional) An IAM role for CloudTrail to use for publishing events to CloudWatch Logs.




## Operations

### Where are CloudTrail logs stored?

All CloudTrail log data is stored as discrete files in S3. This module configures those files to be encrypted with a KMS
Customer Master Key (CMK) used exclusively for CloudTrail.

### What kind of data do CloudTrail log entries contain?

Here's an example CloudTrail entry:

```json
{
            "eventVersion": "1.04",
            "userIdentity": {
                "type": "IAMUser",
                "principalId": "AIDAIS2WCCNPT35B4US6W",
                "arn": "arn:aws:iam::123456789012:user/josh@phoenixdevops.com",
                "accountId": "087285199408",
                "accessKeyId": "AKIAJIMXOF147AFZDOXB",
                "userName": "josh@gruntwork.io"
            },
            "eventTime": "2016-09-29T17:16:33Z",
            "eventSource": "kms.amazonaws.com",
            "eventName": "DescribeKey",
            "awsRegion": "us-west-2",
            "sourceIPAddress": "12.34.567.890",
            "userAgent": "terraform/0.7.4 aws-sdk-go/1.4.10 (go1.7.1; darwin; amd64)",
            "requestParameters": {
                "keyId": "685be88d-5693-4c0e-ba64-2bab5d9806f3"
            },
            "responseElements": null,
            "requestID": "785ead04-8668-11e6-8bd5-19543dc40616",
            "eventID": "6385bc87-a304-4358-9b65-50ece1898daa",
            "readOnly": true,
            "resources": [
                {
                    "ARN": "arn:aws:kms:us-west-2:087285199408:key/685be88d-5693-4c0e-ba64-2bab5d9806f3",
                    "accountId": "087285199408"
                }
            ],
            "eventType": "AwsApiCall",
            "recipientAccountId": "087285199408"
        },
```

As you can see, pretty much all relevant information is included in the call, including the identity of the requester,
the user agent, and the service being called.

### What's the best way to view CloudTrail Log Data?

One way to view CloudTrail log data is just to download files directly from S3. That's how I obtained the example above.
The other way to view log data is directly in the AWS Web Console, which provides a convenient interface for viewing and
filtering all API activity for create, modify, and delete API calls for the last 7 days only.

To view read API calls, or any data older than 7 days, you must access it directly in S3.

Another approach is to view the logs in CloudWatch Logs. If the trail is configured to publish to a CloudWatch Logs group, each
event as described above is visible as a separate entry in the log group.

*Tip: IAM Users with IAM-granted access to CloudTrail can still view some data. See the [Gotchas](#gotchas) for details.*

### Can you get alerted when certain API events occur?

Yes. It's possible to configure CloudTrail to also log events in [CloudWatch Logs](http://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/WhatIsCloudWatchLogs.html).
You can then configure CloudWatch Logs to emit a custom CloudWatch metric for certain events, and then create a CloudWatch
alarm that notifies an SNS Topic when those events occur.

This module does not currently support those features, but if you'd like us to add them please email info@gruntwork.io.




## Gotchas

1. Even if an IAM User doesn't have access to the KMS Key used to encrypt all CloudTrail log data, if they have the
   appropriate IAM Policy, they can still view a subset of CloudTrail events in the AWS Web Console! Specifically, the
   AWS Web Console shows most API activity for create, modify, and delete API calls for the last 7 days only. Note that some activity is not visible, such as SSM calls. Viewing
   any other CloudTrail data does require access to the KMS Key.

1. CloudTrail logs are not delivered instantly to the S3 Bucket. There's typically a delay of a few minutes. While AWS
   documentation states that it takes 15 minutes, in practice we've found it to be around 5 minutes in most cases.

1. CloudTrail records not only API calls made by human IAM Users, but also API calls made by IAM Roles, and even API calls
   made on behalf of an IAM User by internal AWS Services. You can identify this last type of API call by looking for an
   `invokedBy` property in the CloudTrail event. For example:

   ```
   "invokedBy":"signin.amazonaws.com"
   ```

1. If you provide an existing S3 bucket rather than allow this module to create one, you'll need to apply the [recommended
S3 Bucket Policy](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/create-s3-bucket-policy-for-cloudtrail.html).
Furthermore, if you provide an existing S3 bucket, the module will not enable access logging even if you use
`enable_s3_server_access_logging=true`. The access logging bucket will be created, but you'll need to manually enable logging
in the bucket properties of the existing bucket.




## Known Issues

- Terraform has a bug where you will see a diff on the S3 lifecycle rules on every `terraform apply` or `terraform plan`. See
  [#9119](https://github.com/hashicorp/terraform/issues/9119) to track progress on this issue.

- If you attempt to enable or disable archiving of log files, Terraform will attempt to delete and re-create your existing S3
  Bucket. This will fail because your Bucket won't be empty, but more importantly, the goal here is just to change some S3
  Lifecycle Rules, so a destroy/re-create is unnecessary.

  Here's what the `terraform plan` output looks like if you attempt to disable archiving:

  ```
  - module.cloudtrail.aws_s3_bucket.cloudtrail_with_logs_archived

  + module.cloudtrail.aws_s3_bucket.cloudtrail_with_logs_not_archived
      acceleration_status:                                                 "<computed>"
      acl:                                                                 "private"
      arn:                                                                 "<computed>"
      ...
  ```

  Fortunately, there's a 1-line workaround using Terraform's built-in state management features:

  **If you are ENABLING archiving:** `terraform state mv module.cloudtrail.aws_s3_bucket.cloudtrail_with_logs_archived module.cloudtrail.aws_s3_bucket.cloudtrail_with_logs_not_archived`

  **If you are DISABLING archiving:** `terraform state mv module.cloudtrail.aws_s3_bucket.cloudtrail_with_logs_not_archived module.cloudtrail.aws_s3_bucket.cloudtrail_with_logs_archived`

  These commands tell Terraform to update the state file, and treat the bucket that Terraform wanted to create as already
  existing. Now you'll get a yellow "modify" output when running `terraform plan` and no destroy/re-create will be needed.




## TODO

- Document [CLI-based validation](http://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-log-file-validation-cli.html)
  of a CloudTrail log file to ensure it hasn't been tampered with.

- Consider enabling [MFA Delete](http://docs.aws.amazon.com/AmazonS3/latest/dev/UsingMFADelete.html) on the S3 Bucket
  that stores the CloudTrail events.

- Implement support for CloudWatch Logs and declaring specific alerts on individual API events.
  - Examples:
    - Alert if CloudTrail Logging is paused.
    - Alert if CloudTrail S3 Bucket is deleted.
    - Alert if any IAM Policy, IAM User, or IAM Role is changed (or would this be too noisy?)
