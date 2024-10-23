# ChatGPT Clone App Deployment on Kubernetes

## Overview

This project demonstrates the deployment of a ChatGPT clone app using a DevSecOps approach. We will leverage multiple DevOps tools and AWS cloud services for a robust and secure deployment.

---

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Setup Instructions](#setup-instructions)
3. [Kubernetes Cluster Creation using Terraform](#kubernetes-cluster-creation-using-terraform)
4. [Application Deployment on Kubernetes](#application-deployment-on-kubernetes)
5. [Monitoring via Prometheus and Grafana](#monitoring-via-prometheus-and-grafana)
6. [Cleanup: Destroying Infrastructure](#cleanup-destroying-infrastructure)
7. [Conclusion](#conclusion)

---

## Prerequisites

- An AWS account
- Basic knowledge of Terraform, Jenkins, and Kubernetes
- Open a terminal and create a directory for the project:
  ```bash
  mkdir gpt
  cd gpt
  git clone https://github.com/Aakibgithuber/Chat-gpt-deployment.git
  ```
---

## Setup Instructions
### Step 1: Setup Terraform and Configure AWS
1. **Install Terraform:**

```bash
sudo su
snap install terraform --classic
which terraform
```

2. **Configure AWS CLI:**

```bash
aws configure
```

- Enter your access key and secret key.

---

## Kubernetes Cluster Creation using Terraform
### Step 2: Create a New Jenkins Pipeline for EKS
1. **In Jenkins, create a new pipeline named eks-terraform.**
2. **Scroll down and select This project is parameterized.**
3. **Scroll down to the pipeline script and paste the following:**

```groovy
pipeline {
    agent any
    stages {
        stage('Checkout from Git') {
            steps {
                git branch: 'legacy', url: 'https://github.com/Aakibgithuber/Chat-gpt-deployment.git'
            }
        }
        stage('Terraform Version') {
            steps {
                sh 'terraform --version'
            }
        }
        stage('Terraform Init') {
            steps {
                dir('Eks-terraform') {
                    sh 'terraform init'
                }
            }
        }
        stage('Terraform Validate') {
            steps {
                dir('Eks-terraform') {
                    sh 'terraform validate'
                }
            }
        }
        stage('Terraform Plan') {
            steps {
                dir('Eks-terraform') {
                    sh 'terraform plan'
                }
            }
        }
        stage('Terraform Apply/Destroy') {
            steps {
                dir('Eks-terraform') {
                    sh 'terraform ${action} --auto-approve'
                }
            }
        }
    }
}
```

4. **Click on Apply and then Save.**
5. **Now click on Build with Parameters and then on Apply.**

---

### Step 3: Cluster Creation
- It takes about 10 to 15 minutes to create the cluster.
- Once completed, go to your AWS Console and search for EKS.
- Check the Node Groups and go to your EC2 instances—your instance should be ready.

---

## Application Deployment on Kubernetes
### Step 4: Deploy the Application
1. **Back to your EC2 instance, run the following command:**

``bash
aws eks update-kubeconfig --name <clustername> --region <region>
``

Example:

```bash
aws eks update-kubeconfig --name EKS_CLOUD --region us-east-1
```

2. **Clone the repository on your EC2 instance:**

```bash
git clone https://github.com/Aakibgithuber/Chat-gpt-deployment.git
```

3. **Navigate to the Kubernetes folder:**

```bash
cd Chat-gpt-deployment/k8s
```

4. **Deploy the application with the following commands:**

```bash
kubectl apply -f chatbot-ui.yaml
kubectl get all
```

5. **Copy the Load Balancer external IP and paste it in your browser.**

---

## Monitoring via Prometheus and Grafana
### Step 5: Set Up Monitoring
1. **Setup a Monitoring Server:**
- Launch a new EC2 instance with Ubuntu and t2.medium specs.
  
2. **Install Prometheus:**

```bash
sudo useradd --system --no-create-home --shell /bin/false prometheus
wget https://github.com/prometheus/prometheus/releases/download/v2.47.1/prometheus-2.47.1.linux-amd64.tar.gz
tar -xvf prometheus-2.47.1.linux-amd64.tar.gz
cd prometheus-2.47.1.linux-amd64/
sudo mkdir -p /data /etc/prometheus
sudo mv prometheus promtool /usr/local/bin/
sudo mv consoles/ console_libraries/ /etc/prometheus/
sudo mv prometheus.yml /etc/prometheus/prometheus.yml
```

3. **Set Ownership for Directories:**

```bash
sudo chown -R prometheus:prometheus /etc/prometheus/ /data/
```

4. **Create a Systemd Unit File:**

```bash
sudo nano /etc/systemd/system/prometheus.service
```

Add the following content:

```ini
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target
[Service]
User=prometheus
Group=prometheus
Type=simple
Restart=on-failure
ExecStart=/usr/local/bin/prometheus \
    --config.file=/etc/prometheus/prometheus.yml \
    --storage.tsdb.path=/data \
    --web.listen-address=0.0.0.0:9090
[Install]
WantedBy=multi-user.target
```

5. **Enable and Start Prometheus:**

```bash
sudo systemctl enable prometheus
sudo systemctl start prometheus
sudo systemctl status prometheus
```

6. **Open Port 9090 in the Security Group:**

- Access Prometheus at `http://<public_ip>:9090`.

7. **Install Node Exporter:**

- Follow similar steps as above to install Node Exporter.

8. **Configure Prometheus:**

- Edit `/etc/prometheus/prometheus.yml` to include Node Exporter and Jenkins as targets.

```yaml
scrape_configs:
  - job_name: 'node_exporter'
    static_configs:
      - targets: ['localhost:9100']
  - job_name: 'jenkins'
    static_configs:
      - targets: ['<your-jenkins-ip>:<your-jenkins-port>']
```

9. **Reload Prometheus Configuration:**

```bash
curl -X POST http://localhost:9090/-/reload
```

10. **Set Up Grafana:**

```bash
sudo apt-get update
sudo apt-get install -y apt-transport-https software-properties-common
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
sudo apt-get update
sudo apt-get -y install grafana
sudo systemctl enable grafana-server
sudo systemctl start grafana-server
```

11. **Open Port 3000 in the Security Group:**

- Access Grafana at `http://<public_ip>:3000`.
- Login with `admin/admin`.

12.** Add Prometheus as a Data Source in Grafana.**

13. **Import Dashboards for monitoring.**

---

## Cleanup: Destroying Infrastructure
### Step 6: Clean Up Resources

1. **Delete Kubernetes Deployment:**

```bash
kubectl delete deployment chatbot
```

2. **Destroy the EKS Cluster:**

- Go to your Jenkins, locate the eks-terraform pipeline, and select the destroy option.
- Click on Build.

3. **Delete the EC2 Instances:**

- After the EKS cluster deletion, delete the base instance (GPT instance) and monitoring instance.

```bash
terraform destroy --auto-approve
```

4, **Remove IAM Roles and Users from the IAM section.**

---

## Conclusion
You have successfully deployed and monitored the ChatGPT clone app on a Kubernetes cluster using a comprehensive CI/CD pipeline and monitoring system.



