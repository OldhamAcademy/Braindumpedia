# Terraform - AWS IAM

## Getting Started

### Prerequisite Dependencies
1. Terraform - `brew install terraform`
2. AWS CLI - `brew install awscli`
3. Keybase - [keybase.io](http://keybase.io)

#### AWS CLI
Once you've installed the `awscli` tool, you'll need to configure it (`aws configure`).

**RECOMMENDED**

I advise that you setup your AWS CLI with your own access id and secret and then creating another profile with the credentials correlated with an IAM account that has programmatic access (ideally one that's specifically setup for `terraform`, this helps with CloudTrail auditing). 

To do this, once you finished the first step of setting up your own account keys and secret, you can go into your editor and edit `~/.aws/credentials`. You can duplicate the `default` profile under it.

Example:
```
  1 [default]
  2 aws_access_key_id = mydefaultkeyid
  3 aws_secret_access_key = mydefaultaccesskey
  4
  5 [terraform]
  6 aws_access_key_id = terraformuserkeyid
  7 aws_secret_access_key = terraformuseraccesskey
```

If you need the Terraform credentials, ping someone on your team or create the user in AWS IAM.

#### Keybase
This step is mainly important for the person you need to setup an IAM account for. With terraform, you are required to add a PGP key attached to the user upon creation. Instead of pasting a long pgp key, you can actually point it back to a user as long as they have their encrypted pgp keys uploaded. If you don't have your PGP keys uploaded, you should probably follow this step...

```bash
keybase pgp gen
```
*Important*: It will ask you to upload your PGP file to keybase, select Y for this step.

## Creating a user account 

### Setting up your provider 
```nginx
provider "aws" {
  region = "us-west-2"
  shared_credentials_file = "${pathexpand(
    "~/.aws/credentials"
  )}"
  profile = "myprofile"
  skip_credentials_validation = true
}
```
For profile, you just refer this to the profile you set for `~/.aws/credentials`, in this case your terraform user or "default"

### The Resources
The actual account record to be added to IAM.
```nginx
resource "aws_iam_user" "alanl" {
  name = "alanl"
  path = "/"
  force_destroy = true
}
```

If the goal is to create a service account, you can **stop here**. But if you need to create an account that can login to console then continue.

```nginx
resource "aws_iam_user_login_profile" "alanl" {
  user = "${aws_iam_user.alanl.name}"
  # important, set the username to the person you're setting this account up for, hopefully alanl
  pgp_key = "keybase:<username>"
  password_reset_required = true
}

output "password" {
  value = "${aws_iam_user_login_profile.alanl.encrypted_password}"
}
```

## Getting the password
Now that you have your tf files setup, try running the following commands:
```bash
terraform init
terraform plan
terraform apply # sometimes you need to use sudo with this one ¯\_(ツ)_/¯
```

If all went well, you should've seen some green messages with the plan or changes you want to execute and then it successfully outputting an encrypted password for you. You will also notice an encrypted password output.

Copy that output and paste it to the user that you created the account for. Tell them to store it in a file, for example, called `awspass.txt`. In terminal, they would need to run:

```
cat awspass.txt | base64 --decode | keybase pgp decrypt
```

## More Stuff
Do you want to get IAM access key id and secret? 
(If you're creating a service account and you don't need a login profile, you should add this step too).
```nginx
resource "aws_iam_access_key" "alanl" {
  user    = "${aws_iam_user.alanl.name}"
  pgp_key = "keybase:alanlee"
}
```

Then add the following outputs to file:
```nginx
output "id" {
  value = "${aws_iam_access_key.alanl.id}"
}

output "secret" {
  value = "${aws_iam_access_key.alanl.encrypted_secret}"
}
```

The secret is encrypted, which would require the user to run the base64 and keybase decryption just like they would have had for password.