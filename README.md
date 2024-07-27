# Google Kubernetes Engine (GKE) Cluster Terraform module

This module deploys a Kubernetes cluster on Google Cloud Platform (GCP) using the Google Kubernetes Engine (GKE) service. The GKE cluster is provisioned with a single node pool, and it comes with a generated Kubernetes configuration file (`kubeconfig`) that is stored locally.

## Cloud Resource Cost Estimation

This document provides an estimation of monthly costs for the detected cloud resources.

### google_container_node_pool.this

- **Instance usage** (Linux/UNIX, on-demand, c2d-highcpu-2)
  - **Monthly Qty**: 2,920 hours
  - **Monthly Cost**: $52.53

- **Standard provisioned storage** (pd-standard)
  - **Monthly Qty**: 400 GB
  - **Monthly Cost**: $16.00

### google_container_cluster.this

- **Cluster management fee**
  - **Monthly Qty**: 730 hours
  - **Monthly Cost**: $73.00

## Overall Total

- **Total Monthly Cost**: $141.53

*Usage costs can be estimated by updating Infracost Cloud settings, see docs for other options.*

---

2 cloud resources were detected:
- 2 were estimated

| Project | Baseline cost | Usage cost* | Total cost |
|---------|----------------|-------------|------------|
| main    | $142           | -           | $142       |

## Usage

```terraform
provider "google" {
  # Configuration options
  project = var.GOOGLE_PROJECT
  region  = var.GOOGLE_REGION
}

resource "google_container_cluster" "this" {
  name     = var.GKE_CLUSTER_NAME
  location = var.GOOGLE_REGION

  initial_node_count       = 1
  remove_default_node_pool = true
}

resource "google_container_node_pool" "this" {
  name       = var.GKE_POOL_NAME
  project    = google_container_cluster.this.project
  cluster    = google_container_cluster.this.name
  location   = google_container_cluster.this.location
  node_count = var.GKE_NUM_NODES

  node_config {
    machine_type = var.GKE_MACHINE_TYPE
  }
}

module "gke_auth" {
  depends_on = [
    google_container_cluster.this
  ]
  source               = "terraform-google-modules/kubernetes-engine/google//modules/auth"
  version              = ">= 24.0.0"
  project_id           = var.GOOGLE_PROJECT
  cluster_name         = google_container_cluster.this.name
  location             = var.GOOGLE_REGION
}

resource "local_file" "kubeconfig" {
  content  = module.gke_auth.kubeconfig_raw
  filename = "${path.module}/kubeconfig"
}

output "kubeconfig" {
  value = "${path.module}/kubeconfig"
}
```

## Inputs

|       Name       |            Description           |  Type  |     Default     | Required |
|:----------------:|:--------------------------------:|:------:|:---------------:|:--------:|
| GOOGLE_PROJECT   | GCP project name                 | string | no              |    no    |
| GOOGLE_REGION    | GCP region name                  | string | "us-central1-c" |    no    |
| GKE_MACHINE_TYPE | GKE node machine type            | string | "g1-small"      |    no    |
| GKE_NUM_NODES    | Number of nodes in the node pool | number | 2               |    no    |

## Outputs
kubeconfig - Generated Kubernetes configuration file

## Requirements
This module requires Terraform 0.12 or later, and the following provider:

hashicorp/google 4.52.0

## License
This module is licensed under the MIT License. See the LICENSE file for details.
