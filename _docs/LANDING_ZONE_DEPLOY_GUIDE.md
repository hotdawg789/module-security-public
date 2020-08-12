**Note**: This public repo contains the documentation for the private GitHub repo <https://github.com/gruntwork-io/module-security>.
We publish the documentation publicly so it turns up in online searches, but to see the source code, you must be a Gruntwork customer.
If you're already a Gruntwork customer, the original source for this file is at: <https://github.com/gruntwork-io/module-security/blob/master/_docs/LANDING_ZONE_DEPLOY_GUIDE.md>.
If you're not a customer, contact us at <info@gruntwork.io> or <http://www.gruntwork.io> for info on how to get access!

# Landing Zone Deployment Guide

These are the recommended steps to follow in order to deploy an account structure that consists of a master/root account, security account, logs account and one or more app accounts.

![AWS Landing Zone 1_5 - Account Structure](https://user-images.githubusercontent.com/178939/87698024-34500200-c793-11ea-9d35-4cdf1353db51.png)

## Prerequisites

1. Terraform 0.12.28 or higher
2. Terragrunt v0.23.27 or higher

**Note:** When running the terraform `plan` and `apply` commands, we suggest using the `-parallelism=4` flag (e.g `terraform plan -parallelism-4`) for stable performance as the landing zone modules can peg all of your CPU cores and cause timeout/network connectivity errors.

## Apply the Security Baseline to the root account

The first step is to configure the root of our AWS organization and create all the child accounts. First, we’ll deploy the security baseline into the root account with AWS Config & CloudTrail disabled. This is necessary as we are going to use one of the child accounts, which we'll call logs, to aggregate all AWS Config and CloudTrail data, but that child account doesn't exist yet, so we'll initially disable AWS Config and CloudTrail and re-enable it later.

1. To deploy the `account-baseline-root` example to the root account with AWS Config & CloudTrail disabled, set the following parameters to `false`:

   1. To disable AWS Config set the `var.enable_config` to `false`.
   2. To disable Cloudtrail set the `var.enable_cloudtrail` to `false`.

2) Next, be sure to specify a list of child accounts that also includes a logs account. Here are the input parameters you may specify in a `terragrunt.hcl` or `terraform.tfvars` file:

```hcl
name_prefix    = "bob-root"
aws_region     = "us-east-1"
aws_account_id = "<ROOT_ACCOUNT_ID>"

# This will configure AWS Organizations for the first time in the root account. If you already have AWS Organizations
# configured, either run `import` first to import it into state, or set this to false.
create_organization = true

# Disable AWS Config and CloudTrail on the initial deploy
enable_config     = false
enable_cloudtrail = false

child_accounts = {
  security = {
    email = "aws.orgtest.root+bob-security@gruntwork.io",
    tags  = {
      Account-Tag-Example = "tag-value"
    }
  },
  logs = {
    email = "aws.orgtest.root+bob-logs@gruntwork.io",
  },
  dev = {
    email = "aws.orgtest.root+bob-dev@gruntwork.io",
  },
  stage = {
    email = "aws.orgtest.root+bob-stage@gruntwork.io",
  },
  prod = {
    email = "aws.orgtest.root+bob-prod@gruntwork.io",
  },
  shared-services = {
    email = "aws.orgtest.root+bob-shared-services@gruntwork.io",
  },
}
```

Next, authenticate to your root AWS account, and run `terraform apply` (or `terragrunt apply`) to deploy this module and thereby create all the child accounts.

## Apply the Security Baseline to the logs account

