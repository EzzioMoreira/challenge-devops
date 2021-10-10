# DevOps Challenge/SRE
![Diagram](https://github.com/EzzioMoreira/challenge-devops-sre/blob/master/img/diagram.JPG)

### Repositories used in this project:
- [Module Factory Floor](https://github.com/EzzioMoreira/module-factory-floor.git)
- [Deploy Module Factory Floor ](https://github.com/EzzioMoreira/deploy-module-factory-floor.git)
- [Module AWS ECS Fargate](https://github.com/EzzioMoreira/modulo-awsecs-fargate.git)
- [Deploy WS ECS Fargate](https://github.com/EzzioMoreira/deploy-modulo-awsecs-fargate-.git)

# Step 1.0: Deploy VPC infrastructure and ECS cluster.
### Requisites for running this project:
- Git
- Docker
- Docker-compose
- Make
- AWS CLI version 2

### 1.1: Configure credential for AWS.
Create ```.env``` file to AWS credentials with access key and secret key.
```shell
# AWS environment
AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
```
### 1.2: Create ECS cluster and network infrastructure.
Create a ```terraform.tf``` file with the following contents. If you prefer, change the environment variable name.
```terraform
provider "aws" {
  region  = "us-east-2"
  version = "= 3.0"
}
terraform {
  backend "s3" {
    bucket = "your-bucket-here"
    key    = "key-terraform-.tfstate"
    region = "us-east-2"
  }
}

module "dev_cluster" {
  source        = "git@github.com:EzzioMoreira/module-factory-floor.git?ref=v1.0"
  environment   = "development"
}
``` 

### 1.3: Run the following commands to deploy:
```make
make help:          ## Run make help.
terraform-init:     ## Run terraform init to download all necessary plugins
terraform-plan:     ## Exec a terraform plan and puts it on a file called plano
terraform-apply:    ## Uses plano to apply the changes on AWS.
terraform-destroy:  ## Destroy all resources created by the terraform file in this repo.
```

# Step 2.0: Deploy Container Image in AWS ECR:
### 2.1 Configure credential for AWS.
Create ```.env``` file to AWS credentials with access key and secret key.
```shell
# AWS environment
AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
```

### 2.2: Configure your variables in Makefile file.
- ```AWS_ACCOUNT```=you_account-id
- ```APP_IMAGE```=application_name
- ```AWS_REGIO```=you_aws_region

### 2.3: Run the following commands to deploy:
```make
make help:          ## Run make help.
ecr-build:          ## ECR-step:1 Build your Docker image.
ecr-login:          ## ECR-step:2 Retrieve an authentication token and authenticate your Docker client to your registry.
ecr-tag:            ## ECR-step:3 Tag your image so you can push the image to this repository.
ecr-push:           ## ECR-step:4 Push this image to your newly created AWS repository.
```

# Step 3.0: Deploy container in AWS ECS Fargate.
### Requisites for running this project:
- Git
- Docker
- Docker-compose
- Make
- AWS CLI version 2

### 3.1: Clone the following repository:
- [Deploy AWS ECS Fargate](https://github.com/EzzioMoreira/deploy-modulo-awsecs-fargate-.git)

### 3.2 Configure credential for AWS.
Create ```.env``` file to AWS credentials with access key and secret key.
```shell
# AWS environment
AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
```

### 3.3 Configure your variables in Makefile file.
- ```AWS_ACCOUNT```=you_account-id
- ```APP_IMAGE```=application_name
- ```AWS_REGIO```=you_aws_region

### 3.4 Create Service, Task Definition and Application Load Balance.
Create a ```terraform.tf``` file with the following contents. If you prefer, change the variables name.
```terraform
provider "aws" {
  region  = "us-east-2"
  version = "= 3.0"
}

terraform {
  backend "s3" {
    bucket = "your-bucket-here"
    key    = "key-terraform-.tfstate"
    region = "us-east-2"
  }
} 

module "app-deploy" {
  source                 = "git@github.com:EzzioMoreira/modulo-awsecs-fargate.git?ref=v1.3"
  containers_definitions = data.template_file.containers_definitions_json.rendered
  environment            = "development"
  app_name               = "website"
  app_port               = "80"
  fargate_version        = "1.4.0"
}

data "template_file" "containers_definitions_json" {
  template = file("./containers_definitions.json")

  vars = {
    APP_VERSION = var.APP_VERSION
    APP_IMAGE   = var.APP_IMAGE
  }
}

variable "APP_VERSION" {
    default   = "bead89c"
    describle = "Get the value of variable GIT_COMMIT in Makefile."
}

variable "APP_IMAGE" {
  default   = "website"
  describle = "Get the value of variable APP_IMAGE in Makefile"
}

variable "AWS_ACCOUNT" {
  default   = "520044189785"
  describle = "Get the value of variable AWS_ACCOUNT in Makefile"
}
```

### 3.5: Run the following commands to deploy:
```make
make help:          ## Run make help.
terraform-init:     ## Run terraform init to download all necessary plugins
terraform-plan:     ## Exec a terraform plan and puts it on a file called plano
terraform-apply:    ## Uses plano to apply the changes on AWS.
terraform-destroy:  ## Destroy all resources created by the terraform file in this repo.
```

# Step 4.0: Deploy container in your desktop.
### 4.1: Clone the following repository:
- [Deploy AWS ECS Fargate](https://github.com/EzzioMoreira/deploy-modulo-awsecs-fargate-.git)

### 4.2: Run the following commands to deploy container:
```make
make help:          ## Run make help.
docker-run-local:   ## Run the container on the local machine.
docker-stop-local:  ## Destroy the container on the local machine.