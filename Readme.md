# EC2 Nginx Auto-Deploy

A static portfolio website hosted on an AWS EC2 instance, served by Nginx, with an automated CI/CD pipeline using GitHub Actions. Every push to `main` automatically deploys the latest code to the server.

---

## 🏗️ Architecture

```
Local Machine
      │
      └── git push
              │
              ▼
      GitHub Repository
              │
              └── triggers GitHub Actions
                          │
                          ▼
                  SSH into EC2
                          │
                          ▼
                    git pull
                          │
                          ▼
                /var/www/portfolio/
                          │
                          ▼
                        Nginx
                          │
                          ▼
              http://44.203.222.164
```

---

## 🚀 Features

- Static site hosted on AWS EC2 (Ubuntu 22.04)
- Nginx web server with custom error page
- Automated CI/CD via GitHub Actions on every push to `main`
- SSH-based deployment using stored private key secret

---

## 🛠️ Tech Stack

| Tool | Purpose |
|------|---------|
| AWS EC2 | Virtual Linux server |
| Ubuntu 22.04 | Server OS |
| Nginx | Web server |
| GitHub Actions | CI/CD pipeline |
| SSH | Secure deployment connection |

---

## ⚙️ Setup Guide

### 1. Launch EC2 Instance

- **AMI:** Ubuntu 22.04 LTS
- **Instance type:** t2.micro (free tier)
- **Key pair:** Create and download `.pem` file
- **Security Group inbound rules:**

| Type | Port | Source |
|------|------|--------|
| SSH | 22 | My IP |
| HTTP | 80 | 0.0.0.0/0 |

---

### 2. Connect to EC2 and Install Nginx

```bash
sudo apt update && sudo apt install nginx -y
sudo systemctl start nginx && sudo systemctl enable nginx
```

---

### 3. Configure Website Directory

```bash
sudo mkdir -p /var/www/portfolio
sudo chown -R ubuntu:ubuntu /var/www/portfolio
```

---

### 4. Configure Nginx

Create `/etc/nginx/sites-available/portfolio`:

```nginx
server {
    listen 80;
    server_name YOUR_EC2_IP;

    root /var/www/portfolio;
    index index.html;

    error_page 404 500 502 503 504 /error.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

Enable and reload:

```bash
sudo ln -s /etc/nginx/sites-available/portfolio /etc/nginx/sites-enabled/
sudo rm /etc/nginx/sites-enabled/default
sudo nginx -t
sudo systemctl reload nginx
```

---

### 5. Set Up Git on EC2

```bash
sudo apt install git -y
cd /var/www/portfolio
sudo git clone https://github.com/YOUR_USERNAME/ec2-nginx-auto-deploy.git .
sudo chown -R ubuntu:ubuntu /var/www/portfolio
```

---

### 6. Add GitHub Secrets

Go to repo → **Settings → Secrets and variables → Actions**

| Secret | Value |
|--------|-------|
| `EC2_HOST` | Your EC2 public IP |
| `EC2_USER` | `ubuntu` |
| `EC2_SSH_KEY` | Contents of your `.pem` file |

---

### 7. GitHub Actions Workflow

`.github/workflows/deploy.yml`:

```yaml
name: Deploy to EC2

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Deploy to EC2 via SSH
        uses: appleboy/ssh-action@v1.0.0
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ${{ secrets.EC2_USER }}
          key: ${{ secrets.EC2_SSH_KEY }}
          script: |
            cd /var/www/portfolio
            sudo git pull origin main || true
```

---

## 📁 Project Structure

```
├── index.html                  # Main portfolio page
├── error.html                  # Custom error page
├── .github/
│   └── workflows/
│       └── deploy.yml          # CI/CD pipeline
└── README.md
```

---

## 🔄 How to Deploy

Just push to main — Actions handles the rest:

```bash
git add .
git commit -m "your message"
git push
```

---

## 📊 Comparison vs S3 Static Hosting

| | S3 Hosting | EC2 + Nginx |
|--|------------|-------------|
| Server management | AWS managed | Self-managed |
| Web server | S3 built-in | Nginx |
| Deployment | `aws s3 sync` | SSH + `git pull` |
| OS access | None | Full Linux control |
| Scalability | Auto | Manual |

---

## 🔮 Possible Improvements

- [ ] Add CloudFront for HTTPS and CDN caching
- [ ] Connect custom domain via Route 53
- [ ] Provision entire infrastructure with Terraform
- [ ] Add test stage to GitHub Actions before deploy
- [ ] Set up SSL certificate with Let's Encrypt (Certbot)
