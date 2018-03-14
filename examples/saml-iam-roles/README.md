**Note**: This public repo contains the documentation for the private GitHub repo <https://github.com/gruntwork-io/module-security>.
We publish the documentation publicly so it turns up in online searches, but to see the source code, you must be a Gruntwork customer.
If you're already a Gruntwork customer, the original source for this file is at: <https://github.com/gruntwork-io/module-security/blob/master/examples/saml-iam-roles/README.md>.
If you're not a customer, contact us at <info@gruntwork.io> or <http://www.gruntwork.io> for info on how to get access!

# SAML IAM roles example

This is an example of how to use the [saml-iam-roles module](/modules/saml-iam-roles) to create IAM
roles that users authenticated by a SAML Identity Provider (IdP) can assume to get access to this AWS account.

## Quick start

To try these templates out you must have Terraform installed:

1. Open `vars.tf`, specify the environment variables mentioned at the top of the file, and fill in any variables that 
   don't have a default.
1. Download SAML 2.0 Metadata from your IdP and copy its contents to [saml-metadata.xml](./saml-metadata.xml). 
1. Run `terraform apply` to create the IAM Roles. 
1. This module will output the ARNs of these IAM roles
