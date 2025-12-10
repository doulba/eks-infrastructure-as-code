## Repository: `eks-infrastructure-as-code`

**Présentation**

**Démarrage rapide**
  - Terraform >= 1.6.0
  - AWS CLI ou identifiants AWS disponibles via profil ou variables d'environnement (`AWS_PROFILE` ou `AWS_ACCESS_KEY_ID`/`AWS_SECRET_ACCESS_KEY`).
  - `kubectl` pour interagir avec le cluster.

**Déploiement (validation locale — n'applique pas l'infra automatiquement)**
```bash
cd eks-infrastructure-as-code
cp envs/dev.tfvars.example envs/dev.tfvars   # adapter les valeurs (vpc/subnets, tags)
terraform init -backend=false
terraform fmt -recursive
terraform validate
terraform plan -var-file="envs/dev.tfvars"
```
## Repository: `eks-infrastructure-as-code`

Project overview
- Purpose: Terraform configuration to create an AWS EKS cluster along with the required IAM roles, security groups and supporting resources.
- Scope: this folder (`EKS/`) contains the EKS module implementation (resources, optional Karpenter IAM pieces, security groups, data sources, variables and helper scripts).

Quick start
- Prerequisites:
  - Terraform >= 1.6.0
  - AWS CLI or credentials available via environment/profile (`AWS_PROFILE` or `AWS_ACCESS_KEY_ID`/`AWS_SECRET_ACCESS_KEY`)
  - `kubectl` to interact with the cluster

Local validation (does not apply infrastructure):
```bash
cd eks-infrastructure-as-code
cp envs/DEV.tfvars.example envs/DEV.tfvars   # adapt values (vpc/subnets, tags)
terraform init -backend=false
terraform fmt -recursive
terraform validate
terraform plan -var-file="envs/DEV.tfvars"
```

### Important files & structure
- `providers.tf`: required providers and Terraform version.
- `variables.tf`: configurable variables for the module (region, `cluster_name`, node sizing, IAM policy ARNs, etc.).
- `envs/`: environment `tfvars` examples for `DEV`, `STG`, `PRD`.
- `eks_cluster.tf`: EKS cluster resource and cluster-level configuration.
- `iam_roles.tf`: IAM roles for EKS and optional Karpenter.
- `security_groups.tf`: security groups used by control plane, nodes and ALB.
- `data_sources.tf`: subnet lookups and `aws_eks_cluster_auth`.
- `locals.tf`: local values (parsing `vpc_scan_output/`).
- `outputs.tf`: outputs (endpoint, CA, SG ids, role ARNs, etc.).

### Variables of note
- `cluster_name`: cluster name used to prefix resources.
- `private_subnet_ids`, `public_subnet_ids`: if empty, `locals` will extract them from the `vpc_scan_output/` JSON.
- `vpc_id`: optional, used when provided instead of discovery.
- `create_karpenter`: whether to create Karpenter IAM role/instance profile.
- `create_ecr`: whether to create an ECR repository.

### vpc_scan_output/
- This folder contains a JSON snapshot used for local development to discover VPC/subnet/security group IDs. **Do not commit** this folder — it is ignored by `.gitignore`.
- If you prefer live lookups via AWS APIs replace the parsing logic in `locals.tf` with `data` resources (they require AWS credentials at plan time).

### Karpenter
- The repo includes optional IAM pieces for Karpenter. To have Karpenter provision nodes you must also configure an OIDC provider and deploy the Karpenter controller + a Provisioner.

### Outputs
- `eks_cluster_endpoint`, `cluster_ca_certificate`, `eks_control_plane_sg_id`, `eks_cluster_role_arn`, `karpenter_role_arn`, etc.

### Format, validate and troubleshooting
- `terraform fmt -recursive`
- `terraform init -backend=false`
- `terraform validate`
- If `terraform plan` fails with provider or auth errors, ensure AWS credentials and region/profile are set.

### Recommended next steps
- Add a remote backend (S3 + DynamoDB) for team collaboration before applying to production.
- Add managed `aws_eks_node_group` resources or complete the Karpenter setup to provision worker nodes.
- Add `scripts/generate-kubeconfig.sh` to build kubeconfig from outputs.
- Add a CI workflow (GitHub Actions) to run `terraform fmt`, `terraform validate`, and `terraform plan` on PRs.

### Environments (tfvars)
- Recommended: keep environment `tfvars` under `EKS/envs/` using `DEV`, `STG`, `PRD`.
- Commit only `*.tfvars.example` files. Never commit real IDs or secrets.

Usage:
```bash
cd eks-infrastructure-as-code
# Copy example and adapt for your environment
cp envs/DEV.tfvars.example envs/DEV.tfvars
# Edit envs/DEV.tfvars (vpc ids, subnets, tags, etc.)
terraform init -backend=false
terraform plan -var-file="envs/DEV.tfvars"
```

Best practices:
- Use `envs/` to separate configurations (DEV / STG / PRD).
- Add `envs/*.tfvars` to local `.gitignore` if you create files with real IDs or secrets.

### Contact / notes
- Tags: resources merge `local.common_tags` with `var.tags` when provided.
- I can add a root `README.md` that links to this module and/or create a GitHub Actions workflow skeleton to validate Terraform on PRs.
