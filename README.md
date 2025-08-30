# gcp-terraform-foundation

A production-ready Terraform module to bootstrap a secure, well-architected Google Cloud Platform (GCP) project foundation. This module establishes essential guardrails and best practices by provisioning a core set of resources including a Service Project, a Shared VPC, and foundational IAM roles.

This is the first module that should be run in a new GCP environment to establish a strong multi-project architecture.


**Resources Provisioned:**

1.  **Host Project:**
    *   A new GCP project dedicated to shared network resources.
    *   A Shared VPC with predefined subnets in two regions.
    *   Baseline firewall rules (e.g., deny all ingress, allow internal traffic, allow SSH/ICMP via IAP).
    *   Pre-defined IAM roles granted to Google Groups for principle of least privilege.

2.  **Service Project:**
    *   A new GCP project for hosting application workloads (e.g., GKE, Compute Engine).
    *   Linked to the Host Project via Shared VPC.
    *   A dedicated service account for Compute Engine instances with minimal permissions.

## Features

*   **Secure by Design:** Implements a zero-trust network model with `deny-all` ingress firewall rules by default. Access is explicitly allowed via IAP for SSH/RDP.
*   **Least Privilege IAM:** Uses Google Groups for role assignments, making access management scalable and audit-friendly.
*   **Multi-Project Architecture:** Separates network (Host Project) from workloads (Service Project) for better isolation and cost management.
*   **Production Ready:** Includes essential configurations like a logging sink for audit logs.
*   **100% Infrastructure as Code:** Reproducible and version-controlled environment setup.

## Quick Start

### Prerequisites

1.  **A GCP Organization:** You must have an Organization set up in GCP.
2.  **Permissions:** You need the `roles/resourcemanager.projectCreator` and `roles/billing.user` roles at the organization or folder level.
3.  **Tools:** Terraform (`>= 1.0`), `gcloud` CLI, and a billing account ID.

### Usage

1.  **Clone the repository:**
    ```bash
    git clone https://github.com/<your-username>/gcp-terraform-foundation.git
    cd gcp-terraform-foundation
    ```

2.  **Create a `terraform.tfvars` file:**
    Modify the values to fit your environment.
    ```hcl
    # terraform.tfvars
    billing_account_id    = "123456-7890AB-CDEF12" # Your Billing Account ID
    org_id                = "123456789012"         # Your GCP Organization ID
    folder_id             = "environments"         # Optional: Target Folder ID

    host_project_id       = "my-terraform-host-prod"
    service_project_id    = "my-terraform-service-prod"

    network_name          = "foundation-vpc"
    subnets = [
      {
        subnet_name   = "subnet-us-central1"
        subnet_ip     = "10.0.1.0/24"
        subnet_region = "us-central1"
      },
      {
        subnet_name   = "subnet-europe-west1"
        subnet_ip     = "10.0.2.0/24"
        subnet_region = "europe-west1"
      }
    ]

    # Map your Google Groups to foundational roles
    group_network_admins  = "gcp-network-admins@your-domain.com"
    group_security_admins = "gcp-security-admins@your-domain.com"
    group_audit_viewers   = "gcp-audit-viewers@your-domain.com"
    ```

3.  **Initialize and apply Terraform:**
    ```bash
    terraform init
    terraform plan # Review the execution plan
    terraform apply
    ```
    Type `yes` when prompted to create the resources. This process can take 5-10 minutes.




## Testing

After a successful `terraform apply`, verify the resources:

1.  **Verify Projects:** Check the GCP Console to see the new Host and Service projects.
2.  **Verify VPC:** Navigate to **VPC network > VPC** in the Host Project. You should see the `foundation-vpc` with its subnets.
3.  **Verify IAM:** In the Host Project's IAM page, check that your groups are assigned the correct roles.

## Cleaning Up

**Warning:** This will delete all resources created by the module, including the projects and all data within them.

To destroy the entire foundation, run:
```bash
terraform destroy

