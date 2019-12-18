**Note**: This public repo contains the documentation for the private GitHub repo <https://github.com/gruntwork-io/module-security>.
We publish the documentation publicly so it turns up in online searches, but to see the source code, you must be a Gruntwork customer.
If you're already a Gruntwork customer, the original source for this file is at: <https://github.com/gruntwork-io/module-security/blob/master/modules/aws-organizations/core-concepts.md>.
If you're not a customer, contact us at <info@gruntwork.io> or <http://www.gruntwork.io> for info on how to get access!

# AWS Organizations Core Concepts

## Background

### What is AWS Organizations?
AWS Organizations is an account management service that enables you to consolidate multiple AWS accounts into an organization that you create and centrally manage.

### What is a Root Account?
Root account is the parent container for all the accounts for your organization. If you apply a policy to the root, it applies to all organizational units (OUs) and accounts in the organization.

### What are Organization Accounts?
As an administrator of an organization, you can create accounts in your organization and invite existing accounts to join the organization. AWS Organizations includes account management and consolidated billing capabilities that enable you to better meet the budgetary, security, and compliance needs of your business. 

### What is an Organization unit (OU)
A container for accounts within a root. An OU also can contain other OUs, enabling you to create a hierarchy that resembles 
an upside-down tree, with a root at the top and branches of OUs that reach down, ending in accounts that are the leaves 
of the tree. When you attach a policy to one of the nodes in the hierarchy, it flows down and affects all the branches 
(OUs) and leaves (accounts) beneath it. An OU can have exactly one parent, and currently each account can be a member of 
exactly one OU.

This module enables creating a new AWS Organization (or using an existing) and managing Accounts within the Organization, 
but does not manage or create any Organization units. You can create these yourself using 
https://www.terraform.io/docs/providers/aws/r/organizations_organizational_unit.html and then pass in their IDs when 
creating child accounts using this module.

## What resources does this module create?

This module creates the following resources:

- **aws_organizations_organization**: (Optional) An entity that you create to consolidate your AWS accounts.
- **aws_organizations_account**: Zero or more AWS Accounts that contain your AWS resources.

The module does not create and manage [Organization 
Units](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_ous.html) 

## Day-to-day operations

### How do I provision new accounts?
You can provision new accounts by adding entries to `child_accounts` input variable and applying the changes. 
See [variables.tf](variables.tf) for the input format.  


### How do I remove accounts?
Removing entries from `child_accounts` will only remove an AWS account from an organization. Terraform will not close the account. 
The member account must be prepared to be a standalone account beforehand. 
See the [AWS Organizations documentation](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_accounts_remove.html) for more information.
