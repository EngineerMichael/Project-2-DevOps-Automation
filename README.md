# Project-2-DevOps-Automation
A collection of scripts and configurations to automate CI/CD pipelines, infrastructure provisioning, and monitoring.
Sure! Below is the source code for a **DevOps Automation** project that demonstrates how to automate CI/CD pipelines, infrastructure provisioning, and monitoring. This project will use the following tools:

1. **CI/CD Pipeline**: GitHub Actions (or any other CI tool like Jenkins or GitLab CI) for automating build, test, and deployment processes.
2. **Infrastructure Provisioning**: Terraform to automate the creation of cloud infrastructure (AWS example).
3. **Monitoring**: Prometheus and Grafana for monitoring infrastructure and applications.

### **Project Structure Overview**

```plaintext
devops-automation/
│
├── .github/
│   └── workflows/
│       └── ci-cd-pipeline.yml          # GitHub Actions CI/CD pipeline configuration
│
├── terraform/
│   ├── main.tf                         # Terraform configuration to provision infrastructure
│   ├── variables.tf                    # Variables for Terraform
│   ├── outputs.tf                      # Terraform outputs
│   └── provider.tf                     # Cloud provider configuration (AWS in this case)
│
├── monitoring/
│   ├── prometheus.yml                  # Prometheus configuration for monitoring
│   ├── docker-compose.yml              # Docker Compose file to spin up Prometheus and Grafana
│   └── grafana-dashboards.json         # Example Grafana dashboards
│
├── scripts/
│   ├── deploy.sh                       # Deployment script for the application
│   └── monitor.sh                      # Script to check system health
│
└── README.md                           # Project documentation
```

### **Step 1: CI/CD Pipeline using GitHub Actions**

1. **GitHub Actions Workflow (`.github/workflows/ci-cd-pipeline.yml`)**

This file automates the CI/CD process by defining the steps required for building, testing, and deploying the application whenever code is pushed to the repository.

```yaml
name: CI/CD Pipeline

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2
      
      - name: Set up Node.js
        uses: actions/setup-node@v2
        with:
          node-version: '14'

      - name: Install dependencies
        run: npm install

      - name: Run tests
        run: npm test

  deploy:
    runs-on: ubuntu-latest
    needs: build-and-test
    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Setup SSH
        uses: webfactory/ssh-agent@v0.5.3
        with:
          ssh-private-key: ${{ secrets.SSH_PRIVATE_KEY }}

      - name: Deploy to server
        run: |
          ssh -o StrictHostKeyChecking=no user@your-server-ip 'bash -s' < ./scripts/deploy.sh
```

- This workflow automatically runs when changes are pushed to the `main` branch or when a pull request is created against `main`.
- The `build-and-test` job installs dependencies and runs the tests.
- The `deploy` job deploys the application to the server after tests pass.

### **Step 2: Infrastructure Provisioning with Terraform**

Terraform is used to automate the provisioning of infrastructure. In this example, we will provision an EC2 instance on AWS.

1. **Terraform Configuration (`terraform/main.tf`)**

```hcl
provider "aws" {
  region = var.aws_region
}

resource "aws_instance" "example" {
  ami           = "ami-0c55b159cbfafe1f0"  # Use a valid Ubuntu AMI for your region
  instance_type = "t2.micro"
  
  tags = {
    Name = "DevOpsAutomationInstance"
  }
}

output "instance_ip" {
  value = aws_instance.example.public_ip
}
```

- This configuration defines an EC2 instance using an Ubuntu AMI.
- The `output` block outputs the public IP of the created EC2 instance.

2. **Variables (`terraform/variables.tf`)**

```hcl
variable "aws_region" {
  description = "AWS region to provision resources"
  default     = "us-west-2"
}
```

- The `aws_region` variable allows you to specify the AWS region for provisioning resources.

3. **Provider Configuration (`terraform/provider.tf`)**

```hcl
provider "aws" {
  region = "us-west-2"  # specify your region here
}
```

- This file configures the AWS provider.

4. **Apply Terraform Configuration**

To apply the configuration, use the following commands:

```bash
# Initialize Terraform
terraform init

# Apply Terraform configuration to provision resources
terraform apply
```

This will create the infrastructure on AWS (an EC2 instance, in this case).

### **Step 3: Monitoring with Prometheus and Grafana**

