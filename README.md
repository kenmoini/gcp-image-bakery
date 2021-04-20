# GCP RHEL Gold Image Bakery

Red Hat's Cloud Access program provides certified Gold Images to use with a Bring-Your-Own-Subscription model in AWS and Azure but not GCP - for that you'll need to bring your own image.

This repository uses GCP to create a RHEL Gold Image and deposits it into GCP - inception Gold Image baking.

## How It Works

1. With the 

## Prerequiste Setup

### 1. Create a GCP IAM Service Account and JSON key

This is pretty easy to do in GCP.. IAM > Service Accounts > Create, give it a name and permissions, then create a Key, JSON type, it'll download.

### 2. Copy JSON to local directory

By default, the `service_account_file` variable looks for `./gcp-creds.json` - alternatively, set the var to wherever you have it already downloaded.

### 3. Get RHSM Offline API Key

In order to automatically download the RHEL Base Image you need to provide a Red Hat Subscription Manager Offline API Token - you can get one here: https://access.redhat.com/management/api

### 4. Set variables

Modify the `shared_vars.yaml.example` file as needed and copy to `shared_vars.yaml`.

Something to keep an eye on is the RHEL image base - it is updated from time to time and there is no easy way to automatically get the latest image name.  You'll want to run `gcloud compute images | grep 'rhel'` to ensure you are using the latest image.

### 5. Pull in Collections

In case you don't already have the collections on your system, you can load them all with the following command:

```bash
ansible-galaxy install -r collections/requirements.yml
```

---

## Creating the RHEL Gold Image

```bash
ansible-playbook -e "@shared_vars.yaml" gcp-image-bakery.yaml
```

## Delete the Bakery

```bash
ansible-playbook -e "@shared_vars.yaml" destroy-nested-virt-vm.yaml
```

## Available Tags

Listed in order of execution:

- `vm_setup` - This needs to be run or else the `bakery_instance` group isn't made which the following Plays rely on
- `vm_configure` - Safe to skip on subsequent runs of the playbook, targets package management
- `wait_for_ssh` - Safe to skip on subsequent runs of the playbook once the VMs have been stood up
- `rhsm_api` - This interacts with the Red Hat Subscription Manager API and downloads the RHEL KVM image
- `bake_image` - The tasks tagged with this will take the Gold Image on the Bakery VM and create the VM Disk Image in GCP
- `gold_setup` - This tag controls the tasks that create a VM from the created Base Image
- `gold_configure` - Will apply configuration to the booted Base Image VM which will be used to create the final Gold Image
- `gold_destroy` - Safe to skip if you want to keep the Gold Image VM and associated VPC resources online
- `vm_destroy` - Safe to skip if you want to keep the Bakery VM and associated VPC resources online - will fail if Gold Image VM resources aren't deleted first