**Note**: This public repo contains the documentation for the private GitHub repo <https://github.com/gruntwork-io/module-security>.
We publish the documentation publicly so it turns up in online searches, but to see the source code, you must be a Gruntwork customer.
If you're already a Gruntwork customer, the original source for this file is at: <https://github.com/gruntwork-io/module-security/blob/master/modules/aws-auth/AWS-AUTH-LASTPASS.md>.
If you're not a customer, contact us at <info@gruntwork.io> or <http://www.gruntwork.io> for info on how to get access!

# LastPass with AWS


## `aws-auth` script

Before reading these instructions, go through setting up `aws-auth` and understanding the [aws-auth workflow](https://github.com/gruntwork-io/module-security-public/tree/master/modules/aws-auth/README.md).

## Combining it with LastPass

If you've read the [aws-auth README](https://github.com/gruntwork-io/module-security-public/tree/master/modules/aws-auth/README.md), you'll find that using `aws-auth` isn't *really* a one-liner, since you have to set your permanent AWS credentials first:

```bash
export AWS_ACCESS_KEY_ID='<PERMANENT_ACCESS_KEY>'
export AWS_SECRET_ACCESS_KEY='<PERMANENT_SECRET_KEY>'
eval $(aws-auth --serial-number arn:aws:iam::123456789011:mfa/jondoe --token-code 123456)
```

If you store your secrets in a CLI-friendly password manager, such as [lpass](https://github.com/lastpass/lastpass-cli) or [pass](https://github.com/gruntwork-io/module-security-public/tree/master/modules/aws-auth#combining-it-with-password-managers), 
then you can reduce this even further!

If needed, you can create a LastPass account and install the client from [here](https://www.lastpass.com/). The CLI client can be installed from [here](https://github.com/lastpass/lastpass-cli).

#### Create templates

First, store your permanent AWS credentials in `lpass`. I'm sure there are multiple ways this can be done but I created a 
custom note and stored all user account information in it.

1. Open your LastPass Vault > Secure Notes > click on the (+) sign > Add Secure Note.
2. Change "Note Type:" to "Add Custom Template".
3. Give it a name like "AWS Security Credentials".
4. Start adding fields.
5. Save the template when complete.

Fields I used were:

| Field Name    | Field Type    |
| ------------- |-------------: |
| User ID (IAM) | Text |
| Account Name  | Text |
| Account Number | Text |
| Access Key ID | Text with copy button |
| Secret Access Key | Text with copy button |
| MFA ARN | Text with copy button |

Once that's complete, create a second note used to for storing lines of the script.

1. Open your LastPass Vault > Secure Notes > click on the (+) sign > Add Secure Note.
2. Change "Note Type:" to "Add Custom Template".
3. Give it a name like "Script".
4. Start adding fields.
5. Save the template when complete.

Fields I used were:

| Field Name    | Field Type    |
| ------------- |-------------: |
| Param1 | Text |
| Param2 | Text |
| Param3 | Text |
| Param4 | Text |


#### Save secrets and script parameters in LastPass

Create a new Secure Note using the "AWS Security Credentials" template. Store your User ID ARN, Account Name, Account Number, 
Access Key ID, Secret Access Key, and MFA ARN in that. Custom templates can't be added from the `lpass` cli so this has to be 
done within the LastPass GUI.

```bash
$ lpass show aws-johndoe
Foldername/aws-johndoe [id: 1234567890123456]
MFA ARN: arn:aws:iam::123456789012:mfa/johndoe
Secret Access Key: JOHNDOERANDOMSECRETACCESSKEY
Access Key ID: JOHNDOEACCESSKEYID
Account Number: 123456789012
Account Name: Security
User ID (IAM): arn:aws:iam::123456789012:user/johndoe
NoteType: Custom
```

If you will be assuming an IAM Role ARN, put that in `lpass` too:

```bash
lpass add aws-johndoe-role-arn-otheraccount
Username: eval $(AWS_ACCESS_KEY_ID=$(lpass show aws-johndoe --field "Access Key ID") AWS_SECRET_ACCESS_KEY=$(lpass show aws-johndoe --field "Secret Access Key") aws-auth --serial-number $(lpass show aws-johndoe --field "MFA ARN") --token-code "$token" --role-arn $(lpass show aws-johndoe-role-arn-otheraccount --password))
Password: arn:aws:iam::098765432109:role/role-name
```

_Note: For the IAM Role ARNs, I'm using the default `lpass` site template since I only need two fields and I can access it quickly with the CLI. The script with the `--role-arn` flag is saved as `Username` and the Role ARN is being saved as `Password`._

#### Start putting the script together

Now, we can start constructing our script in `lpass` that ties all of this together. Again, since this is a custom template, everything has to
be done within the LastPass console. Create a new Secure Note using the "Script" template.

In the Param1 field, copy in:

```bash
read -p "Enter your MFA token: " token
```

_Note: The other ParamX fields were added for future use and not necessarily for this script._

```bash
$ lpass show aws-auth-security
Foldername/aws-auth-security [id: 7056520004343215957]
Param4:
Param3:
Param2: eval $(AWS_ACCESS_KEY_ID=$(lpass show aws-johndoe --field "Access Key ID") AWS_SECRET_ACCESS_KEY=$(lpass show aws-johndoe --field "Secret Access Key") aws-auth --serial-number $(lpass show aws-johndoe --field "MFA ARN") --token-code "$token")
Param1: read -p "Enter your MFA token: " token
NoteType: Custom
```

Now, to setup your temporary STS credentials so it is *truly* a one-liner!
 
```bash
eval "$(lpass show aws-auth-security --field Param1; lpass show aws-auth-security --field Param2)"
```

**Example**

```bash
$ eval "$(lpass show aws-auth-security --field Param1; lpass show aws-auth-security --field Param2)"
Enter your MFA token: 123456
2018-01-08 16:35:18 [INFO] [aws-auth] Getting temporary credentials and token for MFA device arn:aws:iam::123456789012:mfa/johndoe
2018-01-08 16:35:19 [INFO] [aws-auth] Success!
```

#### Alias all the things!

But that's still a lot of typing. How about we alias that and all the additional IAM Role ARN possibilities? I keep all my aliases defined in `~/.bash_aliases`.

_Note: Remember that `aws-auth-otheraccount` requires we specify the `--role-arn` so we can switch to that role/account. In this example, all 
of that is stored in `lpass` as the secret `aws-johndoe-role-arn-otheraccount`. The `Username` field contains the script and the `Password` field
contains the Role ARN._

_For every AWS account used in your organization, you'll need to create that additional secret and that BASH alias if you're going to follow along with this.
Remember to adjust your `alias` scripts as neccessary._

```bash
# Authenticate to the AWS account with the User ID
alias aws-auth-security='eval "$(lpass show aws-auth-security --field Param1; lpass show aws-auth-security --field Param2)"'

# Authenticate to the AWS account with the role arn
alias aws-auth-otheraccount='eval "$(lpass show aws-auth-security --field Param1; lpass show aws-johndoe-role-arn-otheraccount --username)"'
```

*Note: the double quotes around the `$()` are required.*

**Example**

```bash
$ aws-auth-security
Enter your MFA token: 123456
2018-01-09 21:14:18 [INFO] [aws-auth] Getting temporary credentials and token for MFA device arn:aws:iam::123456789012:mfa/johndoe
2018-01-09 21:14:19 [INFO] [aws-auth] Success!

$ aws-auth-otheraccount
Enter your MFA token: 234567
2018-01-09 21:14:41 [INFO] [aws-auth] Getting temporary credentials and token for MFA device arn:aws:iam::123456789012:mfa/johndoe
2018-01-09 21:14:42 [INFO] [aws-auth] Assuming role arn:aws:iam::098765432109:role/rolename
2018-01-09 21:14:42 [INFO] [aws-auth] Success!

$ aws sts get-caller-identity
{
    "UserId": "AWSRANDOMUSERID:johndoe",
    "Account": "098765432109",
    "Arn": "arn:aws:sts::098765432109:assumed-role/rolename/johndoe"
}
```


### Thanks

- [LastPass](https://github.com/lastpass/lastpass-cli), for creating a CLI.
- [Gruntwork.io](https://github.com/gruntwork-io), for creating the initial script and instructions.
