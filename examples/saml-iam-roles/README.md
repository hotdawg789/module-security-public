**Note**: This public repo contains the documentation for the private GitHub repo <https://github.com/gruntwork-io/module-security>.
We publish the documentation publicly so it turns up in online searches, but to see the source code, you must be a Gruntwork customer.
If you're already a Gruntwork customer, the original source for this file is at: <https://github.com/gruntwork-io/module-security/blob/master/examples/saml-iam-roles/README.md>.
If you're not a customer, contact us at <info@gruntwork.io> or <http://www.gruntwork.io> for info on how to get access!

# SAML IAM roles example

This is an example of how to use the [saml-iam-roles module](/modules/saml-iam-roles) to create IAM
Roles that users authenticated by a SAML Identity Provider (IdP) can assume to get access to this AWS account.

The saml-iam-roles module creates multiple IAM Roles as described in the [Resources Created section](
/modules/saml-iam-roles#resources-created) of the module README. For example, a SAML-authenticated user might assume the
`allow-full-access-from-saml` IAM Role, which grants full access to a given AWS account, or the
`allow-iam-admin-access-from-saml` IAM Role, which grants `iam:*` access (the ability to manage IAM), or the
`allow-read-only-access-from-saml` IAM Role, which grants read-only access to an AWS account. Ultimately, it will be up
to the SAML IdP to assert which users can assume which IAM Roles in which AWS accounts.

## Quick start

To try these templates out you must have Terraform installed:

1. Open `variables.tf`, specify the environment variables mentioned at the top of the file, and fill in any variables that
   don't have a default.
1. Download SAML 2.0 Metadata from your IdP and copy its contents to [saml-metadata.xml](./saml-metadata.xml).
1. Run `terraform init` to instruct Terraform to perform initialization steps.
1. Run `terraform apply` to create the IAM Roles.
1. This module will output the ARNs of these IAM Roles.
