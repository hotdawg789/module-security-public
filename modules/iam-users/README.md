**Note**: This public repo contains the documentation for the private GitHub repo <https://github.com/gruntwork-io/module-security>.
We publish the documentation publicly so it turns up in online searches, but to see the source code, you must be a Gruntwork customer.
If you're already a Gruntwork customer, the original source for this file is at: <https://github.com/gruntwork-io/module-security/blob/master/modules/iam-users/README.md>.
If you're not a customer, contact us at <info@gruntwork.io> or <http://www.gruntwork.io> for info on how to get access!

# IAM Users

This is a Terraform module you can use to create and manage
[IAM users](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users.html) as code.




## How do you use this module?

This module allows you to pass in a map of users to create, where the keys in the map are the user names, and the
values are the following properties for that IAM user (all optional):

* `groups`: a list of IAM groups to add the user to.
* `tags`: a map of tags to apply to the user.
* `pgp_key`: either a base-64 encoded PGP public key, or a [Keybase](https://keybase.io) username in the form
  `keybase:<USERNAME>`, used to encrypt the user's credentials. Required if `create_login_profile` or
  `create_access_keys` is true.
* `create_login_profile`: if set to true, create a password for this user that can be used to login to the AWS Web
  Console. The password will be encrypted using `pgp_key`, so it will NOT be stored in plain text in Terraform state.
* `create_access_keys`: if set to true, create access keys for this user that can be used to authenticate to AWS
  programmatically. The secret access key will be encrypted using `pgp_key`, so it will NOT be stored in plain text in
  Terraform state.
* `path`: the path for the IAM user
* `permissions_boundary`: the ARN of the policy that is used to set the permissions boundary for the user.
* `ssh_public_key`: the public part of the user's SSH key that will be added to their security credentials (e.g., for use with `ssh-grunt`). Currently only supports keys in ssh-rsa format"

Check out the [iam-users example](/examples/iam-users) for working sample code.




## How do you generate passwords and access keys with this module?

This module can optionally create a password for AWS Web Console access and/or access keys for programmatic access for
each IAM user if you set `create_login_profile` and/or `create_access_keys` to `true` for that IAM user, respectively.

To avoid having these secrets stored in plain text in Terraform state, this module will only generate the password or
access keys if you specify the `pgp_key` param for that user. This param can contain either the base-64 encoded PGP
public key for that user or the user's Keybase username in the format `keybase:<USERNAME>`.

We recommend using Keybase, as it makes it easier to manage PGP keys. Have each user at your company:

1. [Install the Keybase app](https://keybase.io/download).
1. Claim a Keybase username.
1. Use the Keybase app to create a PGP key and add it to their profile.
1. Send you their username.

Once you have their user name, set `pgp_key = "keybase:<USERNAME>"` and `create_login_profile` and/or
`create_access_keys` to `true` for that user, and this module will generate the password and/or access keys, and
export them in the output variables `user_passwords` and `user_access_keys`. The output will look something like this:

```
user_access_keys = {
  "alice" = {
    "access_key_id" = "AKIARIUU2OIYE2APGOYK"
    "secret_access_key" = "wcBMA7E6Kn/t1YPfAQgAjLvUWUES/GeLHr/=="
  }
}
user_passwords = {
  "bob" = "wcBMA7E6Kn/t1YPfAQgAdByWFftehuD3uw="
}
```

You can see that Alice's `secret_access_key` and Bob's password are encrypted, so you can safely mail those credentials
to each user.




### How do you decrypt the generated passwords and access keys?

To decrypt a user's password or access keys, that user can decrypt them on the command-line as follows:

```bash
echo "<SECRET>" | base64 --decode | keybase pgp decrypt
```

Note that this only works if the user has the private key for their PGP key on their local computer (which they will
if they used the Keybase app to create the PGP key in the first place).


