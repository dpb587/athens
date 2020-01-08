---
title: Install Athens on BOSH
description: Installing an Athens Instance on BOSH
weight: 11
---

Athens can be deployed in many ways. The following guide explains how to use [BOSH](https://bosh.io), a deployment and lifecycle tool, in order to deploy an Athens server on real VMs in any IaaS that is supported by BOSH.

---

## Prerequisites

* Install [BOSH](#install-bosh)
* Setup the [infrastructure](#setup-the-infrastructure)

### Install BOSH

Make sure to have the [BOSH CLI](https://bosh.io/docs/cli-v2-install/) installed and set up a running [BOSH Director](https://bosh.io/docs/quick-start/) on any infrastructure of your choice.

### Setup the Infrastructure

When using a BOSH environment on "real" infrastructure, a few additional components need to be set up beforehand. Depending on which IaaS you will be deploying you may need to create:

* **Public IP**: a public IP address for association with the Athens VM.
* **Firewall Rules**: the following ingress ports must be allowed

    - `3000/tcp` - Athens

    Egress traffic should be restricted depending on your requirements.

#### Amazon Web Services (AWS)

AWS requires additional settings that should be added to a `credentials.yml` file using the following template:

```yaml
# security group IDs to apply to the VM
athens_security_groups: [sg-0123456abcdefgh]

# VPC subnet to deploy Athens to
athens_subnet_id: subnet-0123456789abcdefgh

# a specific, elastic IP address for the VM
external_ip: 3.123.200.100
```

The credentials need to be added to the `deploy` command, i.e.

```
-o manifests/operations/aws-ops.yml
```

#### VirtualBox

The fastest way to install Athens using BOSH is probably a Director VM running in VirtualBox which is sufficient for development or testing purposes. If you follow the bosh-lite [installation guide](https://bosh.cloudfoundry.org/docs/bosh-lite), no further preparation is required to deploy Athens.


## Deployment

The [athens-bosh-release](https://github.com/s4heid/athens-bosh-release) repository includes manifests for basic deployment configurations inside the `manifests` Directory for quickly creating a standalone Athens server.

Once the [infrastructure](#setup-the-infrastructure) has been prepared and the BOSH Director is running, make sure that a [stemcell](https://bosh.cloudfoundry.org/docs/stemcell/) has been uploaded. If this has not been done yet, choose a stemcell from the [stemcells section of bosh.io](https://bosh.io/stemcells), and upload it via the command line. Additionally, a [cloud config](https://bosh.cloudfoundry.org/docs/cloud-config/) is required for IaaS specific configuration used by the Director and the Athens deployment. Clone the repository and cd into the directory

```console
git clone --recursive https://github.com/s4heid/athens-bosh-release.git
cd athens-bosh-release
```

Upload the cloud config if you haven't already

```console
bosh update-config --type=cloud --name=athens \
    --vars-file=credentials.yml manifests/cloud-config.yml
```

Execute the basic `deploy` command which can be extended with ops/vars files depending on which IaaS you will be deploying to.

```console
bosh -d athens deploy manifests/athens.yml  # add extra arguments
```

For example, if using AWS the deploy command for an Athens Proxy with disk storage would look like

```console
bosh -d athens deploy \
    -o manifests/operations/aws-ops.yml \
    -o manifests/operations/with-persistent-disk.yml \
    -v disk_size=1024 \
    --vars-file=credentials.yml manifests/athens.yml
```

This will deploy a single Athens instance in the `athens` deployment with a persistent disk of 1024MB. The IP address of the Athens instance can be obtained with

```console
bosh -d athens instances
```

which is useful for setting the `GOPROXY` variable.