Now that the logs account is created we need to deploy the security baseline into it. Head over to the [AWS console](https://console.aws.amazon.com/), choose the option to login as a root user, enter the email address you associated with the logs account in the previous step, and then reset the [root user password](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_passwords_change-root.html). Once you're able to login to the logs account as the root user, [create Access Keys for the root user](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_root-user.html#id_root-user_manage_add-key) and use them in the following steps to apply the security baseline to the logs account using the `account-baseline-app` example.

**Note:** The `account-baseline-app` example and module can be used interchangeably between app accounts and log accounts as they deploy most of the same resources.

Here are the input parameters you may specify in a `terragrunt.hcl` or `terraform.tfvars` file:

```hcl
name_prefix    = "bob-logs"
aws_region     = "us-east-1"
aws_account_id = "<LOGS_ACCOUNT_ID>"

# Create an S3 bucket that can be used to aggregate CloudTrail logs from all your AWS accounts
cloudtrail_s3_bucket_name             = "bob-logs-cloudtrail-bucket"
cloudtrail_s3_bucket_already_exists   = false
cloudtrail_cloudwatch_logs_group_name = "bob-cloudtrail-logs"

# Create a KMS master key that can be used to encrypt CloudTrail data. Make the logs account the admin/user of this key.
# In practice, this means you can use IAM policies in the logs account to specify who has access to this key.
cloudtrail_kms_key_administrator_iam_arns = ["arn:aws:iam::<LOGS_ACCOUNT_ID>:root"]
cloudtrail_kms_key_user_iam_arns          = ["arn:aws:iam::<LOGS_ACCOUNT_ID>:root"]

# Specify all the IDs of all AWS accounts that should be allowed to write CloudTrail logs to the S3 bucket in this account
cloudtrail_external_aws_account_ids_with_write_access = [
  "<ROOT_ACCOUNT_ID>",
  "<SECURITY_ACCOUNT_ID>",
  "<DEV_ACCOUNT_ID>",
  "<STAGE_ACCOUNT_ID>",
  "<PROD_ACCOUNT_ID>",
  "<SHARED_SERVICES_ACCOUNT_ID>",
]

# Create an S3 bucket that can be used to aggregate AWS Config data from all your AWS accounts
config_s3_bucket_name          = "bob-logs-config-bucket"
config_should_create_s3_bucket = true

# Specify all the IDs of all AWS accounts that should be allowed to write AWS Config data to the S3 bucket in this account
config_linked_accounts = [
  "<ROOT_ACCOUNT_ID>",
  "<SECURITY_ACCOUNT_ID>",
  "<DEV_ACCOUNT_ID>",
  "<STAGE_ACCOUNT_ID>",
  "<PROD_ACCOUNT_ID>",
  "<SHARED_SERVICES_ACCOUNT_ID>",
]

# Allow users in the security account to assume IAM roles in this account
allow_full_access_from_other_account_arns      = ["arn:aws:iam::<SECURITY_ACCOUNT_ID>:root"]
allow_read_only_access_from_other_account_arns = ["arn:aws:iam::<SECURITY_ACCOUNT_ID>:root"]
allow_logs_access_from_other_account_arns      = ["arn:aws:iam::<SECURITY_ACCOUNT_ID>:root"]
```

Next, authenticate to your logs AWS account, and run `terraform apply` (or `terragrunt apply`) to deploy this module.

## Redeploy the root account to publish logs to the logs account

Now that the logs account is created we can enable AWS Config & CloudTrail in the root account for centralized logging. Here are the input parameters you may specify in a `terragrunt.hcl` or `terraform.tfvars` file:

```hcl
name_prefix    = "bob-root"
aws_region     = "us-east-1"
aws_account_id = "<ROOT_ACCOUNT_ID>"

# This will configure AWS Organizations for the first time in the root account. If you already have AWS Organizations
# configured, either run `import` first to import it into state, or set this to false.
create_organization = true

# Enable AWS Config and configure it to send data to the S3 bucket in the logs account
enable_config                  = true
config_s3_bucket_name          = "bob-logs-config-bucket"
config_should_create_s3_bucket = false
config_central_account_id      = "<LOGS_ACCOUNT_ID>"

# Enable CloudTrail and configure it to send logs to the S3 bucket in the logs account
enable_cloudtrail                     = true
cloudtrail_s3_bucket_name             = "bob-logs-cloudtrail-bucket"
cloudtrail_s3_bucket_already_exists   = true
cloudtrail_kms_key_arn                = "arn:aws:kms:us-east-1:<LOGS_ACCOUNT_ID>:key/<ID_OF_KMS_CMK_IN_LOGS_ACCOUNT>"
cloudtrail_cloudwatch_logs_group_name = "bob-cloudtrail-logs"

child_accounts = {
  security = {
    email = "aws.orgtest.root+bob-security@gruntwork.io",
    tags  = {
      Account-Tag-Example = "tag-value"
    }
  },
  logs = {
    email = "aws.orgtest.root+bob-logs@gruntwork.io",
  },
  dev = {
    email = "aws.orgtest.root+bob-dev@gruntwork.io",
  },
  stage = {
    email = "aws.orgtest.root+bob-stage@gruntwork.io",
  },
  prod = {
    email = "aws.orgtest.root+bob-prod@gruntwork.io",
  },
  shared-services = {
    email = "aws.orgtest.root+bob-shared-services@gruntwork.io",
  },
}
```

**Note:** Be sure to replace `config_s3_bucket_name` with the name of the S3 bucket you created in the logs account. Also be sure to set `config_central_account_id` to the logs account id.

Next, authenticate to your logs AWS account, and run `terraform apply` (or `terragrunt apply`) to deploy this module.

## Apply the Security Baseline to the security account

Next, we need to deploy the security baseline to the security account. Reset the root user password then apply the security baseline using the `account-baseline-security` example. Here are the input parameters you may specify in a `terragrunt.hcl` or `terraform.tfvars` file:

```hcl
name_prefix    = "bob-security"
aws_region     = "us-east-1"
aws_account_id = "<SECURITY_ACCOUNT_ID>"

# Enable AWS Config and configure it to send data to the S3 bucket in the logs account
config_s3_bucket_name          = "bob-logs-config-bucket"
config_should_create_s3_bucket = false
config_central_account_id      = "<LOGS_ACCOUNT_ID>"

# Enable CloudTrail and configure it to send logs to the S3 bucket in the logs account
cloudtrail_s3_bucket_name             = "bob-logs-cloudtrail-bucket"
cloudtrail_s3_bucket_already_exists   = true
cloudtrail_kms_key_arn                = "arn:aws:kms:us-east-1:<LOGS_ACCOUNT_ID>:key/<ID_OF_KMS_CMK_IN_LOGS_ACCOUNT>"
cloudtrail_cloudwatch_logs_group_name = "bob-cloudtrail-logs"

# Allow ssh-grunt in all other accounts to look up user SSH keys in this account
allow_ssh_grunt_access_from_other_account_arns = [
  "arn:aws:iam::<DEV_ACCOUNT_ID>:root",
  "arn:aws:iam::<STAGE_ACCOUNT_ID>:root",
  "arn:aws:iam::<PROD_ACCOUNT_ID>:root",
  "arn:aws:iam::<SHARED_SERVICES_ACCOUNT_ID>:root",
]

# Enable the IAM groups you want
should_create_iam_group_full_access    = true
should_create_iam_group_user_self_mgmt = true
should_create_iam_group_read_only      = true

# Create IAM groups that grant access to the other AWS accounts
iam_groups_for_cross_account_access = [
  {
    group_name    = "_account.dev-dev"
    iam_role_arns = ["arn:aws:iam::<DEV_ACCOUNT_ID>:role/allow-dev-access-from-other-accounts"]
  },
  {
    group_name    = "_account.dev-full-access"
    iam_role_arns = ["arn:aws:iam::<DEV_ACCOUNT_ID>:role/allow-full-access-from-other-accounts"]
  },
  {
    group_name    = "_account.dev-openvpn-admins"
    iam_role_arns = ["arn:aws:iam::<DEV_ACCOUNT_ID>:role/openvpn-allow-certificate-revocations-for-external-accounts"]
  },
  {
    group_name    = "_account.dev-openvpn-users"
    iam_role_arns = ["arn:aws:iam::<DEV_ACCOUNT_ID>:role/openvpn-allow-certificate-requests-for-external-accounts"]
  },
  {
    group_name    = "_account.dev-read-only"
    iam_role_arns = ["arn:aws:iam::<DEV_ACCOUNT_ID>:role/allow-read-only-access-from-other-accounts"]
  },

  # ... Repeat the same set of groups for each of stage, prod, logs, and shared services account IDs too!
]

# Create IAM users in the security account
users = {
  alice = {
    groups = ["user-self-mgmt", "ssh-sudo-users", "_account.dev-full-access"]
  }

  bob = {
    path   = "/"
    groups = ["user-self-mgmt", "_account.dev-full-access", "_account.prod-read-only"]
    tags   = {
      foo = "bar"
    }
  }
  carol = {
    groups               = ["user-self-mgmt", "full-access"]
    pgp_key              = "keybase:carol_on_keybase"
    create_login_profile = true
    create_access_keys   = true
  }
}
```

## Apply the Security Baseline to each app account

Next, we need to deploy the security baseline to each app account: that is, dev, stage, and prod. Reset the root user password for each account then apply the security baseline using the `account-baseline-app` example. Here are the input parameters you may specify in a `terragrunt.hcl` or `terraform.tfvars` file:

```hcl
name_prefix    = "bob-dev"
aws_region     = "us-east-1"
aws_account_id = "<APP_ACCOUNT_ID>"

# Enable AWS Config and configure it to send data to the S3 bucket in the logs account
config_s3_bucket_name          = "bob-logs-config-bucket"
config_should_create_s3_bucket = false
config_central_account_id      = "<LOGS_ACCOUNT_ID>"

# Enable CloudTrail and configure it to send logs to the S3 bucket in the logs account
cloudtrail_s3_bucket_name             = "bob-logs-cloudtrail-bucket"
cloudtrail_s3_bucket_already_exists   = true
cloudtrail_kms_key_arn                = "arn:aws:kms:us-east-1:<LOGS_ACCOUNT_ID>:key/<ID_OF_KMS_CMK_IN_LOGS_ACCOUNT>"
cloudtrail_cloudwatch_logs_group_name = "bob-cloudtrail-logs"

# Specify the services the dev IAM role will have access to
dev_permitted_services = ["ec2", "s3", "rds", "dynamodb", "elasticache", "eks", "ecs"]

# Specify the services the auto-deploy IAM role will have access to
auto_deploy_permissions = ["cloudwatch:*", "logs:*", "dynamodb:*", "ecr:*", "ecs:*", "iam:GetPolicy", "iam:GetPolicyVersion", "iam:ListEntitiesForPolicy", "eks:DescribeCluster", "route53:*", "s3:*", "autoscaling:*", "elasticloadbalancing:*", "iam:GetRole", "iam:GetRolePolicy", "iam:PassRole"]

# Allow users in the security account to access the IAM roles in this account
allow_read_only_access_from_other_account_arns = ["arn:aws:iam::<SECURITY_ACCOUNT_ID>:root"]
allow_billing_access_from_other_account_arns   = ["arn:aws:iam::<SECURITY_ACCOUNT_ID>:root"]
allow_dev_access_from_other_account_arns       = ["arn:aws:iam::<SECURITY_ACCOUNT_ID>:root"]
allow_full_access_from_other_account_arns      = ["arn:aws:iam::<SECURITY_ACCOUNT_ID>:root"]

# Allow a CI server in the shared-services account to assume the auto-deploy IAM role in this account
allow_auto_deploy_from_other_account_arns = ["arn:aws:iam::<SHARED_SERVICES_ACCOUNT_ID>:root"]
```
