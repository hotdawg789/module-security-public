**Note**: This public repo contains the documentation for the private GitHub repo <https://github.com/gruntwork-io/module-security>.
We publish the documentation publicly so it turns up in online searches, but to see the source code, you must be a Gruntwork customer.
If you're already a Gruntwork customer, the original source for this file is at: <https://github.com/gruntwork-io/module-security/blob/master/modules/fail2ban/README.md>.
If you're not a customer, contact us at <info@gruntwork.io> or <http://www.gruntwork.io> for info on how to get access!

# Fail2Ban Module

This module can configure a Linux server to automatically ban malicious ip addresses from connecting to the server
via SSH. This module currently supports Ubuntu, Amazon Linux, Amazon Linux 2, and CentOS (using
[fail2ban](https://www.fail2ban.org)).

The module also optionally creates CloudWatch Metrics to track the number of Banned and Unbanned IP Addresses per AWS
Instance.

## How do you use this module?

#### Example

See the [fail2ban example](/examples/fail2ban) for an example of how to use this module.

#### Installation

To use this module, you just need to:

1. Install [bash-commons](https://github.com/gruntwork-io/bash-commons) on your servers.
1. Install the `fail2ban` module on your servers.

The best way to do that is to use the [Gruntwork Installer](https://github.com/gruntwork-io/gruntwork-installer) in a
[Packer](https://www.packer.io/) template (make sure to replace `<BASH_COMMONS_VERSION>` and
`<MODULE_SECURITY_VERSION>` below with the latest versions from the [bash-commons releases
page](https://github.com/gruntwork-io/bash-commons/releases) and [module-security releases
page](https://github.com/gruntwork-io/module-security-public/releases), respectively):

```
gruntwork-install --module-name bash-commons --tag <BASH_COMMONS_VERSION> --repo https://github.com/gruntwork-io/bash-commons
gruntwork-install --module-name fail2ban --tag <MODULE_SECURITY_VERSION> --repo https://github.com/gruntwork-io/module-security
```

#### Compatibility

This module is known to work on **Ubuntu** and **Amazon Linux 2**. This does not work with CentOS 7 and Amazon Linux 1.

See https://github.com/gruntwork-io/module-security/issues/89 for more details.

#### Configuration Options

You can configure several options to control the behavior of fail2ban. If you're using gruntwork-install, you'll need to
use the --module-param option, such as gruntwork-install --module fail2ban --module-param ban-time=3600).

|Option|Description|Required|Default|
|---|---|---|---|
|--logging-level|The logging level for fail2ban. 1=ERROR, 2=WARN, 3=INFO, 4=DEBUG.|Optional|3=INFO|
|--target|The target for fail2ban. STDOUT, STDERR, SYSLOG, /path/to/file. |Optional|SYSLOG|
|--ignore-ip|The space delimited list of ip addresses or CIDR blocks for fail2ban to ignore.|Optional|127.0.0.1/8|
|--ban-time|The amount of time in seconds a malicious ip address will be banned for.|Optional|86400|
|--find-time|The time window in seconds to look at for failures|Optional|600|
|--max-retry|The number of failures (eg. password failures) that constitute a bad actor|Optional|5|
|--backend|The method used to determine if a logfile contents have changed. Possible values are: auto, pyinotify, gamin, polling|Optional|auto|
|--ssh-port|The port the ssh daemon being protected is running on|Optional|22 (ssh)|
|--no-cloudwatch-metrics|Flag to disable creation of cloudwatch metrics|Optional||

#### CloudWatch Metrics
By default the script will report the count of the number of IP Addresses Banned and Unbanned to the `BannedIPAddresses`
and `UnbannedIPAddresses` CloudWatch metrics in the `Gruntwork/Fail2Ban` namespace. This namespace can be changed using
the `--cloudwatch-namespace` switch.

CloudWatch Metric reporting can be disabled all together during installation using the `--no-cloudwatch-metrics` switch.

##### Permissions
If the option to install CloudWatch Metrics is selected (default behavior), it is assumed that the EC2 Instance has
permissions to publish metric data to the CloudWatch API. This can be done by attaching a policy to the EC2 Instance's
IAM Role. The permissions necessary are:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "Stmt1493296209000",
            "Effect": "Allow",
            "Action": [
                "cloudwatch:PutMetricData"
            ],
            "Resource": [
                "*"
            ]
        }
    ]
}
```

##### Configure the Fail2Ban CloudWatch Action on your EC2 Instances

In order for the EC2 Instance to send metric data to CloudWatch sucessfully, it needs certain data from the EC2 instance.

When your EC2 Instances are booting up, they should run the `configure-fail2ban-cloudwatch.sh` script, which will configure
fail2ban to send data to CloudWatch. The script supports one command line option:

* `--cloudwatch-namespace`: The namespace used to define the cloudwatch metrics. Default value is 'Gruntwork/Fail2Ban'. Optional.

The best way to run a script during boot is to put it in [User
Data](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/user-data.html#user-data-shell-scripts). Here's an example:

```bash
#!/bin/bash
/etc/user-data/configure-fail2ban-cloudwatch/configure-fail2ban-cloudwatch.sh --cloudwatch-namespace Acme/Fail2Ban
```

##### Default zone for firewalld (Amazon Linux 2 and CentOS)

On Amazon Linux 2 and CentOS, the implementation of `fail2ban` uses `firewalld` to manage the `iptables` for
implementing the firewall. In a default installation of `firewalld` on these operating systems, all inbound ports are
blocked on the interfaces except for SSH access. This is caused by the usage of `public` for the configured default
zone. While this is generally more secure, in practice, this adds a layer of complexity that is typically unnecessary
due to the usage of cloud based firewalls in the form of Security Groups. For example, this requires configuring the
firewalld to allow all the applications you expect to expose on the instance, which may be difficult to track in a
docker cluster like ECS or EKS.

As such, this module updates the default zone of `firewalld` to be `trusted`, which means allow all access by default,
and have rules added that block specific IPs. This setup is similar to how `fail2ban` works on Ubuntu and Amazon Linux
1, which use the more traditional approach of setting `iptables` rules directly.

If you wish to revert to the original behavior of `firewalld`, you can update the default zone back to `public` after
the installation call to `fail2ban`. For example:

```
# Install fail2ban using Gruntwork modules
gruntwork-install --module-name bash-commons --tag <BASH_COMMONS_VERSION> --repo https://github.com/gruntwork-io/bash-commons
gruntwork-install --module-name fail2ban --tag <MODULE_SECURITY_VERSION> --repo https://github.com/gruntwork-io/module-security

# firewalld cannot be configured using `firewall-cmd` unless it is started
sudo systemctl start firewalld

# Update the default zone back to public, as Gruntwork fail2ban installer will set the default zone to be trusted.
sudo firewall-cmd --set-default-zone=public

# We stop firewalld at the end to avoid it interfering with additional installation setups.
sudo systemctl stop firewalld
```

The above script will:

1. Install `fail2ban` using this module.
1. Revert the default zone back to `public` after `fail2ban` installation completes, since it is set to `trusted` during
   the installation step.

#### TODO

* Add support for protocols/services other than ssh
