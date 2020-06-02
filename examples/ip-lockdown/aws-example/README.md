**Note**: This public repo contains the documentation for the private GitHub repo <https://github.com/gruntwork-io/module-security>.
We publish the documentation publicly so it turns up in online searches, but to see the source code, you must be a Gruntwork customer.
If you're already a Gruntwork customer, the original source for this file is at: <https://github.com/gruntwork-io/module-security/blob/master/examples/ip-lockdown/aws-example/README.md>.
If you're not a customer, contact us at <info@gruntwork.io> or <http://www.gruntwork.io> for info on how to get access!

# ip-lockdown AWS Example

This is an example of a real world scenario where we demonstrate how
[ip-lockdown](/modules/ip-lockdown) can be invoked as part of `User Data` to automatically lock down the EC2 metadata IP upon
startup of an EC2 instance.

The main motivation for locking down EC2 metadata is as follows:

1. EC2 metadata gives you the credentials you need to assume any IAM role associated with the EC2 instance, and thereby, get all the permissions available in that IAM role.
1. Locking down the metadata to, for example, only the root user, makes sure that if a hacker breaks into your server with a privileged user, they cannot get the full power of the IAM role.

For example, we often give EC2 Instances access to a KMS key via an IAM role so that instance can decrypt secrets before booting. Using ip-lockdown ensures that only the root user can access that KMS key, and therefore, secrets, which is a big improvement to your security posture.

## Quick start

To build the AMIs:

1. Install [Packer](https://www.packer.io/)
1. Set your [GitHub access token](https://help.github.com/articles/creating-an-access-token-for-command-line-use/) as
   the environment variable `GITHUB_OAUTH_TOKEN`.
1. Run `packer build ../ip-lockdown-sample.json`

Running the Terraform Code

1. Edit `variables.tf` and add your own values for: `aws_region` and `key_name`
    1. `aws_region` - This is the AWS region where your EC2 instance will be created. Ex: `us-east-1`
    1. `key_name` - This is the name of the AWS key pair you will use to ssh to this EC2 instance. To create a new key pair see [this guide](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html?icmpid=docs_ec2_console#having-ec2-create-your-key-pair)
1. Run `terraform init`
1. Run `terraform plan`
    * Verify that you will be creating one new EC2 instance and one new security group to associate with that instance.
1. Run `terraform apply`

Verifying that `ip-lockdown` worked.

1. Check your [EC2 Dashboard](https://console.aws.amazon.com/ec2) for your new EC2 instance.
1. SSH to your new EC2 instance
1. Run `sudo iptables -L` and observe that there are now rules that block access for all users except `root`. Example below.

    ``` 
    Chain INPUT (policy ACCEPT)
    target     prot opt source               destination         

    Chain FORWARD (policy ACCEPT)
    target     prot opt source               destination         

    Chain OUTPUT (policy ACCEPT)
    target     prot opt source               destination         
    ACCEPT     all  --  anywhere             instance-data.ec2.internal  owner UID match root
    REJECT     all  --  anywhere             instance-data.ec2.internal  reject-with icmp-port-unreachable
    ```
1. Try to curl metadata by doing `curl http://169.254.169.254/latest/meta-data/` (You should receive an error: `curl: (7) Failed to connect to 169.254.169.254 port 80: Connection refused`)
1. Try to curl the metadata again: `sudo curl http://169.254.169.254/latest/meta-data/` (this should work)
