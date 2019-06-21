**Note**: This public repo contains the documentation for the private GitHub repo <https://github.com/gruntwork-io/module-security>.
We publish the documentation publicly so it turns up in online searches, but to see the source code, you must be a Gruntwork customer.
If you're already a Gruntwork customer, the original source for this file is at: <https://github.com/gruntwork-io/module-security/blob/master/modules/ssm-healthchecks-iam-permissions/README.md>.
If you're not a customer, contact us at <info@gruntwork.io> or <http://www.gruntwork.io> for info on how to get access!

# SSM Healthchecks IAM Permissions

This modules adds the necessary IAM policies to an IAM role so that the [AWS Systems Manager](https://docs.aws.amazon.com/systems-manager/latest/userguide/what-is-systems-manager.html) [agent](https://docs.aws.amazon.com/systems-manager/latest/userguide/ssm-agent.html) gets necessary permissions in order to do automated health checks.

## Motivation 

From: https://docs.aws.amazon.com/systems-manager/latest/userguide/ssm-agent.html

> SSM Agent is installed, by default, on the following Amazon EC2 Amazon Machine Images (AMIs):
>
> - Windows Server (all SKUs)
> - Amazon Linux
> - Amazon Linux 2
> - Ubuntu Server 16.04
> - Ubuntu Server 18.04

We recommend using this module with just about every single EC2 Instance and Auto Scaling Group you launch, or you'll end up with confusing SSM errors in your logs (`syslog`).

## How do you use this module?

This module contains an IAM policy, so you pass in an IAM Role ID (e.g., such as one from an EC2 Instance), and the module will attach the appropriate permissions to it.

You use this module like any other [Terraform module](https://www.terraform.io/docs/modules/usage.html):

```hcl
# ----------------------------------------------------------------------------------------------------------------------
# CREATE IAM POLICIES THAT ALLOW AWS SSM HEALTHCHECKS TO FUNCTION
# ----------------------------------------------------------------------------------------------------------------------

module "aws_ssm_permissions" {
  source = "git::git@github.com:gruntwork-io/module-security.git//modules/ssm-healthchecks-iam-permissions?ref=v[LATEST_VERSION_GOES_HERE]"

  iam_role_id = "${aws_iam_role.example.id}"
  name_prefix = "example"
}
```

