
# NAT Instance Terraform Module

This Terraform stack deploys a cost-optimized **NAT Instance** (EC2) to provide internet access for resources in private subnets (such as EKS nodes).

## 💰 Cost: ~$4-15/month (vs $32-50 for NAT Gateway)

## Architecture

```
Internet
    ↓
Internet Gateway
    ↓
Public Subnet → [NAT Instance t3.nano + EIP]
    ↓
Private Subnets → [EKS Nodes]
```

## Features

- ✅ **Cost-effective**: t3.nano = $3.80/month
- ✅ **Auto-configuration**: user_data automatically configures NAT
- ✅ **Secure**: IMDSv2, encrypted root volume
- ✅ **Auto-discovery**: Discovers subnets/routes by tags
- ✅ **Amazon Linux 2023**: Latest stable AMI

## Quick Deployment

```bash
cd network/natins

# Option 1: With VPC scan
cd ../../scripts && python3 scan-account.py && cd ../network/natins

# Option 2: With VPC ID
echo 'vpc_id = "vpc-xxxxx"' > terraform.tfvars

terraform init
terraform plan
terraform apply
```

## Minimal Configuration

```hcl
vpc_id        = "vpc-xxxxx"
instance_type = "t3.nano"  # $3.80/month
```

## Recommended Instance Types

| Type      | vCPU | RAM    | Cost/month | Usage              |
|-----------|------|--------|------------|--------------------|
| t3.nano   | 2    | 0.5 GB | $3.80      | Light dev/test     |
| t3.micro  | 2    | 1 GB   | $7.59      | Standard dev/test  |
| t3.small  | 2    | 2 GB   | $15.18     | Staging            |
| t3.medium | 2    | 4 GB   | $30.37     | Light production   |

## ⚠️ Important for EKS

The NAT Instance allows your EKS nodes in private subnets to:
- ✅ Pull Docker images from ECR/DockerHub
- ✅ Access AWS APIs
- ✅ Download packages
- ✅ Communicate with outbound internet

## Limitations vs NAT Gateway

| Aspect        | NAT Instance | NAT Gateway      |
|---------------|--------------|------------------|
| Cost          | $4-15/month  | $32-50/month     |
| Bandwidth     | 5-25 Gbps    | 100 Gbps         |
| HA            | Manual       | Automatic        |
| Maintenance   | You          | AWS              |
| Setup         | 5 min        | 1 min            |

## Maintenance

```bash
# SSH to the instance (via bastion or SSM)
aws ssm start-session --target <instance-id>

# Check NAT
sudo iptables -t nat -L -n -v

# Logs
sudo journalctl -u iptables-restore
```

## Outputs

```hcl
nat_instance_id        = "i-xxxxx"
nat_instance_public_ip = "54.x.x.x"
security_group_id      = "sg-xxxxx"
```

## Destroy

```bash
terraform destroy
```
