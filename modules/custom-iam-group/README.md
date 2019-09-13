**Note**: This public repo contains the documentation for the private GitHub repo <https://github.com/gruntwork-io/module-security>.
We publish the documentation publicly so it turns up in online searches, but to see the source code, you must be a Gruntwork customer.
If you're already a Gruntwork customer, the original source for this file is at: <https://github.com/gruntwork-io/module-security/blob/master/modules/custom-iam-group/README.md>.
If you're not a customer, contact us at <info@gruntwork.io> or <http://www.gruntwork.io> for info on how to get access!

# Custom IAM Group

This Gruntwork Terraform Module creates an IAM group and attaches a provided set of IAM managed policies to the group. This can be used in addition to the [iam-groups module](/modules/iam-groups) that creates best-practices set of groups and permissions. You can use this to easily create IAM groups with a defined set of permissions.


## How do you use this module?


### Requirements

- You will need to be authenticated to AWS with an account that has `iam:*` permissions.


### Instructions

Check out the [custom-iam-group example](../../examples/custom-iam-group) for a working example.


## Resources Created


### IAM Groups

This module creates an IAM group with the provided name and attaches each of the requested policies.



## Resources NOT Created


### IAM Users

This module does not create any IAM Users, nor assign any existing IAM Users to IAM Groups. You can use the [iam-users module](/modules/iam-users) to create users.


### IAM Roles

This module does not create any IAM Roles. Those should be created with Terraform, but in separate templates in the
context of your specific needs, not as a generic set of roles.



## Background Information

For background information on IAM, IAM users, IAM policies, and more, check out the [background information docs in
the iam-policies module](/modules/iam-policies#background-information).
