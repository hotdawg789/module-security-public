**Note**: This public repo contains the documentation for the private GitHub repo <https://github.com/gruntwork-io/module-security>.
We publish the documentation publicly so it turns up in online searches, but to see the source code, you must be a Gruntwork customer.
If you're already a Gruntwork customer, the original source for this file is at: <https://github.com/gruntwork-io/module-security/blob/master/examples/aws-organizations/README.md>.
If you're not a customer, contact us at <info@gruntwork.io> or <http://www.gruntwork.io> for info on how to get access!

# AWS Organizations Example

This is an example of how to use the [aws-organizations module](/modules/aws-organizations) to create an Organization and
child accounts.

This example uses an existing organization and adds two child accounts to it.

**NOTE:** Destroying the example or removing entries from `child_accounts` will only remove an AWS account from an organization. 
Terraform **will not close the account**. The member account must be prepared to be a standalone account beforehand. 
See the [AWS Organizations documentation](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_accounts_remove.html) 
for more information.

## Quick start

To try these templates out you must have Terraform installed:

1. Open `variables.tf` and fill in any variables that don't have a default.
1. Run `terraform init` to instruct Terraform to perform initialization steps.
1. Run `terraform plan` to confirm that Terraform will create what looks like a reasonable set of resources.
1. Run `terraform apply`.
1. Now log into the AWS Web Console, go to the Organizations console, and locate your new child accounts.
