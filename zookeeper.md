# Deploy Zookeeper using BOSH v2

Zookeeper is the simpliest release with BOSH links support you can play with

## Preparation

```
$ bosh upload-stemcell https://bosh.io/d/stemcells/bosh-aws-xen-hvm-ubuntu-trusty-go_agent
```

Zookeeper manifest (from [here](https://github.com/cppforlife/zookeeper-release/blob/master/manifests/zookeeper.yml)):

```
---
name: zookeeper

releases:
- name: zookeeper
  version: 0.0.3
  url: https://bosh.io/d/github.com/cppforlife/zookeeper-release?v=0.0.3
  sha1: ae79ed352a4e0bdf398d188d6944fbb782668c7e

stemcells:
- alias: default
  os: ubuntu-trusty
  version: latest

update:
  canaries: 2
  max_in_flight: 1
  canary_watch_time: 5000-60000
  update_watch_time: 5000-60000

instance_groups:
- name: zookeeper
  azs: [z1, z2]
  instances: 3 
  jobs:
  - name: zookeeper
    properties: {}
  vm_type: default
  stemcell: default
  persistent_disk: 10240
  networks:
  - name: default

- name: smoke_tests
  azs: [z1]
  lifecycle: errand
  instances: 1
  jobs:
  - name: smoke_tests
    properties: {}
  vm_type: default
  stemcell: default
  networks:
  - name: default
```

I have changed the number of availability zones to two and number of instances to 3.

## Deploy

```
bosh -d zookeeper deploy zookeeper.yml
```

## Verify

```
# show instances
$ bosh -d zookeeper instances

# run tests
$ bosh -d zookeeper run-errand smoke_tests
```

## Clean Up

```
$ bosh -d zookeeper delete-deployment
$ bosh -e aws-demo clean-up --all
```

## Useful Links

1. [Bootstrap BOSH 2.0 on AWS](http://starkandwayne.com/blog/bootstrap-bosh-2-0-on-aws/)
2. [Zookeeper BOSH release](https://github.com/cppforlife/zookeeper-release)
3. [BOSH v2 CLI](https://github.com/cloudfoundry/bosh-cli)
4. [Apache ZooKeeper](https://zookeeper.apache.org)
