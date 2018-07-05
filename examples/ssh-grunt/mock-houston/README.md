**Note**: This public repo contains the documentation for the private GitHub repo <https://github.com/gruntwork-io/module-security>.
We publish the documentation publicly so it turns up in online searches, but to see the source code, you must be a Gruntwork customer.
If you're already a Gruntwork customer, the original source for this file is at: <https://github.com/gruntwork-io/module-security/blob/master/examples/ssh-grunt/mock-houston/README.md>.
If you're not a customer, contact us at <info@gruntwork.io> or <http://www.gruntwork.io> for info on how to get access!

# Mock Houston

This folder contains Terraform code to deploy a mock version of Gruntwork Houston. It is solely used in our automated
tests for the [ssh-grunt houston example](/examples/ssh-grunt/houston). This allows us to do an end-to-end test where
ssh-grunt talks with an API Gateway endpoint that has IAM authentication, which is hard to test any other way. Do NOT
use this module for anything real!




