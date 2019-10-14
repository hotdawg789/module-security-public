**Note**: This public repo contains the documentation for the private GitHub repo <https://github.com/gruntwork-io/module-security>.
We publish the documentation publicly so it turns up in online searches, but to see the source code, you must be a Gruntwork customer.
If you're already a Gruntwork customer, the original source for this file is at: <https://github.com/gruntwork-io/module-security/blob/master/examples/custom-iam-entity/README.md>.
If you're not a customer, contact us at <info@gruntwork.io> or <http://www.gruntwork.io> for info on how to get access!

# Custom IAM Entity Example

This is an example of how to use the [custom-iam-entity module](/modules/custom-iam-entity) to create an IAM group or role with attached IAM policies.

Use the boolean variables `should_create_iam_group` and `should_create_iam_role` variables to specify whether to create a group and/or a role, respectively. Name the group and/or role  with the `iam_group_name` and `iam_role_name` variables. If creating an IAM role, you must also specify a list of ARNs that are allowed to assume the role in `assume_role_arns`.

This example will create an IAM group and/or role with several example IAM policies attached. You can attach policies in any combination of the following variables:

- `iam_policy_arns`: Attach policies by providing a list of policy ARNs
- `iam_customer_managed_policy_names`: Attach policies by providing a list of customer-managed policy names
- `iam_aws_managed_policy_names`: Attach policies by providing a list of AWS-managed policy names

All of these variables are optional.

If you provide customer-managed policy names, the module will look up the current AWS account ID and use it to generate the policy ARN for attachment.

    arn:aws:iam::0123456789012:policy/MyCustomPolicy

If you provide AWS-managed policy names, the module will use the AWS formatted policy names for attachment. For example:

    arn:aws:iam::aws:policy/AmazonEC2ReadOnlyAccess

Any policies you provide must already exist or Terraform will throw an error.

## Quick start

To try these templates out you must have Terraform installed:

1. Open `vars.tf` and fill in any variables that don't have a default.
1. Review the policies in `vars.tf` and decide which policies you would like to include for your group.
1. Lean on the example policy in `main.tf` to generate your own managed policies to attach to the group.
1. Run `terraform init`
1. Run `terraform apply` to create the IAM Groups.
1. Now log into the AWS Web Console and assign some IAM Users to IAM Groups.
