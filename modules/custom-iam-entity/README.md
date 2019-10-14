**Note**: This public repo contains the documentation for the private GitHub repo <https://github.com/gruntwork-io/module-security>.
We publish the documentation publicly so it turns up in online searches, but to see the source code, you must be a Gruntwork customer.
If you're already a Gruntwork customer, the original source for this file is at: <https://github.com/gruntwork-io/module-security/blob/master/modules/custom-iam-entity/README.md>.
If you're not a customer, contact us at <info@gruntwork.io> or <http://www.gruntwork.io> for info on how to get access!

# Custom IAM Entity

This Gruntwork Terraform Module creates an IAM group and/or role and attaches a provided set of IAM managed policies to the group. This can be used in conjunction with the [iam-groups](/modules/iam-groups), [cross-account-iam-roles](/modules/cross-account-iam-roles), and [saml-iam-roles](/modules/saml-iam-roles) modules which create a set of groups and roles with smart defaults. Use this module to easily create IAM groups and roles with a defined set of permissions.

### Requirements

- You will need to be authenticated to AWS with an account that has `iam:*` permissions.


### Instructions

Check out the [custom-iam-entity example](../../examples/custom-iam-entity) for a working example.


#### Resources Created


* **IAM group** - (optional) an IAM group with the provided name and attaches each of the requested policies.
* **IAM role** - (optional) an IAM role with the provided name and attaches each of the requested policies.

If neither role nor group are provided, this module does nothing.


#### Resources NOT Created


* **IAM users** - This module does not create any IAM Users, nor assign any existing IAM Users to IAM Groups. You can use the [iam-users module](/modules/iam-users) to create users.
* **IAM policies** - This module only attaches policies by ARN or by name. It does not create any new policies.


## Background Information

For background information on IAM, IAM users, IAM policies, and more, check out the [background information docs in
the iam-policies module](/modules/iam-policies#background-information).
