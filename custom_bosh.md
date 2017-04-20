# Custom BOSH

This excercise shows how to deploy BOSH with custom manifest.

## Prerequisites

The same as for [ordinary deployment](./README.md).

## Deployment

Set up infrastructure without installing the `Director`.

```
$ bbl up \
	--aws-access-key-id <INSERT ACCESS KEY ID> \
	--aws-secret-access-key <INSERT SECRET ACCESS KEY> \
	--aws-region eu-central-1 \
	--iaas aws \
	--no-director
  
```

Clone manifest, make nessesary modifications and deploy BOSH.

```
$ git clone https://github.com/cloudfoundry/bosh-deployment.git deploy
bosh create-env deploy/bosh.yml  \
  --state ./state.json  \
  -o deploy/aws/cpi.yml  \
  -o deploy/external-ip-with-registry-not-recommended.yml \
  --vars-store ./creds.yml  \
  -l <(bbl bosh-deployment-vars) 
```

## Verify

```
$ eval "$(bbl print-env)"
$ export BOSH_CA_CERT="$(bosh int creds.yml --path /default_ca/ca)"
$ export BOSH_CLIENT_SECRET="$(bosh int creds.yml --path /admin_password)"
$ export BOSH_CLIENT=admin
$ bosh deployments
```
