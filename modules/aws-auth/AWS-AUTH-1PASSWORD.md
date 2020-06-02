**Note**: This public repo contains the documentation for the private GitHub repo <https://github.com/gruntwork-io/module-security>.
We publish the documentation publicly so it turns up in online searches, but to see the source code, you must be a Gruntwork customer.
If you're already a Gruntwork customer, the original source for this file is at: <https://github.com/gruntwork-io/module-security/blob/master/modules/aws-auth/AWS-AUTH-1PASSWORD.md>.
If you're not a customer, contact us at <info@gruntwork.io> or <http://www.gruntwork.io> for info on how to get access!

# 1Password with AWS

## `aws-auth` script

Before reading these instructions, go through setting up `aws-auth` and understanding the [aws-auth workflow](https://github.com/gruntwork-io/module-security-public/tree/master/modules/aws-auth/README.md).

## `jq`

You need to install the json tool `jq` for this to work. On a Mac, this can be as simple as `brew install jq`. If this does not work, or if you are on another platform, see [the documentation](https://stedolan.github.io/jq/download/) for further details.

## Setting up the AWS 1Password Record

To make this a little easier to use, add a few extra fields to the record. To make this integration most effective, you should also use 1Password for AWS MFA (within the same record).
 * Add a text field with the label `AWS_ACCESS_KEY_ID` and the value of your AWS_ACCESS_KEY_ID.
 * Add a password field with the label `AWS_SECRET_ACCESS_KEY` and the value of your AWS_SECRET_ACCESS_KEY.
 * Add a text field with the label `AWS_TOTP_SERIAL_NUMBER` and the value of your AWS_TOTP_SERIAL_NUMBER - this should look like `arn:aws:iam::<ACCOUNT_NUMBER>:mfa/<USER>`.

## `op`

Install and configure the 1Password CLI, `op`, according to [the instructions](https://support.1password.com/command-line-getting-started/).

You will need to retrieve your AWS credential UUID with something like this:

```
$ op get item '<1PASSWORD AWS LOGIN NAME>' | jq -r '.uuid' | pbcopy
```

## Setting up your shell function to combine with 1Password

Add this function to your `.zshrc`, `.bashrc` or whatever:

```shell
aws_auth() {
  AWS_ITEM_ID="<UUID YOU COPIED EARLIER>"
  AWS_ITEM=(op get item "$AWS_ITEM_ID" --fields AWS_ACCESS_KEY_ID,AWS_SECRET_ACCESS_KEY,AWS_TOTP_SERIAL_NUMBER)
  AWS_CREDS="$($AWS_ITEM)" || eval "$(op signin)" && AWS_CREDS="$($AWS_ITEM)"
  export AWS_ACCESS_KEY_ID="$(echo $AWS_CREDS | jq -r '.AWS_ACCESS_KEY_ID')"
  export AWS_SECRET_ACCESS_KEY="$(echo $AWS_CREDS | jq -r '.AWS_SECRET_ACCESS_KEY')"
  AWS_AUTH=(aws-auth --serial-number $(echo $AWS_CREDS | jq -r '.AWS_TOTP_SERIAL_NUMBER') --token-code $(op get totp "$AWS_ITEM_ID"))
  if [ -z "$1" ]; then
    eval "$($AWS_AUTH)"
  else
    eval "$($AWS_AUTH --role-arn $1)"
  fi
}
```
then run `source ~/.bashrc` or `source ~/.zshrc` or whatever.

## Usage
This allows you to run `aws_auth` to add your MFA token to the current session, or `aws_auth <ROLE ARN>` to assume a new role.
