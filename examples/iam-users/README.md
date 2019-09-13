**Note**: This public repo contains the documentation for the private GitHub repo <https://github.com/gruntwork-io/module-security>.
We publish the documentation publicly so it turns up in online searches, but to see the source code, you must be a Gruntwork customer.
If you're already a Gruntwork customer, the original source for this file is at: <https://github.com/gruntwork-io/module-security/blob/master/examples/iam-users/README.md>.
If you're not a customer, contact us at <info@gruntwork.io> or <http://www.gruntwork.io> for info on how to get access!

# IAM Users Example

This is an example of how to use the [iam-users module](/modules/iam-users) to create and manage IAM users as code. 

This example will create several IAM users, creating login profiles and access keys for some of them, and adding some 
of them to some IAM groups. 

See the [iam-users module](/modules/iam-users) for additional details.

## Quick start

To try these configurations out you must have Terraform installed:

1. Open `variables.tf` and fill in any variables that don't have a default.
1. Run `terraform init`.
1. Run `terraform apply`.
1. When you're done testing, run `terraform destroy`.
