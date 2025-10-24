# DevOps Assignment - ITA713

## Project Overview
Multi-server Django–PostgreSQL application deployment on AWS using Terraform, Ansible, Docker Swarm, and CI/CD automation.

## Project Structure

.
├── ansible/                    # Ansible playbooks for automation
│   ├── deploy_stack.yml       # Docker stack deployment playbook
│   ├── install_docker.yml     # Docker installation playbook
│   ├── inventory.ini          # Ansible inventory file
│   └── setup_swarm.yml        # Docker Swarm initialization playbook
│
├── .github/                   # CI/CD Configuration
│   └── workflows/
│       └── deploy.yml         # GitHub Actions workflow
│
├── django_app/               # Django Web Application
│   ├── manage.py            # Django management script
│   ├── requirements.txt     # Python dependencies
│   ├── accounts/            # Accounts application
│   │   ├── __init__.py
│   │   ├── models.py        # Login model
│   │   ├── views.py         # Login/Register/Home views
│   │   ├── urls.py          # URL routing
│   │   └── templates/       # HTML templates
│   │       ├── login.html
│   │       ├── register.html
│   │       └── home.html
│   └── config/              # Project configuration
│       ├── __init__.py
│       ├── settings.py      # Django settings
│       ├── urls.py          # Main URL configuration
│       └── wsgi.py
│
├── docker/                  # Docker Configuration
│   ├── docker-compose.yml  # Docker Compose for Swarm
│   └── Dockerfile.web      # Web application Dockerfile
│
├── scripts/                # Utility Scripts
│   └── bootstrap.sh       # Environment setup script
│
├── selenium/              # Automated Testing
│   └── test_app.py       # Selenium test scripts
│
└── terraform/            # Infrastructure as Code
    ├── main.tf          # Main Terraform configuration
    ├── outputs.tf       # Output definitions
    ├── terraform-key.pem # SSH key for instances
    └── terraform.tfstate # Terraform state file

Note: Some of these file weren't pushed due to secutiry concerns (ex- terraform-key.pem is not on the repo)

## Architecture

┌─────────────────┐     ┌─────────────────┐
│   GitHub Repo   │────>│ GitHub Actions  │
└────────┬────────┘     └────────┬────────┘
         │                       │
         │                       ▼
         │               ┌─────────────────┐
         │               │  Docker Swarm   │
         │               │    Cluster      │
         │               └────────┬────────┘
         │                       │
         ▼                       ▼
┌─────────────────┐     ┌─────────────────┐
│    Terraform    │────>│     Ansible     │
└────────┬────────┘     └────────┬────────┘
         │                       │
         │                       ▼
         │             ┌───────────────────┐
         │             │  Infrastructure   │
         └────────────>│   (AWS EC2)      │
                      └───────────────────┘

┌─── Application Stack ───┐
│ ┌─────────────────┐    │
│ │  Django Web App │    │
│ │   (2 replicas)  │    │
│ └────────┬────────┘    │
│          │             │
│ ┌────────▼────────┐    │
│ │   PostgreSQL    │    │
│ │   (1 replica)   │    │
│ └─────────────────┘    │
└─────────────────────────┘


### Deployment Flow:

1. *Infrastructure Provisioning*:
   - Terraform creates 4 EC2 instances (t2.micro)
   - Allocates Elastic IPs
   - Configures security groups and networking

2. *Configuration Management*:
   - Ansible installs Docker on all nodes
   - Initializes Docker Swarm (1 manager, 2 workers)
   - Configures overlay networking

3. *Application Deployment*:
   - Docker images built and pushed to Docker Hub
   - Stack deployed via Docker Swarm
   - Web service: 2 replicas across workers
   - Database: 1 replica on manager

4. *CI/CD Automation*:
   - GitHub Actions triggers on push
   - Automated testing with Selenium
   - Continuous deployment to production

## Infrastructure (EC2 Instances)

| Role | IP Address | Purpose |
|------|------------|---------|
| *Controller* | 54.123.45.67 | Terraform/Ansible/CI Runner |
| *Manager* | 52.198.76.54 | Docker Swarm Manager |
| *Worker A* | 35.177.88.123 | Docker Swarm Worker |
| *Worker B* | 18.202.45.198 | Docker Swarm Worker |

