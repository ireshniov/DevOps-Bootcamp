******

<details>
<summary>Exercise 0: Prerequisites </summary>
 <br />

</details>

******

<details>
<summary>Exercise 1: Create a Terraform project to spin up EKS cluster </summary>
 <br />

`main.tf`
```terraform
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "5.40.0"
    }
  }
}

provider "aws" {
  region = var.region
}
```

`eks.tf`
```terraform
module "eks" {
  source  = "terraform-aws-modules/eks/aws"
  version = "20.11.0"

  cluster_name                   = var.cluster_name
  cluster_version                = "1.29"
  cluster_endpoint_public_access = true

  enable_cluster_creator_admin_permissions = true

  vpc_id = module.vpc.vpc_id
  // Subnets where worker nodes will run
  subnet_ids = module.vpc.private_subnets

  tags = {
    "environment" = "development"
    "application" = var.cluster_name
  }

  # starting from EKS 1.23 CSI plugin is needed for volume provisioning.
  cluster_addons = {
    aws-ebs-csi-driver = {}
  }

  // Configuration of the worker nodes
  eks_managed_node_groups = {
    // development environment
    dev = {
      min_size     = 2
      max_size     = 5
      desired_size = 3

      instance_types = ["t2.small"]

      # EBS CSI Driver policy
      iam_role_additional_policies = {
        AmazonEBSCSIDriverPolicy = "arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy"
      }
    }
  }

  fargate_profile_defaults = {}

  fargate_profiles = {
    dev = {
      name       = "app-dev-fargate-profile"
      subnet_ids = module.vpc.private_subnets
      selectors = [{
        namespace = "dev"
      }]
    }
  }
}
```

`vpc.tf`
```terraform
// https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources/availability_zones
data "aws_availability_zones" "azs" {
  state = "available"
}

module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "5.8.1"

  name = "${var.cluster_name}-vpc"
  cidr = var.vpc_cidr_block

  private_subnets = var.private_subnet_cidr_blocks
  public_subnets  = var.public_subnet_cidr_blocks
  azs             = data.aws_availability_zones.azs.names

  // All private subnets will route their internet traffic through this single NAT gateway
  enable_nat_gateway   = true
  single_nat_gateway   = true
  enable_dns_hostnames = true

  // these tags are here to help Cloud Control Manager to identify which VPC and Subnets it should connect to
  tags = {
    "kubernetes.io/cluster/${var.cluster_name}" = "shared"
    #     Name = "${var.cluster_name}-vpc"
  }

  public_subnet_tags = {
    // tag is required
    "kubernetes.io/cluster/${var.cluster_name}" = "shared"
    "kubernetes.io/role/elb" = 1
    Name                     = "${var.cluster_name}-public-subnet"
  }

  private_subnet_tags = {
    "kubernetes.io/cluster/${var.cluster_name}" = "shared"

    // private subnet is closed to internet
    // tag is required
    "kubernetes.io/role/internal-elb" = 1
    Name                              = "${var.cluster_name}-private-subnet"
  }
}
```

`variables.tf`
```terraform
variable "region" {}
variable "vpc_cidr_block" {}
variable "private_subnet_cidr_blocks" {}
variable "public_subnet_cidr_blocks" {}

variable "cluster_name" {}
```

`output.tf`
```terraform
output "cluster_endpoint" {
  description = "Endpoint for EKS control plane"
  value       = module.eks.cluster_endpoint
}

output "cluster_security_group_id" {
  description = "Security group ids attached to the cluster control plane"
  value       = module.eks.cluster_security_group_id
}

output "region" {
  description = "AWS region"
  value       = var.region
}

output "cluster_name" {
  description = "Kubernetes Cluster Name"
  value       = module.eks.cluster_name
}
```

Terraform plan
```shell
(base) ➜  terraform git:(main) ✗ terraform plan
Plan: 58 to add, 0 to change, 0 to destroy.

Changes to Outputs:
  + cluster_endpoint          = (known after apply)
  + cluster_name              = "terraform-exercises"
  + cluster_security_group_id = (known after apply)
  + region                    = "eu-central-1"
```

Terraform apply
```shell
(base) ➜  terraform git:(main) terraform apply

Apply complete! Resources: 58 added, 0 changed, 0 destroyed.

Outputs:

cluster_endpoint = "[cluster endpoint]"
cluster_name = "terraform-exercises"
cluster_security_group_id = "sg-0f6bbcf1160f517ae"
region = "eu-central-1"
```

