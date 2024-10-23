# ChatGPT Clone App Deployment on Kubernetes

## Overview

This project demonstrates the deployment of a ChatGPT clone app using a DevSecOps approach. We will leverage multiple DevOps tools and AWS cloud services for a robust and secure deployment.

---

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Setup Instructions](#setup-instructions)
3. [Steps for Deployment](#steps-for-deployment)
4. [CI/CD Pipeline](#CI/CD-pipeline)
5. [Kubernetes Cluster Creation using Terraform](#kubernetes-cluster-creation-using-terraform)
6. [Application Deployment on Kubernetes](#application-deployment-on-kubernetes)
7. [Monitoring via Prometheus and Grafana](#monitoring-via-prometheus-and-grafana)
8. [Cleanup: Destroying Infrastructure](#cleanup-destroying-infrastructure)
9. [Conclusion](#conclusion)

---

## Prerequisites

- An AWS account
- Basic knowledge of Terraform, Jenkins, and Kubernetes
- Open a terminal and create a directory for the project:
  ```bash
  mkdir gpt
  cd gpt
  git clone https://github.com/dahrihadri/Chat-gpt-deployment.git
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

2. **Configure AWS:**

- Create an IAM user:
  - Go to the IAM section of your AWS account.
  - Click on Users → Add user.
  - Provide a username and select AWS Management Console access.
  - Set a password and click Next.
  - Attach policies (e.g., AdministratorAccess).
  - Click Next and then Create User.
  - Download the CSV with your access credentials.

3. **Configure AWS CLI:**

```bash
aws configure
```

- Enter your access key and secret key from the CSV file.

---

## Steps for Deployment
### Step 2: Build Infrastructure with Terraform

- Execute the following commands:

```bash
terraform init
terraform validate
terraform plan
terraform apply --auto-approve
```

- This will create an EC2 instance configured to run Jenkins, Docker, and SonarQube.

### Step 3: Setup SonarQube and Jenkins

1. **SonarQube:**

  - Access SonarQube via `http://<public_ip>:9000`.
  - Default username/password: admin/admin.
  - Update your password.

2. **Jenkins:**

  - Access Jenkins via `http://<public_ip>:8080`.
  - Retrieve the initial admin password:

```bash
sudo su
cat /var/lib/jenkins/secrets/initialAdminPassword
```

  - Install suggested plugins and set up your Jenkins user.

---

## CI/CD Pipeline
### Step 4: Set Up Jenkins Pipeline

1. **Install Required Plugins:**

  - Eclipse Temurin Installer
  - SonarQube Scanner
  - NodeJS Plugin
  - OWASP Plugin

  > The OWASP Plugin in Jenkins is like a `security assistant` that helps you find and fix security issues in your software. It uses the knowledge and guidelines from the Open Web Application Security Project       > (OWASP) to scan your web applications and provide suggestions on how to make them more secure. It’s a tool to ensure that your web applications are protected against common security threats and vulnerabilities

  - Prometheus metrics
  - Docker-related plugins
  - Kubernetes plugins

2. **Add Credentials for SonarQube and Docker:**

  - Generate a token in SonarQube and add it to Jenkins credentials.
  - Add Docker Hub credentials.

3. **Configure Jenkins Tools:**

  - Add JDK, NodeJS, Docker, and Sonar Scanner through the Jenkins interface.

4. **Configure Global Settings for SonarQube:**

  - Add SonarQube server details in Jenkins global settings.
  - Set up webhooks for SonarQube.

5. **Run the Pipeline:**

  - Create a new pipeline job in Jenkins and use the following script:

```groovy
pipeline {
    agent any
    tools {
        jdk 'jdk17'
        nodejs 'node16'
    }
    stages {
        stage('Checkout from Git') {
            steps {
                git branch: 'legacy', url: 'https://github.com/Aakibgithuber/Chat-gpt-deployment.git'
            }
        }
        // Further stages for build, analysis, and deployment...
    }
}
```

---


## Kubernetes Cluster Creation using Terraform
### Step 5: Create a New Jenkins Pipeline for EKS
1. **In Jenkins, create a new pipeline named eks-terraform.**
2. **Scroll down and select This project is parameterized.**
3. **Scroll down to the pipeline script and paste the following:**

```groovy
pipeline {
    agent any
    stages {
        stage('Checkout from Git') {
            steps {
                git branch: 'legacy', url: 'https://github.com/dahrihadri/Chat-gpt-deployment.git'
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

### Step 5: Cluster Creation
- It takes about 10 to 15 minutes to create the cluster.
- Once completed, go to your AWS Console and search for EKS.
- Check the Node Groups and go to your EC2 instances—your instance should be ready.

---

## Application Deployment on Kubernetes
### Step 6: Deploy the Application
1. **Back to your EC2 instance, run the following command:**

```bash
aws eks update-kubeconfig --name <clustername> --region <region>
```
  - The command `aws eks update-kubeconfig --name EKS_CLOUD --region us-east-1` is like telling our computer, "Hey, I'm using Amazon EKS (Elastic Kubernetes Service) in the 'us-east-1' region, and I want to   connect to it." You could use your desired location.

Example:

```bash
aws eks update-kubeconfig --name EKS_CLOUD --region us-east-1
```

2. **Clone the repository on your EC2 instance:**

```bash
git clone https://github.com/dahrihadri/Chat-gpt-deployment.git
```

3. **Navigate to the Kubernetes folder:**

```bash
cd Chat-gpt-deployment/k8s
```

4. **Deploy the application with the following commands:**

  - You will find the deployment file for Kubernetes chatbot. Run the following command to deploy the application

  ```bash
  kubectl apply -f chatbot-ui.yaml
  kubectl get all
  ```

5. **Copy the Load Balancer external IP and paste it in your browser.**

6. **Now, again, the application is deployed on Kubernetes.**

**Load Balancer Ingress**

> The Load Balancer Ingress is a mechanism that helps distribute incoming internet traffic among multiple servers or services, ensuring efficient and reliable delivery of requests.

> It’s like having a receptionist at a busy office building entrance who guides visitors to different floors or departments, preventing overcrowding at any one location. In the digital world, a Load Balancer 
> Ingress helps maintain a smooth user experience, improves application performance, and ensures that no single server becomes overwhelmed with too much traffic.


**Service.yaml**

> The `service.yaml` file is like a set of rules that helps computers find and talk to each other within a software application. It's like a directory that says, "Hey, this is how you can reach different parts of > our application." It specifies how different parts of your application communicate and how other services or users can connect to them.

---

## Monitoring via Prometheus and Grafana

**Prometheus**

> is like a detective that constantly watches your software and gathers data about how it’s performing. It’s good at collecting metrics, like how fast your software is running or how many users are visiting your > website.

**Grafana**

> On the other hand, is like a dashboard designer. It takes all the data collected by Prometheus and turns it into easy-to-read charts and graphs. This helps you see how well your software is doing at a glance 
> and spot any problems quickly.

> In other words, `Prometheus` collects the information, and `Grafana` makes it look pretty and understandable so you can make decisions about your software. They’re often used together to monitor and manage 
> applications and infrastructure.

### Step 7: Set Up Monitoring
1. **Setup a Monitoring Server:**
- Go to the EC2 console and launch an instance having a base image of Ubuntu with t2.medium specs because Minimum Requirements to Install Prometheus:

  - 2 CPU cores.
  - 4 GB of memory.
  - 20 GB of free disk space.
  
2. **Install Prometheus:**
  - First, create a dedicated Linux user for Prometheus and download Prometheus:

  ```bash
  sudo useradd --system --no-create-home --shell /bin/false prometheus
  wget https://github.com/prometheus/prometheus/releases/download/v2.47.1/prometheus-2.47.1.linux-amd64.tar.gz
  ```

  - Extract Prometheus files, move them, and create directories:

  ```bash
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

- Add the following content:

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
Now go to the security group of your EC2 to enable port 9090 in which Prometheus will run.

  - Access Prometheus at `http://<public_ip>:9090`.

7. **Install Node Exporter:**

**Node Exporter**

> Node Exporter is like a “reporter” tool for Prometheus, which helps collect and provide information about a computer (node) so Prometheus can monitor it. It gathers data about things like CPU usage, memory, 
> disk space, and network activity on that computer.

**Node Port Exporter**

> A Node Port Exporter is a specific kind of Node Exporter that is used to collect information about network ports on a computer. It tells Prometheus which network ports are open and what kind of data is going in > and out of those ports. This information is useful for monitoring network-related activities and can help you ensure that your applications and services are running smoothly and securely.

- Run the following commands for installation:

a. Create a system user for Node Exporter and download Node Exporter:

```bash
sudo useradd --system --no-create-home --shell /bin/false node_exporter
wget https://github.com/prometheus/node_exporter/releases/download/v1.6.1/node_exporter-1.6.1.linux-amd64.tar.gz
```

b. Extract Node Exporter files, move the binary, and clean up:

```bash
tar -xvf node_exporter-1.6.1.linux-amd64.tar.gz
sudo mv node_exporter-1.6.1.linux-amd64/node_exporter /usr/local/bin/
rm -rf node_exporter*
```

c. Create a systemd unit configuration file for Node Exporter:

```bash
sudo nano /etc/systemd/system/node_exporter.service
```

d. Add the following code to the node_exporter.service file:

```ini
[Unit]
Description=Node Exporter
After=network.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=default.target
```

e. Enable and start Node Exporter:

```bash
sudo useradd -m -s /bin/bash node_exporter
sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter
sudo systemctl daemon-reload
sudo systemctl start node_exporter
sudo systemctl enable node_exporter
sudo system
```

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
### Step 8: Clean Up Resources

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



