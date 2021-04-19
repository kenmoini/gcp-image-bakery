# GCP RHEL Gold Image Bakery

Red Hat's Cloud Access program provides certified Gold Images to use with a Bring-Your-Own-Subscription model in AWS and Azure but not GCP - for that you'll need to bring your own image.

This repository uses GCP to create a RHEL Gold Image and deposits it into GCP - inception Gold Image baking.

## Prerequiste Setup

### 1. Create a ServiceAccount and JSON key

This is pretty easy to do in GCP.. IAM > Service Accounts > Create, give it a name and permissions, then create a Key, JSON type, it'll download.

### 2. Copy JSON to local directory

By default, the `service_account_file` var looks for `./gcp-creds.json` - alternatively, set the var to wherever you have it already downloaded.

### 3. Set variables

Modify the `shared_vars.yaml.example` file as needed and copy to `shared_vars.yaml`.

Something to keep an eye on is the RHEL image base - it is updated from time to time and there is no easy way to automatically get the latest image name.  You'll want to run `gcloud compute images | grep 'rhel'` to ensure you are using the latest image.

### 4. Pull in Collections

In case you don't already have the collections on your system, you can load them all with the following command:

```bash
ansible-galaxy install -r collections/requirements.yml
```

## Creating the RHEL Gold Image

```bash
ansible-playbook -e "@shared_vars.yaml" gcp-image-bakery.yaml
```

## Delete the Bakery

```bash
ansible-playbook -e "@shared_vars.yaml" destroy-nested-virt-vm.yaml
```