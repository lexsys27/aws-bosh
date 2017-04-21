# Cloud Foundry on AWS

This exercise shows how to install Cloud Foundry on top of BOSH bootstrap installation

## Create load balancers

Cloud Foundry needs two load balancers - one for web and one for SSH.

First generate self-signed certificate for wildcard domain `*.cf.example.com`

```
$ openssl req \
       -newkey rsa:2048 -nodes -keyout cf.example.com.key \
       -x509 -days 365 -out cf.example.com.crt

$ openssl rsa -in cf.example.com.key -out cf.example.com.key.rsa
```

Now create a pair of load balancers and deploy SSL certificate on them.

```
$ bbl create-lbs \
  --type cf \
  --key cf.example.com.key.rsa \
  --cert cf.example.com.crt
```

You need to update `cloud config` for vm extensions with loadbalancers to be available for Cloud Foundry deployment.

```
$ bosh update-cloud-config <(bbl cloud-config)
```

## DNS

Add two CNAME records to your DNS (like Route53)

- `*.cf.example.com` pointing to `CFRouter` load balancer
- `ssh.cf.example.com` pointing to `CFSSHPro` load balancer

Load balancers address can be found in AWS Web Concole.

Example:

```
*.cf.example.com.     CNAME stack-bbl-CFRouter-IAS5K0ZQ0B29-688572787.eu-central-1.elb.amazonaws.com.
ssh.cf.example.com. CNAME stack-bbl-CFSSHPro-PCU0FRV05OLH-1930364030.eu-central-1.elb.amazonaws.com.
```

## Stemcell

Upload stemcell

```
$ bosh upload-stemcell https://bosh.io/d/stemcells/bosh-aws-xen-hvm-ubuntu-trusty-go_agent\?v\=3363.15
```

## Deployment

Clone deployment manifest and specify system domain.

```
$ git clone https://github.com/cloudfoundry/cf-deployment.git c

# Add line system_domain: cf.domain.com
$ vim c/cf-deployment-vars.yml
```

Now generate all other variables

```
$ bosh -n interpolate \
  --vars-store c/cf-deployment-vars.yml \
  -o c/operations/aws.yml \
  -o c/operations/scale-to-one-az.yml \
  --var-errs c/cf-deployment.yml
```

And finally deploy Cloud Foundry

```
$ bosh -d cf deploy \
  --vars-store c/cf-deployment-vars.yml \
  -o c/operations/aws.yml \
  -o c/operations/scale-to-one-az.yml \
  c/cf-deployment.yml
```

## Verify

To verify the deployment run `smoke tests`.

```
$ bosh -d cf run-errand smoke-tests
```
