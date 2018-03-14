**Note**: This public repo contains the documentation for the private GitHub repo <https://github.com/gruntwork-io/module-security>.
We publish the documentation publicly so it turns up in online searches, but to see the source code, you must be a Gruntwork customer.
If you're already a Gruntwork customer, the original source for this file is at: <https://github.com/gruntwork-io/module-security/blob/master/modules/saml-iam-roles/README.md>.
If you're not a customer, contact us at <info@gruntwork.io> or <http://www.gruntwork.io> for info on how to get access!

# A best-practices set of IAM roles for SAML access

This module can be used to allow users authenticated via external Security Assertion Markup Language (SAML) identity 
providers such as Google, Amazon SSO, Microsoft Active Directory Federation Services (ADFS), Okta, and OneLogin to access 
your AWS accounts ([saml-access](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_providers_enable-console-saml.html)). 
This allows you to define each environment (mgmt, stage, prod, etc) in a separate AWS account and to use SAML to assume
different roles in each account.

If you're not familiar with IAM concepts, start with the [Background Information](#background-information) section as a 
way to familiarize yourself with the terminology.

## How do you use this module?

To set up SAML access to AWS, you need to:

1. [Create an Identity Provider in each account](#create-identity-provider)
1. [Create IAM roles in each of your accounts](#create-iam-roles)
1. [Create permissions to assume the IAM roles in other accounts](#create-permissions-to-assume-the-iam-roles-in-other-accounts)

This module takes care of [creating IAM roles](#create-iam-roles) and [creating the appropriate permissions](#create-permissions-to-assume-the-iam-roles-in-other-accounts) 

### Create Identity Provider

If you want to allow users of a SAML Identity Provider (IdP) to access your AWS accounts, you will first need to create
a SAML Identity Provider within IAM. Check out the [saml-iam-roles 
example](/examples/saml-iam-roles) for a working sample code of how to use this module.

### Create IAM roles in each account

If you want to allow users from SAML IdPs to access your AWS accounts, use this module in each AWS account to create IAM 
roles that specify which services those users may access. Check out the [saml-iam-roles 
example](/examples/saml-iam-roles) for a working sample code of how to use this module.

### Create permissions to assume the IAM roles in other accounts

Finally, this module will also grant access to users of each SAML provider listed in the various
`allow_*_access_from_saml_provider_arns` variables to assume the corresponding role. Check out the [saml-iam-roles 
example](/examples/saml-iam-roles) for a working sample code of how to use this module.

## Resources Created

This module creates the following IAM roles (all optional):

* **allow-read-only-access-from-saml**: Users authenticated by the SAML providers in
 `var.allow_read_only_access_from_saml_provider_arns` will get read-only access to all services in this account.

* **allow-billing-access-from-saml**: Users authenticated by the SAML providers in
  `var.allow_billing_access_from_saml_provider_arns` will get full (read and write) access to the billing details for 
  this account.

* **allow-ssh-iam-access-from-saml**: Users authenticated by the SAML providers in
  `var.allow_ssh_iam_access_from_saml_provider_arns` will get read access to IAM Groups and public SSH keys. This is
  useful to allow [ssh-iam](/modules/ssh-iam) running on EC2 Instances in other AWS accounts to validate SSH 
  connections against IAM users defined in this AWS account.

* **allow-dev-access-from-saml**:Users authenticated by the SAML providers in
  `var.allow_dev_access_from_saml_provider_arns` will get full (read and write) access to the services in this account 
  specified in `var.dev_permitted_services`.

* **allow-full-access-from-saml**: Users authenticated by the SAML providers in
  `var.allow_full_access_from_saml_provider_arns` will get full (read and write) access to all services in this account.
  
* **allow-auto-deploy-access-from-saml**: Users authenticated by the SAML providers in
  `var.allow_read_only_access_from_saml_provider_arns` will get automated deployment access to all services in this 
  account with the permissions specified in `var.auto_deploy_permissions`. The main use case is to allow a CI server 
  (e.g. Jenkins) in another AWS account to do automated deployments in this AWS account.


## How to switch between accounts


TODO: Provide additional documentation around `gruntsaml` and AWS Console SAML integration