Configure kubectl
```shell
(base) ➜  terraform git:(main) aws eks update-kubeconfig --profile [profile] --name terraform-exercises --region eu-central-1
```

Verify the Cluster
```shell
(base) ➜  terraform git:(main) ✗ kubectl cluster-info
Kubernetes control plane is running at [url]
CoreDNS is running at [url]

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
(base) ➜  terraform git:(main) ✗ kubectl get nodes
NAME                                          STATUS   ROLES    AGE     VERSION
ip-10-0-2-228.eu-central-1.compute.internal   Ready    <none>   2d17h   v1.29.8-eks-a737599
ip-10-0-2-250.eu-central-1.compute.internal   Ready    <none>   2d17h   v1.29.8-eks-a737599
ip-10-0-2-93.eu-central-1.compute.internal    Ready    <none>   2d17h   v1.29.8-eks-a737599
```
</details>

******

<details>
<summary>Exercise 2: Create Terraform project to spin up MySQL for data persistence </summary>
 <br />

`main.tf`
```terraform
terraform {
  required_providers {
    ...
    kubernetes = {
      source  = "hashicorp/kubernetes"
      version = "2.32.0"
    }
  }
}

# This gives back object with certificate-authority among other attributes: https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources/eks_cluster#attributes-reference
data "aws_eks_cluster" "cluster" {
    name       = module.eks.cluster_name
    depends_on = [module.eks.cluster_name]
}

provider "kubernetes" {
  host                   = data.aws_eks_cluster.cluster.endpoint
  cluster_ca_certificate = base64decode(data.aws_eks_cluster.cluster.certificate_authority.0.data)
  exec {
    api_version = "client.authentication.k8s.io/v1beta1"
    args = ["eks", "get-token", "--cluster-name", data.aws_eks_cluster.cluster.name]
    command     = "aws"
  }
}

provider "helm" {
  kubernetes {
    host                   = data.aws_eks_cluster.cluster.endpoint
    cluster_ca_certificate = base64decode(data.aws_eks_cluster.cluster.certificate_authority.0.data)
    exec {
      api_version = "client.authentication.k8s.io/v1beta1"
      args        = ["eks", "get-token", "--cluster-name", data.aws_eks_cluster.cluster.name]
      command     = "aws"
    }
  }
}
```

`mysql-tf`
```terraform
resource "helm_release" "mysql" {
  name       = "mysql"
  repository = "https://charts.bitnami.com/bitnami"
  chart      = "mysql"
  version = "11.1.17"
  timeout = "1000"

  values = [
    file("${path.module}/mysql-values.yaml")
  ]

  # Set chart values individually
  set {
    name  = "volumePermissions.enabled"
    value = true
  }

}
```
`mysq-values.yaml`
```yaml
architecture: replication

auth:
  rootPassword: secret
  database: terraform-exercises-app
  username: dev
  password: secret

volumePermissions:
  enabled: true

primary:
  persistence:
    enabled: true
    storageClass: gp2
    accessModes: ["ReadWriteOnce"]

secondary:
  # 1 primary and 2 secondary replicas
  replicaCount: 2
  persistence:
    enabled: true
    # Storage class for EKS volumes
    storageClass: gp2
    accessModes: ["ReadWriteOnce"]
```

Reinit terraform cause new dependencies
```shell
(base) ➜  terraform git:(main) ✗ terraform init -upgrade
```

Terraform plan && apply.
Check for updates
```shell
(base) ➜  terraform git:(main) kubectl get all
NAME                    READY   STATUS    RESTARTS   AGE
pod/mysql-primary-0     1/1     Running   0          18h
pod/mysql-secondary-0   1/1     Running   0          18h
pod/mysql-secondary-1   1/1     Running   0          18h

NAME                               TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/kubernetes                 ClusterIP   172.20.0.1      <none>        443/TCP    3d19h
service/mysql-primary              ClusterIP   172.20.27.75    <none>        3306/TCP   18h
service/mysql-primary-headless     ClusterIP   None            <none>        3306/TCP   18h
service/mysql-secondary            ClusterIP   172.20.196.48   <none>        3306/TCP   18h
service/mysql-secondary-headless   ClusterIP   None            <none>        3306/TCP   18h

NAME                               READY   AGE
statefulset.apps/mysql-primary     1/1     18h
statefulset.apps/mysql-secondary   2/2     18h

(base) ➜  terraform-exercises git:(master) ✗ eksctl get fargateprofile --cluster terraform-exercises        
NAME                    SELECTOR_NAMESPACE      SELECTOR_LABELS POD_EXECUTION_ROLE_ARN                                                                  SUBNETS                                              TAGS                                                     STATUS
app-dev-fargate-profile dev                     <none>          arn:aws:iam::173861077519:role/app-dev-fargate-profile-2024092621560489160000000f       subnet-041f18762ea5a03c5,subnet-0a38e9a79571edf42    application=terraform-exercises,environment=development  ACTIVE

```

