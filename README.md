# aws-github-actions-cicd-pipeline

# Project Overview

This project demonstrates a complete CI/CD (Continuous Integration and Continuous Deployment) workflow using GitHub Actions and AWS EC2.

Instead of manually copying website files and redeploying updates, the deployment process was fully automated using GitHub Actions workflows, SSH authentication, and secure GitHub repository secrets.

Whenever code is pushed to the GitHub repository, GitHub Actions automatically connects to an AWS EC2 instance and deploys the updated website files to the Nginx web server.

# Architecture Flow

Developer Pushes Code
        ↓
GitHub Repository
        ↓
GitHub Actions Workflow
        ↓
SSH Authentication
        ↓
Secure File Transfer (SCP)
        ↓
AWS EC2 Instance
        ↓
Nginx Web Server Updated Automatically

# AWS Services & Tools Used

* GitHub
* GitHub Actions
* AWS EC2
* Amazon VPC
* Security Groups
* Nginx
* SSH
* SCP
* YAML
* Git
* VS Code
* Terminal (zsh)

# Key Features

* Automated website deployment using GitHub Actions
* Secure SSH authentication with EC2
* Deployment triggered automatically on Git push
* Secure GitHub Secrets management
* Automated Nginx website updates
* CI/CD workflow using YAML pipelines
* Real-time deployment automation

# Project Structure

<img width="2877" height="556" alt="Broswer showing deployed website (1)" src="https://github.com/user-attachments/assets/cb30224b-e174-4700-9d7d-766b6e0bf4da" />

# Deployment Workflow

1. Created GitHub Repository

A GitHub repository was created to store the website files and GitHub Actions workflow configuration.

2. Created Website Files

A basic HTML landing page was developed for deployment testing.

3. Configured GitHub Actions Workflow

A workflow YAML file was created inside:

.github/workflows/deploy.yml

This workflow automatically triggers whenever code is pushed to the main branch.

4. Provisioned AWS EC2 Deployment Server

An Amazon Linux EC2 instance was launched with:

* Nginx installed
* SSH access enabled
* HTTP access enabled
* Public IP enabled

5. Configured SSH Authentication

A dedicated SSH key pair was generated and used for secure remote access between GitHub Actions and the EC2 instance.

6. Added GitHub Repository Secrets

Sensitive deployment credentials were securely stored using GitHub Secrets:

* HOST
* USERNAME
* SSH_KEY

7. Automated File Deployment

GitHub Actions automatically deploys updated website files to the EC2 instance whenever changes are pushed to the main branch. The workflow securely authenticates using GitHub Secrets and performs automated deployment using SSH and SCP.

# GitHub Actions Workflow
<img width="1440" height="900" alt="Screenshot 2026-05-16 at 2 29 50 AM" src="https://github.com/user-attachments/assets/3b4281a5-758d-4275-8329-10c4c8aa7565" />


YAML
name: Deploy Website

on:
  push:
    branches:
      - main

jobs:

  deploy:

    runs-on: ubuntu-latest

    steps:

      - name: Checkout Repository
        uses: actions/checkout@v4

      - name: Copy Files To EC2
        uses: appleboy/scp-action@v0.1.7

        with:
          host: ${{ secrets.HOST }}
          username: ${{ secrets.USERNAME }}
          key: ${{ secrets.SSH_KEY }}

          source: "index.html"

          target: "/home/ec2-user/site"

      - name: Deploy Website On EC2
        uses: appleboy/ssh-action@v1.0.3

        with:
          host: ${{ secrets.HOST }}
          username: ${{ secrets.USERNAME }}
          key: ${{ secrets.SSH_KEY }}

          script: |
            sudo cp /home/ec2-user/site/index.html /usr/share/nginx/html/index.html
            sudo systemctl restart nginx

# GitHub Actions automatically:

* connects to EC2 using SSH
* securely transfers website files using SCP
* updates the Nginx web directory
* restarts Nginx automatically

# Challenges Encountered & Fixes

Issue: GitHub rejected the initial push because the remote repository already contained files.
Cause: The local and remote repositories had unrelated commit histories.
Fix: Pulled the remote repository changes and merged histories before pushing successfully.

Issue: The .github/workflows directory was initially difficult to locate.
Cause: Folders beginning with . are hidden by default on macOS.
Fix: Created the directory manually using terminal commands:
mkdir -p .github/workflows

Issue: SSH returned "Permissions 0644 are too open"
Cause: The SSH private key file permissions were too permissive.
Fix: "chmod 400 week11-cicd-key.pem" to make the key read only and private enough for SSH to trust it.

Issue: GitHub Actions returned "ssh: no key found"
Cause: The SSH private key stored in GitHub Secrets was improperly formatted.
Fix: Re-added the full private key, including:
* BEGIN RSA PRIVATE KEY
* END RSA PRIVATE KEY

Issue: GitHub Actions failed with "dial tcp :22: i/o timeout"
Cause: The EC2 security group only allowed SSH access from the local machine IP.
Fix: Updated the EC2 security group inbound rules to temporarily allow: SSH (Port 22) → 0.0.0.0/0 for GitHub Actions connectivity during CI/CD testing.

# Lessons Learned
* SSH private key permissions matter. A key with permissions of 644 will be rejected — chmod 400 is not optional. Linux treats overly permissive key files as a security risk and refuses to use them.
* GitHub Actions runners are external machines. They are not your local computer. Any security group rule that restricts SSH access to your local IP will block GitHub Actions from reaching your EC2 instance entirely.
* GitHub Secrets must be stored exactly as they are. The full private key including the header and footer lines — BEGIN RSA PRIVATE KEY and END RSA PRIVATE KEY — must be present. A truncated or reformatted key will fail silently with a misleading error.
* Hidden directories require terminal commands to create. The .github/workflows directory starts with a dot, making it invisible in Finder. mkdir -p .github/workflows creates it correctly from the terminal. To show hidden folde(s) run ls -la
* CI/CD eliminates an entire category of human error. Manual SSH, manual SCP, manual Nginx restarts — every one of those steps is a place where something can go wrong. Automating the deployment pipeline removes those failure points entirely.
* Unrelated Git histories require a merge before pushing. When a local and remote repository have separate commit histories, GitHub will reject the push until the histories are reconciled.


# Final Outcome
<img width="1440" height="900" alt="Screenshot 2026-05-16 at 2 28 14 AM" src="https://github.com/user-attachments/assets/ec868ef6-4417-4137-bf8f-d7d527d09f60" />

A fully functional CI/CD pipeline was successfully implemented.

Website updates can now be deployed automatically by simply pushing code changes to the GitHub repository.

This eliminated the need for:

* manual SSH deployments
* manual SCP file transfers
* manual Nginx restarts

# Next Project
Docker Containerization. The next phase of the cloud engineering journey focuses on containerization using Docker.

Instead of deploying applications directly onto EC2 instances, applications will be packaged into portable containers that can run consistently across environments.

# Screenshots

<img width="1440" height="900" alt="Screenshot 2026-05-16 at 2 29 12 AM" src="https://github.com/user-attachments/assets/07b771a8-0319-4b63-a3ae-878e1b5d90e9" />

<img width="2880" height="1800" alt="Broswer showing deployed website (3)" src="https://github.com/user-attachments/assets/e9a47cda-470c-4eff-a3b6-bc8439d0577e" />
