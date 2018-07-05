**Note**: This public repo contains the documentation for the private GitHub repo <https://github.com/gruntwork-io/module-security>.
We publish the documentation publicly so it turns up in online searches, but to see the source code, you must be a Gruntwork customer.
If you're already a Gruntwork customer, the original source for this file is at: <https://github.com/gruntwork-io/module-security/blob/master/examples/ssh-grunt/packer/README.md>.
If you're not a customer, contact us at <info@gruntwork.io> or <http://www.gruntwork.io> for info on how to get access!

# Example ssh-grunt Packer templates

This folder contains a couple Packer templates that show examples of how to create an Amazon Machine Image (AMI) with
[ssh-grunt](/modules/ssh-grunt) installed. One of the AMIs is configured to use `ssh-grunt` with IAM and the other
with Gruntwork Houston.

**Note**: To make it possible to run automated tests against these examples, we build the `ssh-grunt` binary locally and
use the Packer `file` provisioner to copy it into our AMI. This is NOT how you would do it in a real-world use case.
Instead, we recommend using the [Gruntwork installer](https://github.com/gruntwork-io/gruntwork-installer):

```json
{
  "type": "shell",
  "inline": "curl -Ls https://raw.githubusercontent.com/gruntwork-io/gruntwork-installer/master/bootstrap-gruntwork-installer.sh | bash /dev/stdin --version {{user `gruntwork_installer_version`}}"
},
{
  "type": "shell",
  "inline": [
    "gruntwork-install --binary-name ssh-grunt --tag v0.14.0 --repo https://github.com/gruntwork-io/module-security",
    "sudo /usr/local/bin/ssh-grunt iam install --iam-group MyIamGroup --iam-group-sudo MyIamSudoGroup"
  ],
  "environment_vars": [
    "GITHUB_OAUTH_TOKEN={{user `github_auth_token`}}"
  ]
}
```




## Building the IAM example AMI

1. Create two IAM groups: one with IAM users that need SSH access with sudo privileges and one with IAM users that need
   SSH access without sudo privileges.
1. Build the `ssh-grunt` binary: `build-binary.sh`
1. Build the Packer template under `ssh-grunt-iam.json`, passing the name of the groups you created in the previous
   step using the `iam_group_sudo` and `iam_group` variables:
   `packer build -var iam_group_sudo=ssh-sudo-group -var iam_group=ssh-group ssh-grunt-iam.json`.




## Building the Houston example AMI

1. Add SSH Roles—one for normal SSH access and one for SSH access with sudo permissions—to users in your Identity
   Provider (i.e., in Google or ADFS).
1. Build the `ssh-grunt` binary: `build-binary.sh`
1. Build the Packer template under `ssh-grunt-houston.json`, passing the name of the SSH Roles you created in the previous
   step using the `ssh_role` and `ssh_role_sudo` variables, as well as the URL of your Houston deployment using the
   `houston_url` variable:
   `packer build -var ssh_role=ssh-role -var ssh_role_sudo=ssh-sudo-role -var houston_url=https://houston.your-company.com ssh-grunt-houston.json`.