We will use **Prometheus** for monitoring metrics and **Grafana** for visualizing those metrics.

1. **Prometheus Configuration (`monitoring/prometheus.yml`)**

This file configures Prometheus to scrape metrics from your application.

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'nodejs-app'
    static_configs:
      - targets: ['localhost:8080']  # Replace with your application's metrics endpoint
```

2. **Docker Compose for Prometheus and Grafana (`monitoring/docker-compose.yml`)**

This file defines the Docker containers for Prometheus and Grafana.

```yaml
version: '3'

services:
  prometheus:
    image: prom/prometheus
    container_name: prometheus
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    ports:
      - "9090:9090"

  grafana:
    image: grafana/grafana
    container_name: grafana
    environment:
      GF_SECURITY_ADMIN_PASSWORD: "admin"
    ports:
      - "3000:3000"
    depends_on:
      - prometheus
```

- Prometheus will scrape metrics from the application and store them.
- Grafana will be accessible on port 3000 for visualizing those metrics.
- Grafana's default username is `admin`, and the password is set as `admin`.

3. **Grafana Dashboards (`monitoring/grafana-dashboards.json`)**

Here’s an example of a basic Grafana dashboard JSON file.

```json
{
  "dashboard": {
    "title": "Node.js Application Metrics",
    "panels": [
      {
        "title": "CPU Usage",
        "type": "graph",
        "targets": [
          {
            "expr": "rate(process_cpu_seconds_total[5m])",
            "legendFormat": "CPU Usage"
          }
        ]
      }
    ]
  }
}
```

- This dashboard shows CPU usage from the `process_cpu_seconds_total` metric.

4. **Run Prometheus and Grafana with Docker Compose**

To start Prometheus and Grafana, use Docker Compose:

```bash
cd monitoring
docker-compose up -d
```

- Access **Prometheus** at `http://localhost:9090`.
- Access **Grafana** at `http://localhost:3000` (login with `admin`/`admin`).

---

### **Step 4: Deployment Script**

1. **Deployment Script (`scripts/deploy.sh`)**

This script will be used in the CI/CD pipeline to deploy the application to the server.

```bash
#!/bin/bash

# SSH into the server and pull the latest changes from GitHub
ssh user@your-server-ip << 'EOF'
  cd /path/to/your/app
  git pull origin main
  npm install
  pm2 restart app
EOF
```

- This script SSHs into the server, pulls the latest code from the repository, installs dependencies, and restarts the application.

2. **Health Monitoring Script (`scripts/monitor.sh`)**

This script checks the health of the deployed application.

```bash
#!/bin/bash

# Check if the application is running
if curl -s http://localhost:8080/health | grep "OK"; then
  echo "Application is healthy"
else
  echo "Application is down"
  exit 1
fi
```

- This script pings the `/health` endpoint of the application to check if it's running and returns an appropriate status.

---

### **Step 5: Documentation (`README.md`)**

Provide documentation for setting up and running the project.

```markdown
# DevOps Automation

This project automates the process of CI/CD, infrastructure provisioning, and monitoring.

## Prerequisites

- Terraform
- Docker
- Prometheus and Grafana (via Docker Compose)
- AWS account (for infrastructure provisioning)
- Node.js application to deploy

## Steps

### 1. Set up Infrastructure
- Run `terraform apply` to provision the infrastructure on AWS.

### 2. CI/CD Pipeline
- GitHub Actions will automatically run on every push to the `main` branch to build, test, and deploy the application.

### 3. Monitoring
- Run `docker-compose up` in the `monitoring` directory to start Prometheus and Grafana for application monitoring.
- Access Prometheus at `http://localhost:9090` and Grafana at `http://localhost:3000`.

### 4. Deploying
- The deployment script (`scripts/deploy.sh`) is used in the CI/CD pipeline to deploy the latest code to the server.

## Conclusion

This project demonstrates automating the CI/CD process, provisioning cloud infrastructure, and monitoring the application using DevOps best practices.
```

---

### **Conclusion**

This **DevOps Automation** project provides a simple yet effective pipeline for automating infrastructure provisioning (with Terraform), continuous integration/deployment (with GitHub Actions), and monitoring (with Prometheus and Grafana). You can extend this project to deploy and monitor different types of applications, integrate with additional tools, or scale it to larger infrastructure.
