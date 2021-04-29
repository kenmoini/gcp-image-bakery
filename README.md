# GCP RHEL Gold Image Bakery

Red Hat's Cloud Access program provides certified Gold Images to use with a Bring-Your-Own-Subscription model in AWS and Azure but not GCP - for that you'll need to bring your own image.

This repository uses GCP to create a VM, which creates a RHEL Base Image, which is used to create another VM, which is then configured to be the RHEL Gold Image, and deposits that VM as a Compute Disk Image into GCP for use with your own subscriptions via Red Hat Cloud Access - inception Gold Image baking!

## How It Works

With the proper input (GCP SA, RHSM API Key, etc) the `gcp-image-bakery.yaml` Playbook will:

1. Create a Storage Bucket if the specified bucket does not exist (optionally can skip deletion of bucket on Playbook completion)
2. Create an ephemeral VPC in GCP
3. Create a VM with Nested Virtualization enabled (the Bakery VM)
4. Configure the Bakery VM with Libvirt and Guestfish, start Libvirt
5. Download the RHEL KVM Guest Image from Red Hat's CDN via API requests
6. Set the initial root password of the image, convert to needed format for use in GCP
7. Upload to GCP Storage Bucket, import as new Compute Disk Image

You could by all means stop right here with this RHEL Base Image, but you don't have access to all the key features of Compute Engine - plus, what if you want to include other things such as agents and other authentication means in your Gold Image?

From this point the Playbook will:

8. Create a new VM (the Gold Image Smelter VM) in the same VPC, built from the RHEL Base Image that was just created
9. Connect to the Smelter VM, configure basics as needed, finalize by sealing the VM and adding GCP Compute Engine configuration
10. Shut down the Smelter VM, convert to a VM Snapshot, then a Compute Disk Image which will be the final Gold Image
11. Clean up after itself, only leaving the Base and Gold Images behind as Compute Disk Images

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

## Usage

```bash
ansible-playbook -e "@shared_vars.yaml" gcp-image-bakery.yaml
# to destroy:
ansible-playbook -e "@shared_vars.yaml" --skip-tags=vm_setup,base_play,gold_recycle,gold_smelter,gold_casting gcp-image-bakery.yaml
```

## Available Tags

Listed in order of execution:

- `local_pip_gcp_storage` - This runs locally to ensure the needed PIP modules are loaded for interacting with GCP Cloud Storage
- `vm_setup` - This needs to be run or else the `bakery_instance` group isn't made which the following Plays rely on
- `vm_configure` - Safe to skip on subsequent runs of the playbook, targets package management
- `wait_for_ssh` - Safe to skip on subsequent runs of the playbook once the VMs have been stood up
- `rhsm_api` - This interacts with the Red Hat Subscription Manager API and downloads the RHEL KVM image
- `bake_image` - The tasks tagged with this will take the Gold Image on the Bakery VM and create the VM Disk Image in GCP
- `gold_destroy` - Safe to skip if you want to keep the Gold Image VM and associated VPC resources online
- `gold_setup` - This tag controls the tasks that create a VM from the created Base Image
- `gold_configure` - Will apply configuration to the booted Base Image VM which will be used to create the final Gold Image
- `gold_destroy` - Safe to skip if you want to keep the Gold Image VM and associated VPC resources online
- `vm_destroy` - Safe to skip if you want to keep the Bakery VM and associated VPC resources online - will fail if Gold Image VM resources aren't deleted first

## TODO

- Explore other options of providing authentication to the GCP modules