</details>

******

<details>
<summary>Exercise 3: Configure remote state </summary>
 <br />

Create `terraform-exercises-tf` bucket in AWS

Add backend
```terraform
terraform {
  ...
  backend "s3" {
    bucket = "terraform-exercises-tf"
    key    = "state.tfstate"
    region = "eu-central-1"
  }
}
```

Reinit terraform

</details>

******

<details>
<summary>Exercise 4: CI/CD pipeline for Terraform project </summary>
 <br />

1. Configure jenkins + jenkins agent
Including installing terraform on agent

2. Configure multibranch pipeline `terraform` in jenkins
Add `jenkins_aws_access_key_id` and `jenkins_aws_secret_access_key` credentials in pipeline `terraform` credentials

3. Apply changes to terraform scripts

Use string variables instead arrray (cause hard to use array in Jenkinsfile)

`terraform.tfvars`
```terraform
region         = "eu-central-1"
vpc_cidr_block = "10.0.2.0/23"
private_subnet_cidr_blocks = "10.0.2.0/25,10.0.2.128/25" // = subset of VPC
public_subnet_cidr_blocks = "10.0.3.0/25,10.0.3.128/25" // = subset of VPC

cluster_name = "terraform-exercises"
```

`vpc.tf`
```terraform
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  ...
  private_subnets = split(",", var.private_subnet_cidr_blocks)
  public_subnets  = split(",", var.public_subnet_cidr_blocks)
```

Use aws_eks_cluster_auth resource because `exec` command requires aws cli installed
```terraform
data "aws_eks_cluster_auth" "cluster" {
  name       = module.eks.cluster_name
  depends_on = [module.eks.cluster_name]
}


provider "kubernetes" {
  host                   = data.aws_eks_cluster.cluster.endpoint
  token                  = data.aws_eks_cluster_auth.cluster.token
  cluster_ca_certificate = base64decode(data.aws_eks_cluster.cluster.certificate_authority.0.data)
}

provider "helm" {
  kubernetes {
    host                   = data.aws_eks_cluster.cluster.endpoint
    token                  = data.aws_eks_cluster_auth.cluster.token
    cluster_ca_certificate = base64decode(data.aws_eks_cluster.cluster.certificate_authority.0.data)
  }
}
```

4. Add Jenkinsfile
`Jenkinsfile`
```text
#!/usr/bin/env groovy

pipeline {   
    agent any

    environment {
        AWS_ACCESS_KEY_ID = credentials('jenkins_aws_access_key_id')
        AWS_SECRET_ACCESS_KEY = credentials('jenkins_aws_secret_access_key')
    }

    stages {
        stage("provision server") {
            environment {
                TF_VAR_region = 'eu-central-1'
                TF_VAR_vpc_cidr_block = '10.0.2.0/23'
                TF_VAR_private_subnet_cidr_blocks = '10.0.2.0/25,10.0.2.128/25'
                TF_VAR_public_subnet_cidr_blocks = '10.0.3.0/25,10.0.3.128/25'
                TF_VAR_cluster_name = 'terraform-exercises'
            }

            steps {
                script {
                    // The -migrate-state option will attempt to copy existing state to the new backend,
                    // and depending on what changed, may result in interactive prompts to confirm migration of workspace states.
                    // The -force-copy option suppresses these prompts and answers "yes" to the migration questions.
                    // Enabling -force-copy also automatically enables the -migrate-state option.
                    sh "terraform init -force-copy"
                    sh "terraform plan"
                    sh "terraform apply --auto-approve"
                }
            }
        }
    }
}
```
5. Build

</details>

******
