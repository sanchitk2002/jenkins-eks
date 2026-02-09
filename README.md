# 🚀 Jenkins + Terraform + Ansible Deployment on AWS EKS

This project demonstrates **CI/CD automation** using Jenkins, Terraform, and Ansible to deploy a **web server on AWS EC2** via Jenkins agents running on **EKS Kubernetes pods**.

---

## 🛠️ Tech Stack

- **Jenkins** – CI/CD automation
- **Terraform** – Infrastructure as Code (IaC)
- **Ansible** – Configuration Management
- **AWS EC2** – Virtual server for web hosting
- **AWS EKS** – Kubernetes cluster for Jenkins agents
- **GitHub** – Source code repository

---

## 📝 Project Overview

1. **Checkout code** from GitHub repository  
2. **Terraform stage**:  
   - Create EC2 instance and security group  
   - Capture public IP  
3. **Ansible stage**:  
   - Configure the EC2 instance (install Nginx)  
   - Use SSH key securely from Kubernetes secret  

---

## 🔑 Prerequisites

- AWS CLI configured with proper IAM permissions  
- Kubernetes cluster with Jenkins deployed  
- SSH key pair for EC2 access (private key stored locally and as a Kubernetes secret)  
- GitHub repository containing:  
  - `Jenkinsfile`  
  - `terraform/` folder (`main.tf`, `outputs.tf`)  
  - `ansible/nginx.yaml` playbook  

---

## 📂 File Structure

.
├── Jenkinsfile
├── terraform/
│ ├── main.tf
│ └── outputs.tf
├── ansible/
│ └── nginx.yaml
└── README.md


---

## ⚡ Step-by-Step Guide

### 1️⃣ Create SSH Key Secret in Kubernetes

```bash
# Ensure your private key is accessible
ls ~/jenkins.pem

# Create Kubernetes secret
kubectl create secret generic jenkins-ssh-key \
  --from-file=jenkins.pem=~/jenkins.pem \
  -n jenkins
```
```bash
# Copy SSH key to temporary path
cp /etc/jenkins-ssh-key/jenkins.pem /tmp/jenkins.pem
chmod 600 /tmp/jenkins.pem

# Disable host key checking for Ansible
export ANSIBLE_HOST_KEY_CHECKING=False

# Run Ansible playbook
ansible-playbook -i ansible/inventory ansible/nginx.yaml --private-key=/tmp/jenkins.pem -u ubuntu
```
---

## 💡 Tips & Best Practices

- **Use separate Terraform and Ansible containers** in Jenkins pods to keep concerns isolated.  
- **Store SSH keys securely** using Kubernetes secrets; never commit private keys to Git.  
- **Wait for SSH readiness** before running Ansible to avoid connection errors.  
- **Disable host key checking in Ansible** in CI/CD pipelines for ephemeral instances:  
  ```bash
  export ANSIBLE_HOST_KEY_CHECKING=False
