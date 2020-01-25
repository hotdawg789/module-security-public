**Note**: This public repo contains the documentation for the private GitHub repo <https://github.com/gruntwork-io/module-security>.
We publish the documentation publicly so it turns up in online searches, but to see the source code, you must be a Gruntwork customer.
If you're already a Gruntwork customer, the original source for this file is at: <https://github.com/gruntwork-io/module-security/blob/master/modules/guardduty/core-concepts.md>.
If you're not a customer, contact us at <info@gruntwork.io> or <http://www.gruntwork.io> for info on how to get access!

# AWS GuardDuty Core Concepts

## Background

### What is GuardDuty?
Amazon GuardDuty is a threat detection service that continuously monitors for
malicious activity and unauthorized behavior in an AWS account. The service
analyzes events across multiple AWS data sources, such as AWS CloudTrail, Amazon
VPC Flow Logs, and DNS logs, and uses machine learning, anomaly detection, and
integrated threat intelligence to identify and prioritize potential threats.

### Why use GuardDuty?
GuardDuty offers a simple way to improve the security of your AWS account in a
very non-intrusive way. It is fully managed and operates completely independently
from your resources with no impact in performance.

### What is a "Finding"?
A Finding is a detection of a potentially malicious activity. The GuardDuty
Findings can be viewed at the GuardDuty console and can be exported to other
sources such as S3 or an SNS topic. You can read more about Findings at their [AWS
documentation](https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_findings.html),
including more information about all the [types of Findings](https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_finding-types-active.html)
that GuardDuty monitors.

### Where should I enable GuardDuty?
GuardDuty is a regional service, which you can turn on by enabling its detector in
a specific region. All data analyzed is regionally based and
doesn’t cross AWS regional boundaries. You can choose to aggregate the findings
by exporting it to other services such as S3 or SNS topics through CloudWatch.
As best practice, it is however advised to enable GuardDuty across all regions
on your AWS account, even in the regions where you don't actively use. However,
for regions introduced after March 2019, which are opted out by default, it is
best to just [leave the region disabled](https://docs.aws.amazon.com/general/latest/gr/rande-manage.html)
if you don't intend to use it.

It is also advised to enable GuardDuty in all AWS accounts in your organization.
It is possible manage your GuardDuty findings centrally by defining a GuardDuty
master account and adding detectors in other AWS accounts as members. This
multi-account connection is also done regionally and can be better visualized
in the diagram below:

![GuardDuty Multi Account Setup](_docs/multiaccount_guardduty.png)

## Resources created

This module creates the following AWS resources:

- **GuardDuty Detectors**: an enabled GuardDuty detector in every specified region.

If the variable `publish_findings_to_sns` is set to `true`:
- **CloudWatch Event Rules**: a CloudWatch Event Rule that is triggered on sources
coming from AWS GuardDuty to generate a [CloudWatch Event](https://docs.aws.amazon.com/AmazonCloudWatch/latest/events/WhatIsCloudWatchEvents.html)
in every specified region.
- **CloudWatch Event Targets**: A CloudWatch Event Target that publishes the findings
to a SNS Topic in the same region for every specified region.
- **SNS Topics**: An SNS Topic for every specified region where AWS GuardDuty
Findings in that region will be published to.

## Gotchas
- CloudWatch does not publish Findings to SNS instantly, they are published with
a window of 5 minutes which cannot be customized. Subsquent Findings of the same
type are aggregated and sent as a new notification in a customizable timeframe
which defaults to every 6 hours.

## Known Issues

1. As of 0.12.16, Terraform still does not support:
 * [for_each or counts with modules](https://github.com/hashicorp/terraform/issues/17519)
 * [for_each or counts with providers](https://github.com/hashicorp/terraform/issues/19932)
 * [interpolating variables on the provider parameter of a resource](https://github.com/hashicorp/terraform/issues/18682)

The combination of these limitations makes it impossible to natively write Terraform
code that creates the necessary resources across all regions. It is then necessary
to work around this by specifying all the wanted regions and employing hard coded
repetition of all possible regions by creating one AWS provider per region and
one module definition per provider.

1. As of 2 Dec 2019, AWS GuardDuty is not yet one of the [services that you can enable with AWS Organizations Trusted Access](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_integrated-services-list.html). This would be an ideal approach
when available.

1. As of terraform 0.12.17 and terraform provider aws 2.41.0, terraform will output an ugly
error when configuring a provider for a region that has been [opted-out](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/manage-account-payment.html#manage-account-payment-enable-disable-regions).

```
Error: error using credentials to get account ID: error calling sts:GetCallerIdentity: InvalidClientTokenId: The security token included in the request is invalid.
```

It is however possible to work around this, with significant modifications such
as having a fallback region and having additional flags for creating resources
in the affected modules (because of another limitation listed above, modules
don't have counts).

```
provider "aws" {
  alias  = "apeast"
  region = contains("ap-east-1", local.enabled_regions) ? "ap-east-1" : local.enabled_regions[0]
}
```


# More info

https://aws.amazon.com/guardduty/faqs/
