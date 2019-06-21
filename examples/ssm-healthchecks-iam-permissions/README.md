**Note**: This public repo contains the documentation for the private GitHub repo <https://github.com/gruntwork-io/module-security>.
We publish the documentation publicly so it turns up in online searches, but to see the source code, you must be a Gruntwork customer.
If you're already a Gruntwork customer, the original source for this file is at: <https://github.com/gruntwork-io/module-security/blob/master/examples/ssm-healthchecks-iam-permissions/README.md>.
If you're not a customer, contact us at <info@gruntwork.io> or <http://www.gruntwork.io> for info on how to get access!

# IAM-SSM-healthcheck permissions

This is an example of how to use the [ssm-healthcheck-iam-permissions module](/modules/ssm-healthchecks-iam-permissions)
to grant permissions to EC2 instances so that the AWS SSM agent on the instance is able to properly query SSM to do
healthchecks.
