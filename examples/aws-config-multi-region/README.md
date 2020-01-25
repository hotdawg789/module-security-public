**Note**: This public repo contains the documentation for the private GitHub repo <https://github.com/gruntwork-io/module-security>.
We publish the documentation publicly so it turns up in online searches, but to see the source code, you must be a Gruntwork customer.
If you're already a Gruntwork customer, the original source for this file is at: <https://github.com/gruntwork-io/module-security/blob/master/examples/aws-config-multi-region/README.md>.
If you're not a customer, contact us at <info@gruntwork.io> or <http://www.gruntwork.io> for info on how to get access!

# AWS Config Multi Region Example

This is an example of how to use the [aws-config-multi-region module](/modules/aws-config-multi-region) to enable AWS
Config across all regions on your AWS Account.

See the README at [aws-config-multi-region module](/modules/aws-config-multi-region) for additional details.

## Quick start

This example contains two flavors:

- [Terraform](#terraform)
- [Terragrunt](#terragrunt)

### Terraform

1. Open `variables.tf` and fill in any variables that don't have a default.
1. Run `terraform init` to instruct Terraform to perform initialization steps.
1. Run `terraform plan` to confirm that Terraform will create what looks like a reasonable set of resources.
1. Run `terraform apply` to enable Config.
1. Now log into the AWS Web Console, go to the Config console, and within a few minutes resources should start to appear.

### Terragrunt

1. Open `terragrunt.hcl` and update any variables to fit your environment.
1. Run `terragrunt plan` to confirm that Terraform will create what looks like a reasonable set of resources.
1. Run `terragrunt apply` to enable Config.
1. Now log into the AWS Web Console, go to the Config console, and within a few minutes resources should start to appear.