## Technology Stack

- *Infrastructure*: AWS EC2, Terraform
- *Configuration*: Ansible
- *Containerization*: Docker, Docker Swarm
- *Application*: Django 4.2, PostgreSQL 14
- *CI/CD*: GitHub Actions
- *Testing*: Selenium

## Quick Start

### Prerequisites:
- AWS Account with IAM credentials
- Terraform installed
- Ansible installed
- Docker installed locally
- Python 3.8+

### Bootstrap Everything:
bash
# Clone repository
git clone https://github.com/bhavesh230904/DevOps_Assignment.git
cd DevOps_Assignment

# Checkout your branch
git checkout ITA700

# Configure AWS credentials
aws configure

# Run bootstrap script
chmod +x scripts/bootstrap.sh
./scripts/bootstrap.sh


### Access Points:
- *Application*: http://52.71.195.227:8000
- *Login Credentials*: 
  - Username: ITA735
  - Password: 2022PE0540

## Manual Deployment Steps

### 1. Provision Infrastructure with Terraform:
bash
cd terraform
terraform init
terraform apply -auto-approve
terraform output


### 2. Configure Servers with Ansible:
bash
cd ../ansible

# Update inventory.ini with Terraform output IPs

# Install Docker
ansible-playbook -i inventory.ini install_docker.yml

# Initialize Swarm
ansible-playbook -i inventory.ini setup_swarm.yml

# Deploy Application
ansible-playbook -i inventory.ini deploy_stack.yml


### 3. Run Database Migrations:
bash
ssh -i terraform-key.pem ubuntu@52.71.195.227
docker ps  # Find web container ID
docker exec -it CONTAINER_ID python manage.py migrate
exit


## Application Features

### Login/Register System
- *Register*: Create account with Roll No (username) and Admission No (password)
- *Login*: Authenticate against PostgreSQL database
- *Home*: Displays personalized greeting with username
- *Logout*: Session management and cleanup

### Database Schema
sql
Table: login
- username (varchar, unique)
- password (varchar)


## CI/CD Pipeline

### GitHub Actions Workflow
yaml
Trigger: Push to ITA735 branch
Steps:
  1. Configure AWS credentials
  2. Run Terraform to provision infrastructure
  3. Install Ansible
  4. Execute Ansible playbooks
  5. Deploy Docker stack
  6. Run Selenium tests


## Docker Services

### Web Service (Django)
- *Image*: bhavesh230904/devops-web:latest
- *Replicas*: 2
- *Port*: 8000
- *Network*: Overlay network (app-network)

### Database Service (PostgreSQL)
- *Image*: postgres:14
- *Replicas*: 1
- *Port*: 5432
- *Volumes*: Persistent storage

## Testing

### Selenium Tests
bash
cd selenium
python test_app.py


Tests include:
- Registration workflow
- Login functionality
- Home page display
- Logout mechanism

## Monitoring & Troubleshooting

### Check Service Status:
bash
ssh -i terraform-key.pem ubuntu@52.71.195.227
docker service ls
docker service ps devops-app_web
docker service logs devops-app_web


### Check Swarm Nodes:
bash
docker node ls


### Scale Services:
bash
docker service scale devops-app_web=3


## Security Considerations

- SSH access via key-pair authentication
- Security groups restrict access to necessary ports only
- Database credentials stored in environment variables
- No sensitive data in repository

## Cleanup

### Destroy Infrastructure:
bash
cd terraform
terraform destroy -auto-approve


## Project Screenshots

### Registration Page
![Registration Page](https://github.com/aditya183749/DevOps_Assignment12/blob/ITA713/Screenshot%202025-10-24%20160227.png)

### Login Page
![Login Page](https://github.com/MaliMali15/DevOpsAssign12/blob/ITA735/login.png)

### Home Page
![Home Page](https://github.com/aditya183749/DevOps_Assignment12/blob/ITA713/Screenshot%202025-10-24%20155311.png)
