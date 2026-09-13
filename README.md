# EC2 Deployment with Terraform and Cloud-Init

This project demonstrates how to deploy and configure an AWS EC2 web server using **Terraform** and **cloud-init**.

Terraform creates the AWS infrastructure, while cloud-init automatically configures the EC2 instance and installs NGINX when the instance starts.

## Project Objective

The goal of this project is to:

* Create AWS infrastructure using Terraform
* Deploy an Ubuntu EC2 instance
* Create a VPC and public subnet
* Configure internet access using an Internet Gateway and route table
* Allow HTTP traffic using a Security Group
* Use Terraform `user_data` to pass a cloud-init configuration to EC2
* Automatically install and start NGINX
* Verify the web server through a browser

## Architecture

```text
                    Internet
                       |
                       v
              Internet Gateway
                       |
                       v
                    VPC
                10.0.0.0/16
                       |
                       v
              Public Subnet
                10.0.1.0/24
                       |
                       v
                  EC2 Instance
                       |
                  cloud-init
                       |
                       v
                    NGINX
                       |
                    HTTP :80
```

## Project Structure

```text
ec2-cloud-init/
|
├── main.tf
├── variables.tf
├── outputs.tf
├── cloud-init.yaml
├── .gitignore
├── .terraform.lock.hcl
├── README.md
|
```

### File Descriptions

| [`main.tf`](main.tf) | Defines the AWS infrastructure |
| [`variables.tf`](variables.tf) | Defines Terraform input variables |
| [`outputs.tf`](outputs.tf) | Outputs useful information such as the EC2 public IP |
| [`cloud-init.yaml`](cloud-init.yaml) | Installs and starts NGINX |
| [`.gitignore`](.gitignore) | Prevents local Terraform files from being committed |
| [`.terraform.lock.hcl`](.terraform.lock.hcl) | Locks the Terraform provider version |



## How It Works

Terraform first creates the AWS infrastructure:

1. VPC
2. Public subnet
3. Internet Gateway
4. Route table
5. Route table association
6. Security Group
7. EC2 instance

The EC2 instance uses Terraform's `user_data`:

```hcl
user_data = file("${path.module}/cloud-init.yaml")
```

This passes the `cloud-init.yaml` file to the EC2 instance during launch.

The cloud-init configuration installs and starts NGINX:

```yaml
#cloud-config

package_update: true

packages:
  - nginx

runcmd:
  - systemctl enable nginx
  - systemctl start nginx
```

This means NGINX is installed and started automatically when the EC2 instance boots.

## Deployment

Initialize Terraform:

```bash
terraform init
```

Validate the configuration:

```bash
terraform validate
```

Review the infrastructure changes:

```bash
terraform plan
```

Deploy the infrastructure:

```bash
terraform apply
```

Get the EC2 public IP:

```bash
terraform output
```

Open the public IP in a browser:

```text
http://<PUBLIC_IP>
```

The NGINX welcome page should be displayed.

## Result

The EC2 instance was successfully created and NGINX was automatically installed and started using cloud-init.

### NGINX 

<img width="1424" height="780" alt="cloud-init-nginx" src="https://github.com/user-attachments/assets/71705dcd-dba0-4ea1-b95a-df9081db52bf" />


## Troubleshooting

During deployment, the EC2 instance initially failed because the Security Group and subnet belonged to different VPCs.

The Security Group was originally created in the default VPC, while the EC2 instance was being created in the new custom VPC.

The issue was resolved by associating the Security Group with the custom VPC:

```hcl
vpc_id = aws_vpc.main.id
```

This was a useful reminder that AWS networking resources such as subnets and Security Groups must belong to the appropriate VPC.

## Destroying Resources

To remove the AWS resources created by Terraform:

```bash
terraform destroy
```

## What I Learned

Through this project I practiced:

* Terraform AWS provider configuration
* Terraform resources and variables
* EC2 deployment
* VPC and subnet configuration
* Internet Gateway and route tables
* Security Groups
* Terraform `user_data`
* Cloud-init
* Automated NGINX installation
* Terraform `plan`, `apply` and `destroy`
* Troubleshooting AWS networking issues
* Managing Terraform projects with Git and GitHub
