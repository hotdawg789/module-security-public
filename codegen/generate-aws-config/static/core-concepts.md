**Note**: This public repo contains the documentation for the private GitHub repo <https://github.com/gruntwork-io/module-security>.
We publish the documentation publicly so it turns up in online searches, but to see the source code, you must be a Gruntwork customer.
If you're already a Gruntwork customer, the original source for this file is at: <https://github.com/gruntwork-io/module-security/blob/master/codegen/generate-aws-config/static/core-concepts.md>.
If you're not a customer, contact us at <info@gruntwork.io> or <http://www.gruntwork.io> for info on how to get access!

### Using AWS Config in multiple regions and accounts

Managing AWS Config in all regions of all accounts presents considerable challenges when creating the appropriate configuration resources in the right order as well as when reviewing AWS Config output. Furthermore, it is impractical to set up S3 buckets for each account:region pair and to then review the results. We recommend the following strategy for handling Config across multiple AWS accounts:

1. When using this module, begin with the account that should act as the "central" account for collecting AWS Config output from child accounts. In the Gruntwork reference architecture this would be the "security" account.
1. Use the `global_recorder_region` variable to designate one region as the recorder for global resource types.
1. For the central account, set `should_create_s3_bucket=true` and provide a name with the `s3_bucket_name` variable. This will create an S3 bucket with encryption enabled, public access disabled, a lifecycle policy with expiration dates for the Config logs stored in that bucket, and an access policy permitting access to all the accounts named in the `linked_accounts` variable. If you wish to use SNS for delivery notifications, provide a name in `sns_topic_name` and set `should_create_sns_topic=true`.
1. Run this module on all child accounts, setting the `global_recorder_region` for one region as mentioned above, and passing the S3 bucket created in the previous step in the `s3_bucket_name` variable. Set `should_create_s3_bucket=false`. You must also set `central_account_id` to the account from the first step above. If you wish to use SNS for delivery notifications, provide a name in `sns_topic_name` and set `should_create_sns_topic=false` to use the SNS topic in the central account.

Once complete, the following configuration will be established:

* In each account, an IAM role will exist with suitable permissions for AWS Config
* In each region of each account, Config will be enabled. It will be configured to send objects to an S3 bucket located in the `global_recorder_region` of the central account, and, if SNS is enabled, to deliver SNS notifications to the topic in the corresponding region of the central account. Additionally, aggregation will be enabled for the `global_recorder_region` of the central account.
* In the `global_recorder_region` of the central account: Config will be enabled with an aggregator, S3 bucket, and an SNS topic.

If you add more accounts, update the `linked_accounts` variable, then rerun terraform in the central account to ensure that permissions are configured for the new account.
