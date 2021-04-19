# GCP RHEL Gold Image Bakery

Red Hat's Cloud Access program provides certified Gold Images to use with a Bring-Your-Own-Subscription model in AWS and Azure but not GCP - for that you'll need to bring your own image.

This repository uses GCP to create a RHEL Gold Image and deposits it into GCP - inception Gold Image baking.

## Prerequiste Setup

### 1. Create a ServiceAccount and JSON key

This is pretty easy to do in GCP.. IAM > Service Accounts > Create, give it a name and permissions, then create a Key, JSON type, it'll download.

### 2. Copy JSON to local directory

By default, the `service_account_file` var looks for `./gcp-creds.json` - alternatively, set the var to wherever you have it already downloaded.

### 3. Set variables

Modify the `shared_vars.yaml` file as needed for project and so on.

## Creating the RHEL Gold Image

```bash
ansible-playbook gcp-image-bakery.yaml
```

## Delete the Bakery

```bash
ansible-playbook destroy-nested-virt-vm.yaml
```