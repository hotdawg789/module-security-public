**Note**: This public repo contains the documentation for the private GitHub repo <https://github.com/gruntwork-io/module-security>.
We publish the documentation publicly so it turns up in online searches, but to see the source code, you must be a Gruntwork customer.
If you're already a Gruntwork customer, the original source for this file is at: <https://github.com/gruntwork-io/module-security/blob/master/examples/aws-config/README.md>.
If you're not a customer, contact us at <info@gruntwork.io> or <http://www.gruntwork.io> for info on how to get access!

# AWS Config Example

This is an example of how to use the [aws-config module](/modules/aws-config) to enable AWS Config on your AWS account.

This example enables Config in a single region by creating an S3 bucket, an SNS topic, an IAM role with the necessary permissions, and the Config resources to put it all together.

## Quick start

To try these templates out you must have Terraform installed:

1. Open `variables.tf` and fill in any variables that don't have a default.
1. Run `terraform init` to instruct Terraform to perform initialization steps.
1. Run `terraform plan` to confirm that Terraform will create what looks like a reasonable set of resources.
1. Run `terraform apply` to enable Config.
1. Now log into the AWS Web Console, go to the Config console, and within a few minutes resources should start to appear.
