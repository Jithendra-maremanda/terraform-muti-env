# Terraform Workspaces Project

## Description

This project demonstrates the use of Terraform workspaces to manage multiple environments (e.g., dev, prod) with a single set of Terraform configurations. It provisions AWS EC2 instances and associated security groups, tailoring resources based on the selected workspace.

## Prerequisites

*   **Terraform:** Ensure Terraform is installed on your local machine. You can download it from [terraform.io](https://www.terraform.io/downloads.html).
*   **AWS Account:** You need an AWS account and your AWS credentials configured locally (e.g., via `aws configure` or environment variables).
*   **S3 Bucket for Backend:** An S3 bucket named `roboshop-terraform-state` must exist in the `us-east-1` region for storing the Terraform state. This bucket should have versioning enabled and server-side encryption. The `provider.tf` specifies `use_lockfile = true` which is not a standard S3 backend attribute; locking is typically handled via a DynamoDB table.

## Usage

1.  **Navigate to Workspace Directory:**
    All Terraform commands should be run from the `workspaces` directory.
    ```bash
    cd workspaces
    ```

2.  **Initialize Terraform:**
    Initialize Terraform to download providers and configure the backend.
    ```bash
    terraform init
    ```

3.  **Create or Select a Workspace:**
    Terraform uses workspaces to isolate environments.
    *   To create a new workspace (e.g., `dev`):
        ```bash
        terraform workspace new dev
        ```
    *   To list existing workspaces:
        ```bash
        terraform workspace list
        ```
    *   To select an existing workspace:
        ```bash
        terraform workspace select <workspace_name>
        ```
    The `default` workspace exists initially. This project is best demonstrated by creating and using at least two workspaces, like `dev` and `prod`.

4.  **Apply Configuration:**
    Apply the configuration for the selected workspace.
    ```bash
    terraform apply
    ```
    Terraform will display a plan and prompt for confirmation. The instance type will change based on the workspace (e.g., `t3.micro` for `dev`, `t3.small` for `prod`).

5.  **Destroy Resources:**
    To remove resources managed by the current workspace:
    ```bash
    terraform destroy
    ```
    Confirm when prompted.

## Inputs

Variables are defined in `workspaces/variable.tf`:

*   `project`: (Default: `"roboshop"`) Project name, used in tagging.
*   `common_tags`: (Default: `{ Project = "roboshop", Terraform = "true" }`) Tags applied to all resources.
*   `sg_name`: (Default: `"allow-all"`) Base name for the security group.
*   `sg_description`: (Default: `"allowing all ports from all IP"`) Security group description.
*   `instances`: (Default: `["mongodb", "redis"]`) List of instance names. EC2 instances are named `<project>-<instance_name>-<workspace>`.
*   `from_port`: (Default: `0`) Start port for security group rules.
*   `to_port`: (Default: `0`) End port for security group rules (0 means all ports with `protocol = "-1"`).
*   `cidr_blocks`: (Default: `["0.0.0.0/0"]`) Allowed CIDR blocks for ingress.
*   `ami_id`: (Default: `"ami-09c813fb71547fc4f"`) AMI ID for EC2 instances (RHEL 9).
*   `instance_type`: (Default: `{ dev = "t3.micro", prod = "t3.small" }`) Map of instance types per workspace.

## Outputs

No explicit outputs are defined in `workspaces/ec2.tf`. Outputs can be added to display resource attributes (e.g., instance IPs) after `terraform apply`.

## Backend Configuration

The project uses an S3 backend for state management, configured in `workspaces/provider.tf`:

*   **Bucket:** `roboshop-terraform-state`
*   **Key:** `workspace-demo` (Workspace states are stored like `env:/<workspace_name>/<key>`)
*   **Region:** `us-east-1`
*   **Encryption:** Enabled
*   **`use_lockfile = true`**: This is noted in `provider.tf`. Standard S3 backends use DynamoDB for locking, specified via a `dynamodb_table` argument. If this argument is absent, Terraform's default locking mechanisms for S3 will apply, or it might indicate a custom setup.

This configuration ensures centralized and secure state management.