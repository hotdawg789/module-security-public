**Note**: This public repo contains the documentation for the private GitHub repo <https://github.com/gruntwork-io/module-security>.
We publish the documentation publicly so it turns up in online searches, but to see the source code, you must be a Gruntwork customer.
If you're already a Gruntwork customer, the original source for this file is at: <https://github.com/gruntwork-io/module-security/blob/master/examples/aws-organizations-config-rules/README.md>.
If you're not a customer, contact us at <info@gruntwork.io> or <http://www.gruntwork.io> for info on how to get access!

# AWS Organization Config Rules Example

This is an example of how to use the [aws-organizations-config-rules module](/modules/aws-organizations-config-rules) to 
enable a set of best practices AWS Config Managed rules on your AWS Organization.

**NOTE:** You will have to apply the changes on your Organization root account.

## Quick start

To try these templates out you must have Terraform installed:

1. Open `variables.tf` and fill in any variables that don't have a default.
1. Run `terraform init` to instruct Terraform to perform initialization steps.
1. Run `terraform plan` to confirm that Terraform will create what looks like a reasonable set of resources.
1. Run `terraform apply` to enable Config rules.
1. Now log into the AWS Web Console, go to the Config console, and within a few minutes resources should start to appear.
