**Note**: This public repo contains the documentation for the private GitHub repo <https://github.com/gruntwork-io/module-security>.
We publish the documentation publicly so it turns up in online searches, but to see the source code, you must be a Gruntwork customer.
If you're already a Gruntwork customer, the original source for this file is at: <https://github.com/gruntwork-io/module-security/blob/master/examples/ip-lockdown/local-test/README.md>.
If you're not a customer, contact us at <info@gruntwork.io> or <http://www.gruntwork.io> for info on how to get access!

# ip-lockdown Docker Example

In this example we will use our [Packer](https://www.packer.io/) template to create a [Docker](https://www.docker.com/) image running Ubuntu with [ip-lockdown](/modules/ip-lockdown) installed and configured.

## Quick start

To build the AMIs:

1. Install [Packer](https://www.packer.io/)
1. Set your [GitHub access token](https://help.github.com/articles/creating-an-access-token-for-command-line-use/) as
   the environment variable `GITHUB_OAUTH_TOKEN`.
1. Run `packer build ../ip-lockdown-sample.json`

To start docker:

1. Run `docker-compose run ip-lockdown /bin/bash`
1. You can now test `ip-lockdown`:
```bash
    root@b5fc026fb5c1:/# ip-lockdown --help

    Usage: ip-restrict <IP> [<USER>...]

    This script will lock down IP so that only the OS users [<USER>...] can access them.

    Examples:

    ip-restrict.sh 169.254.169.254 root foo bar baz
  ```