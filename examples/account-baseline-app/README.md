**Note**: This public repo contains the documentation for the private GitHub repo <https://github.com/gruntwork-io/module-security>.
We publish the documentation publicly so it turns up in online searches, but to see the source code, you must be a Gruntwork customer.
If you're already a Gruntwork customer, the original source for this file is at: <https://github.com/gruntwork-io/module-security/blob/master/examples/account-baseline-app/README.md>.
If you're not a customer, contact us at <info@gruntwork.io> or <http://www.gruntwork.io> for info on how to get access!

# AWS App Account Baseline Example

This is an example of how to use the [account-baseline-app](/modules/account-baseline-app) to establish security baseline 
for AWS Landing Zone for configuring the app accounts (e.g., dev, stage, prod) of an AWS Organization - including setting up AWS Config, AWS CloudTrail, 
Amazon Guard Duty, IAM users, IAM groups, IAM password policy, and more.

## Quick start

To try these templates out you must have Terraform installed:

1. Open `variables.tf` and fill in any variables that don't have a default.
1. Run `terraform init` to instruct Terraform to perform initialization steps.
1. Run `terraform plan` to confirm that Terraform will create what looks like a reasonable set of resources.
1. Run `terraform apply`.
1. Now log into the AWS Web Console, and locate your new artifacts in IAM, Config, GuardDuty, etc.
