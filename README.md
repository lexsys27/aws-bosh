# aws-bosh
Exercise of deploying BOSH to AWS using bosh-bootloader

# Prerequisites

## Install BOSH v2 CLI

```
$ brew install cloudfoundry/tap/bosh-cli
$ source ~/.zshrc
$ bosh2 -v
version 2.0.14-d5180d5-2017-04-18T00:16:26Z
```

## Update Terraform

```
$ tf version
Terraform v0.7.4
$ brew upgrade terraform
$ tf version
Terraform v0.9.3
```

You don't need `bosh-init` installed separately.

## Install BOSH-bootloader

```
$ wget https://github.com/cloudfoundry/bosh-bootloader/releases/download/v3.0.4/bbl-v3.0.4_osx
$ chmod +x bbl-v3.0.4_osx
$ sudo mv bbl-v3.0.4_osx /usr/local/bin/bbl
$ bbl version
bbl 3.0.4 (darwin/amd64)
```

## Create AWS user and assign policy

Policy

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ec2:*",
                "cloudformation:*",
                "elasticloadbalancing:*",
                "iam:*"
            ],
            "Resource": [
                "*"
            ]
        }
    ]
}
```

It has very wide permissions - starting and stopping instances in EC2 and managing users in IAM.

You need to copy this policy to clipboard before executing following commands.

```
$ aws iam create-user --user-name "lx-bbl"
$ aws iam put-user-policy --user-name "lx-bbl" \
        --policy-name "lx-bbl-policy" \
        --policy-document "$(pbpaste)"
$ aws iam create-access-key --user-name "lx-bbl"
```

# Deployment

```
bbl up \
	--aws-access-key-id <INSERT ACCESS KEY ID> \
	--aws-secret-access-key <INSERT SECRET ACCESS KEY> \
	--aws-region eu-central-1 \
	--iaas aws
```

# Verification

```
$ bbl director-address
https://35.146.48.144:25555
$ bbl director-username
admin
$ bbl director-password
XXX

$ bbl director-ca-cert > bosh.crt
$ export BOSH_CA_CERT=bosh.crt
$ export BOSH_ENVIRONMENT=https://35.146.48.144:25555

$ bosh alias-env lx
$ bosh log-in
$ bosh cloud-config
```

# Notes

- AWS Cloud Formation stack needs to be manually deleted before restarting deployment process
- Why do we need Cloud Formation if there is already terraform performing the same function in vendor independent way?
- `bosh-init` is the part of `bosh-cli` now and you don't need to install it separately
- commands for targeting to bosh director are for the first version
- installation names `bosh2` by default - `bbl` expects `bosh` so link it
