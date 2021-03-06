# GKE

Follow these steps to deploy the infrastructure that will hold the Aporeto Control Plane on GKE.

## Requirements

You will need:

* [Terraform 0.11.14](https://www.terraform.io/downloads.html)
* a GCS bucket where to store the terraform states
* the path of your cloud provider's service account
* The name of your cloud provider's project

## Steps

1. Copy the [terraform template for GKE](gke.template.tf) into a file named `main.tf`
2. Change the `<variable>` values:
    * `<bucket>`: Name of the shared GCS bucket where to store the terraform states
    * `<path>`: Path to your cloud provider's service account
    * `<project>`: Name of your cloud provider's project
3. Deploy the terraform script from the folder that contains your `main.tf`

```bash
# Initialize terraform
terraform init
```

``` bash
# Create a workspace for each locations
terraform workspace new active
terraform workspace select active

# Check deployment
terraform plan

# Run deployment (Takes about ~20 minutes)
terraform apply
```

## Example

Below is an example of a `main.tf` file on GKE.

```hcl
terraform {
  backend "gcs" {
    # Name of the shared GCS bucket where to store the terraform states
    bucket = "tf-states"
    prefix = "production"
  }
}

provider "google" {
  # Path to your cloud provider's service account
  credentials = "${file("~/.config/gcloud/application_default_credentials.json")}"
  # Name of your cloud provider's project
  project     = "demoproject"
}

module "aporeto-gke" {
  source = "git::https://github.com/aporeto-inc/tabularasa//modules/aporeto-gke"

  cluster_name               = "${local.cluster_name}"
  location                   = "${local.location}"
  disable_highwind_node_pool = "${local.disable_highwind_node_pool}"
}

locals {
  env = "${terraform.workspace}"

  # Define our setups
  names = {
    "active"  = "production-west"
    "passive" = "production-central"
  }

  locations = {
    "active"  = "us-west1"
    "passive" = "us-central1"
  }

  without_highwind = {
    "active"  = false
    "passive" = true
  }

  # set our vars
  cluster_name = "${lookup(local.names, local.env)}"
  location     = "${lookup(local.locations, local.env)}"

  disable_highwind_node_pool = "${lookup(local.without_highwind, local.env)}"

}
```
