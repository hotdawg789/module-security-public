**Note**: This public repo contains the documentation for the private GitHub repo <https://github.com/gruntwork-io/module-security>.
We publish the documentation publicly so it turns up in online searches, but to see the source code, you must be a Gruntwork customer.
If you're already a Gruntwork customer, the original source for this file is at: <https://github.com/gruntwork-io/module-security/blob/master/modules/aws-organizations-config-rules/core-concepts.md>.
If you're not a customer, contact us at <info@gruntwork.io> or <http://www.gruntwork.io> for info on how to get access!

# AWS Organizations Config Rules Core Concepts

## Background

### What are Managed Config Rules?
AWS Config provides [AWS managed rules](https://docs.aws.amazon.com/config/latest/developerguide/evaluate-config_use-managed-rules.html), 
which are predefined, customizable rules that AWS Config uses to evaluate whether your AWS resources comply with common best practices.

### Why Organization Level Config Rules?

By configuring and applying the rules at master account in AWS Organizations, you enforce governance by ensuring that 
the underlying AWS Config rules are not modifiable by your organization’s member accounts. Individual accounts can still
configure additional Config rules, but cannot remove or disable those configured at master account.

## What resources does this module create?

This module creates the following AWS Config Managed Rules:

- **[encrypted-volumes](https://docs.aws.amazon.com/config/latest/developerguide/encrypted-volumes.html):** Checks whether the EBS volumes that are in an attached state are encrypted.
- **[rds-storage-encrypted](https://docs.aws.amazon.com/config/latest/developerguide/rds-storage-encrypted.html):** Checks whether storage encryption is enabled for your RDS DB instances.
- **[iam-password-policy](https://docs.aws.amazon.com/config/latest/developerguide/iam-password-policy.html):** Checks whether the account password policy for IAM users meets the specified requirements.
- **[s3-bucket-public-read-prohibited](https://docs.aws.amazon.com/config/latest/developerguide/s3-bucket-public-read-prohibited.html):** Checks that your Amazon S3 buckets do not allow public read access.
- **[s3-bucket-public-write-prohibited](https://docs.aws.amazon.com/config/latest/developerguide/s3-bucket-public-write-prohibited.html):** Checks that your Amazon S3 buckets do not allow public write access.
- **[vpc-sg-open-only-to-authorized-ports](https://docs.aws.amazon.com/config/latest/developerguide/vpc-sg-open-only-to-authorized-ports.html):** Checks whether the security group with 0.0.0.0/0 of any Amazon Virtual Private Cloud (Amazon VPC) allows only specific inbound TCP or UDP traffic.
- **[root-account-mfa-enabled](https://docs.aws.amazon.com/config/latest/developerguide/root-account-mfa-enabled.html):** Checks whether users of your AWS account require a multi-factor authentication (MFA) device to sign in with root credentials.


## Day-to-day operations

### How do I configure the rules?
By default, the module will enable all rules defined in [What resources does this module create](#what-resources-does-this-module-create) section,
but you can choose to disable one or more rules using the `enable_*` input variables and configure the attributes
for certain rules. 

Some of the rules have additional attributes. The module has default values for those, but you can adjust the values with
the input parameters defined in [variables.tf](variables.tf). The following snippet is an example of adjusting attributes 
for password policy and disabling one of the default rules:

```hcl
module "organizations_config_rules" {
  source = "../../modules/aws-organizations-config-rules"

  # Custom password policy configuration   
  iam_password_policy_minimum_password_length = 20
  iam_password_policy_require_symbols         = false

  # Turn off insecure security group rule   
  enable_insecure_sg_rules = false
}
```

### How do I add additional rules?
In addition to the predefined set of rules, you can add additional rules. The following snippet is an example of configuring
ACM Certificate expriration check with a custom input parameter:

```hcl
module "organizations_config_rules" {
  source = "../../modules/aws-organizations-config-rules"

  # Configure additional managed rules   
  additional_rules = {
    acm-certificate-expiration-check = {
      description      = "Checks whether ACM Certificates in your account are marked for expiration within the specified number of days.",
      identifier       = "ACM_CERTIFICATE_EXPIRATION_CHECK",
      trigger_type     = "PERIODIC"
      input_parameters = { "daysToExpiration": "14"},
    }
  }
}
```

For a full list of available managed rules and their configuration, see https://docs.aws.amazon.com/config/latest/developerguide/managed-rules-by-aws-config.html

### How do I exclude specific accounts?
By default, the config rules trickle down to all Organization accounts. You can, however, optionally exclude specific accounts
using the `excluded_accounts` input variable. Note that you cannot exclude all accounts in your Organization. If you do that,
the AWS API will error with: `AllAccountsExcluded: For this OrganizationConfigResource, all member accounts in this 
organization are part of an exclusion list. Modify your exclusion list and try again.`.
