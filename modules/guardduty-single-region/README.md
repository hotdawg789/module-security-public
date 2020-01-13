**Note**: This public repo contains the documentation for the private GitHub repo <https://github.com/gruntwork-io/module-security>.
We publish the documentation publicly so it turns up in online searches, but to see the source code, you must be a Gruntwork customer.
If you're already a Gruntwork customer, the original source for this file is at: <https://github.com/gruntwork-io/module-security/blob/master/modules/guardduty-single-region/README.md>.
If you're not a customer, contact us at <info@gruntwork.io> or <http://www.gruntwork.io> for info on how to get access!

# GuardDuty Single Region Module

This Gruntwork Terraform module enables AWS GuardDuty within a single region
in your AWS account. The module also optionally creates an SNS topic where
GuardDuty findings can be published to through a CloudWatch Event.

It is important to note, however, that this module is intended as piece of the
GuardDuty Multi Region Module. Best practices recommends enabling AWS GuardDuty
across all regions and there aren't usually use cases where you would want to
enable GuardDuty in a single region.

For more information regarding AWS GuardDuty and our recommended Gruntwork Terraform
module for activating it, please see the [guardduty-multi-region module](../guardduty-multi-region).
