# EKS Terraform Module (paas/eks)

This module provisions an AWS EKS cluster and all required resources for a production-ready Kubernetes control plane.

## Features
- Creates EKS cluster with managed control plane
- Provisions IAM roles for EKS and Karpenter (if enabled)
- Configures security groups, OIDC provider, and VPC integration
- Supports managed node groups or Karpenter autoscaling
- Outputs all key resource IDs and ARNs

## Prerequisites
- VPC and subnets must exist (see network stack)
- AWS CLI configured with appropriate credentials
- Terraform >= 1.0

## Usage

1. Copy and customize the example variables:
   ```sh
   cp terraform.tfvars.example terraform.tfvars
   # Edit terraform.tfvars to match your environment
   ```
2. Initialize and apply:
   ```sh
   terraform init
   terraform plan
   terraform apply
   ```

## Variables
- See `variables.tf` for all configurable options (cluster name, VPC IDs, subnet IDs, etc).
- Sensitive values (e.g., public IPs for access) should be set via variables, not hardcoded.

## Outputs
- See `outputs.tf` for all exported values (cluster endpoint, CA, security group IDs, IAM ARNs, etc).


## Karpenter Integration
- If you want to use Karpenter for autoscaling, set the relevant variables and deploy the Karpenter addon (see eks-bootstrap repo).

## Post-Install Configuration
- All post-install configuration (addons, manifests, GitOps, ArgoCD, etc.) is managed in the [eks-bootstrap GitHub repository](https://github.com/doulba/eks-bootstrap). After applying this module, refer to eks-bootstrap for deploying cluster addons and GitOps applications.

## Security
- Never commit real AWS credentials or sensitive data.
- Use example files for sharing configuration.

## Troubleshooting
- Ensure all required variables are set and VPC/subnets exist.
- Check AWS permissions for the deploying user.
- For OIDC/Karpenter, verify IAM and OIDC provider setup.

---
For more details, see comments in each .tf file or open an issue.
