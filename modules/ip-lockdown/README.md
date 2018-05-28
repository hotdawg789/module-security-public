**Note**: This public repo contains the documentation for the private GitHub repo <https://github.com/gruntwork-io/module-security>.
We publish the documentation publicly so it turns up in online searches, but to see the source code, you must be a Gruntwork customer.
If you're already a Gruntwork customer, the original source for this file is at: <https://github.com/gruntwork-io/module-security/blob/master/modules/ip-lockdown/README.md>.
If you're not a customer, contact us at <info@gruntwork.io> or <http://www.gruntwork.io> for info on how to get access!

# ip-lockdown Module

This module can lock down specified outgoing ip addresses on a Linux server such that only specific OS users can access them. 
The main motivation for locking down EC2 metadata is as follows:

1. EC2 metadata gives you the credentials you need to assume any IAM role associated with the EC2 instance, and thereby, get all the permissions available in that IAM role.
1. Locking down the metadata to, for example, only the root user, makes sure that if a hacker breaks into your server with a privileged user, they cannot get the full power of the IAM role.

This module has been tested specifically with Ubuntu, but will probably work with any Debian distribution that uses [iptables](http://ipset.netfilter.org/iptables.man.html).

#### Example
In the example below we restrict access to [ec2-instance-metadata endpoint](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-metadata.html) to the users `foo`, `bar` and `root`. All other users on the instance will be blocked from access.

`./ip-lockdown 169.254.169.254 foo bar root`

Normally users make a `curl` call to get metadata like the AWS region or credentials associated with this EC2 Instance's IAM Role. Following the invocation of ip-lockdown, only users foo, bar, and root can query that data.

The complete example of using terraform to deploy a generated AMI into your AWS account and automatically invoke `ip-lockdown` from the `User Data` is also available in the [examples](/examples/ip-lockdown/aws-example) folder.

#### Installation

To use this module, you just need to:

1. Install [bash-commons](https://github.com/gruntwork-io/bash-commons) on your servers.
1. Install the `ip-lockdown` script on your servers.

The best way to do that is to use the [Gruntwork Installer](https://github.com/gruntwork-io/gruntwork-installer) in a
[Packer](https://www.packer.io/) template (make sure to replace `<BASH_COMMONS_VERSION>` and
`<MODULE_SECURITY_VERSION>` below with the latest versions from the [bash-commons releases
page](https://github.com/gruntwork-io/bash-commons/releases) and [module-security releases
page](https://github.com/gruntwork-io/module-security-public/releases), respectively):

```
gruntwork-install --module-name bash-commons --tag <BASH_COMMONS_VERSION> --repo https://github.com/gruntwork-io/bash-commons
gruntwork-install --module-name ip-lockdown --tag <MODULE_SECURITY_VERSION> --repo https://github.com/gruntwork-io/module-security
```

|Option|Description|Required|Example|
|---|---|---|---|
|IP|IP address that will be locked down (outgoing access will be disabled) for all _but_ the users specified in subequent `[<USER> ... ]]` arguments|Required|169.254.169.254|
|USER|Space separated whitelist of users who will be allowed outgoing access to specified ip address|Optional|root (or any other OS user name)|

## How do you use this module?

This script will insert the necessary rules to achieve the proper end result of only allowing the specified users to access the locked ips.  

* It will **NOT** modify your existing rules
* It will **NOT** add multiple identical rules if you keep running the script (it is idempotent)
* It will automatically add rules at the correct rule index if you chose to add more users later or to add more IP addresses later.

#### General iptables overview ####
The `ip-lockdown` script uses [iptables](http://ipset.netfilter.org/iptables.man.html) under the hood. The iptables application works by defining rules that can then be applied to each outgoing packet. The rules are applied in order (`sudo iptables -L line-num` to see your current rules as well as their _rule-indicies_).

In order to block access to a specific IP address for a specific user you need 2 rules.
1. A rule to ALLOW packets going to `YYY` owned by user `foo`
2. A rule to BLOCK packets going to `YYY`

The ordering of the rules is important as [iptables](http://ipset.netfilter.org/iptables.man.html) will go through the rules list until it finds a matching rule. As soon as a matching rule is found none of the subsequent rules are evaluated.

In the above example, reversing those two rules would result in all access to `YYY` blocked even though there is a subsequent rule to allow `foo` to access.

#### Why not use groups? ####

In an ideal scenario, rather than adding one allow rule per user, we would just create a new `canAccessIP` group, and add our required users to that group.  Then we would just need two iptables rules to manage access.  

Unfortunately iptables suffers from the limitation that it will _only_ compare the **primary** group of the user rather than **all** of the groups that users belongs to.  This limitation is the reason the ip-lockdown script has to create rules per user. As we do not want to modify each user and update their primary user group in case that this causes issues for some other process.

For reference of this limitation see the following:
* [iptables user/group matching module code](https://elixir.bootlin.com/linux/latest/source/net/netfilter/xt_owner.c)
* [iptables-match-output-rule-for-supplementary-groups](https://serverfault.com/questions/742693/iptables-match-output-rule-for-supplementary-groups)