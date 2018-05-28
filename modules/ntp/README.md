**Note**: This public repo contains the documentation for the private GitHub repo <https://github.com/gruntwork-io/module-security>.
We publish the documentation publicly so it turns up in online searches, but to see the source code, you must be a Gruntwork customer.
If you're already a Gruntwork customer, the original source for this file is at: <https://github.com/gruntwork-io/module-security/blob/master/modules/ntp/README.md>.
If you're not a customer, contact us at <info@gruntwork.io> or <http://www.gruntwork.io> for info on how to get access!

# NTP Module

This module installs and configures [NTP](http://www.ntp.org/) on a Linux server. This helps prevent clock drift on the
server; if the clock drifts too far, you'll get all sorts of strange errors, such as AWS API calls failing due to 
invalid timestamps.

This script currently works on:

* Ubuntu
* Amazon Linux and Amazon Linux 2 (currently a no-op, since Amazon Linux already has NTP installed & configured)
* CentOS




## How do you use this module?

To use this module, you just need to:

1. Install [bash-commons](https://github.com/gruntwork-io/bash-commons) on your servers.
1. Install the `ntp` module on your servers.

The best way to do that is to use the [Gruntwork Installer](https://github.com/gruntwork-io/gruntwork-installer) in a
[Packer](https://www.packer.io/) template (make sure to replace `<BASH_COMMONS_VERSION>` and
`<MODULE_SECURITY_VERSION>` below with the latest versions from the [bash-commons releases
page](https://github.com/gruntwork-io/bash-commons/releases) and [module-security releases
page](https://github.com/gruntwork-io/module-security-public/releases), respectively):

```
gruntwork-install --module-name bash-commons --tag <BASH_COMMONS_VERSION> --repo https://github.com/gruntwork-io/bash-commons
gruntwork-install --module-name ntp --tag <MODULE_SECURITY_VERSION> --repo https://github.com/gruntwork-io/module-security
```

See the [ntp example](/examples/ntp) for working sample code.
