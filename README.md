# 🚀 Jenkins on EKS → Terraform → Ansible → NGINX

## End-to-End CI/CD Deployment on AWS (Step-by-Step Guide)

This project demonstrates a complete real-world DevOps CI/CD pipeline
where:

-   Jenkins runs on Amazon EKS
-   Jenkins dynamically launches Kubernetes agent pods
-   Terraform provisions an EC2 instance
-   Ansible installs NGINX on the EC2 instance via SSH
-   The NGINX application becomes publicly accessible

This README is written as a **hands-on guided tutorial**, not just a
command reference.

------------------------------------------------------------------------

# 🏗 Architecture Overview

Developer triggers Jenkins Pipeline →\
Jenkins (running on EKS) →\
Terraform Pod provisions EC2 →\
Ansible Pod configures EC2 →\
NGINX is deployed and accessible

------------------------------------------------------------------------

# 📋 Prerequisites

## 1️⃣ AWS Requirements

You must have:

-   AWS Account
-   IAM User with permissions for:
    -   EC2
    -   EKS
    -   VPC
    -   IAM

Verify credentials:

aws sts get-caller-identity

------------------------------------------------------------------------

## 2️⃣ Install Required Tools (Local Machine)

Ensure these are installed:

aws --version\
kubectl version --client\
terraform -version\
eksctl version\
helm version

Then configure AWS:

aws configure

------------------------------------------------------------------------

# 🚀 STEP 1: Create EKS Cluster

We create a Kubernetes cluster where Jenkins will run.

eksctl create cluster\
--name jenkins-eks\
--region ap-south-1\
--nodegroup-name workers\
--node-type t3.medium\
--nodes 2

Wait 10--15 minutes for cluster creation.

Verify:

kubectl get nodes

You should see worker nodes in Ready state.

------------------------------------------------------------------------

# 🚀 STEP 2: Install Jenkins on EKS

## Create Namespace

kubectl create namespace jenkins

## Install Jenkins Using Helm

helm repo add jenkins https://charts.jenkins.io\
helm repo update

helm install jenkins jenkins/jenkins -n jenkins

Verify installation:

kubectl get pods -n jenkins\
kubectl get svc -n jenkins

## 📸 Screenshot: Jenkins UI Pipeline Stages

Place image here:

![Jenkin-UI](github.com/sanchitk2002/jenkins-eks/blob/main/screenshots/jenkins-ui.png))

------------------------------------------------------------------------

# 🚀 STEP 3: Generate SSH Key for EC2

Terraform will attach this key to EC2.\
Ansible will use it for SSH.

ssh-keygen -t rsa -b 4096 -f jenkins.pem\
chmod 600 jenkins.pem

------------------------------------------------------------------------

# 🚀 STEP 4: Store SSH Key in Kubernetes Secret

kubectl create secret generic jenkins-ssh-key\
--from-file=jenkins.pem\
-n jenkins

Verify:

kubectl get secret -n jenkins

------------------------------------------------------------------------

# 🚀 STEP 5: Configure AWS Credentials in Jenkins

Go to:

Jenkins → Manage Jenkins → Credentials → Global

Add:

Kind: Username with password\
ID: aws-creds\
Username: AWS_ACCESS_KEY_ID\
Password: AWS_SECRET_ACCESS_KEY

------------------------------------------------------------------------

# 🚀 STEP 6: Terraform Provisions EC2

When the pipeline runs:

-   Terraform initializes
-   Creates Security Group
-   Creates EC2 instance
-   Outputs Public IP

## 📸 Screenshot: AWS Console (Security Group + EC2)

Place image here:

![Security Group]([https://github.com/sanchitk2002/jenkins-eks/blob/main/screenshots/sg.png))
![Security Group]([screenshots/instance.png))

------------------------------------------------------------------------

# 🚀 STEP 7: Ansible Installs NGINX

Pipeline waits for SSH (port 22).\
Then Ansible runs:

-   Updates apt cache
-   Installs nginx package

------------------------------------------------------------------------

# 🚀 STEP 8: Jenkins Pipeline Execution

Jenkins dynamically creates:

-   Terraform container
-   Ansible container

Pipeline Stages:

1.  Terraform Apply
2.  Run Ansible

![Terminal]([https://github.com/sanchitk2002/jenkins-eks/blob/main/screenshots/terminal.png))

------------------------------------------------------------------------

# ✅ Final Verification

Open in browser:

http://`<EC2_PUBLIC_IP>`{=html}

You should see the NGINX welcome page 🎉

![NGINX]([https://github.com/sanchitk2002/jenkins-eks/blob/main/screenshots/nginx.png))

------------------------------------------------------------------------

# 🔧 Troubleshooting

❌ No valid credential sources found\
→ Ensure AWS credentials are exported inside pipeline.

❌ InvalidClientTokenId\
→ Check IAM credentials.

❌ SSH Timeout\
→ Ensure EC2 is fully running and security group allows port 22.

------------------------------------------------------------------------

# 🧠 What You Learned

-   Running Jenkins on Kubernetes (EKS)
-   Dynamic Jenkins agent pods
-   Infrastructure as Code using Terraform
-   Configuration management using Ansible
-   Secure secret handling in Kubernetes
-   Real-world CI/CD debugging

------------------------------------------------------------------------

# 🚀 Future Improvements

-   Use IRSA instead of AWS access keys
-   Add Terraform destroy stage
-   Add Ingress + ALB
-   Add GitHub Webhooks
-   Store Terraform state in S3 + DynamoDB

------------------------------------------------------------------------

# 📁 Recommended Repository Structure

. ├── Jenkinsfile\
├── terraform/\
├── ansible/\
└── screenshots/\
├── sg-ec2.png\
├── terminal-output.png\
└── jenkins-stages.png

------------------------------------------------------------------------

# 📜 License

For educational and demonstration purposes.

Happy DevOps 🚀
