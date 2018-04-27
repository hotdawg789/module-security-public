**Note**: This public repo contains the documentation for the private GitHub repo <https://github.com/gruntwork-io/module-security>.
We publish the documentation publicly so it turns up in online searches, but to see the source code, you must be a Gruntwork customer.
If you're already a Gruntwork customer, the original source for this file is at: <https://github.com/gruntwork-io/module-security/blob/master/examples/ip-lockdown/README.md>.
If you're not a customer, contact us at <info@gruntwork.io> or <http://www.gruntwork.io> for info on how to get access!

# ip-lockdown Module Example

Here you can find a few examples of how to use the [ip-lockdown module](/modules/ip-lockdown) to configure a Linux server to
automatically restrict outgoing connections to all but specified users to specific IPs. The examples demonstrate how you can use a [Packer](https://www.packer.io/) template that creates an Ubuntu AMI as well as a [Docker](https://www.docker.com/) image and installs and configures `ip-lockdown` on them.

## Using the examples

* [Build and run the docker example](/examples/ip-lockdown/local-test)
* [Build and run the full AWS/Terraform example](/examples/ip-lockdown/aws-example)