# 🚀 DevOps Mega Project — End-to-End CI/CD + GitOps

**Version:** 1.0.0 | **Updated:** 2026-09-14 | **Author:** Rohit Vishwakarma ([@Rohitz999](https://github.com/Rohitz999))

A **production-grade, fully automated** DevOps pipeline that takes code from Git commit to live HTTPS deployment — with **zero manual steps**.

**🌐 Live Demo:** [https://app.mechnomax.co.in](https://app.mechnomax.co.in)

---

## 📖 Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Prerequisites](#prerequisites)
- [Setup](#setup)
  - [Step 1 — Jenkins Server](#step-1--jenkins-server-)
  - [Step 2 — SonarQube Server](#step-2--sonarqube-server-)
  - [Step 3 — K3s + ArgoCD + Ingress + SSL](#step-3--k3s--argocd--ingress--ssl-)
  - [Step 4 — DNS + Nginx Reverse Proxy](#step-4--dns--nginx-reverse-proxy-)
  - [Step 5 — Gmail SMTP Setup](#step-5--gmail-smtp-setup-)
  - [Step 6 — Jenkins Credentials](#step-6--jenkins-credentials-)
  - [Step 7 — Create Jenkins Jobs](#step-7--create-jenkins-jobs-)
  - [Step 8 — Deploy ArgoCD Application](#step-8--deploy-argocd-application-)
- [Verification](#verification)
- [Usage](#usage)
- [Troubleshooting](#troubleshooting)
- [Appendix](#appendix)

---

## Overview

**What:** A fully automated CI/CD + GitOps pipeline for a Java Spring Boot application.

**Why:** Demonstrates the complete DevOps lifecycle — build → test → scan → containerize → deploy — with auto-sync to Kubernetes and HTTPS live in under 2 minutes per commit.

**Time to complete:** ~3–4 hours for first-time setup

**Difficulty:** Intermediate

**What you'll build:**

```
git push → Jenkins → SonarQube → Docker → Trivy
       → DockerHub → GitOps Repo → ArgoCD → K3s → HTTPS Live
```

**Technology stack:**

| Layer | Tool |
|-------|------|
| Source Control | GitHub |
| CI Server | Jenkins |
| Code Quality | SonarQube |
| Container Build | Docker |
| Security Scan | Trivy |
| Registry | DockerHub |
| GitOps | ArgoCD |
| Orchestration | K3s |
| Ingress | Nginx |
| SSL | cert-manager + Let's Encrypt |
| Notifications | Gmail SMTP |

---

## Architecture

```
     ┌──────────────────┐
     │   Developer      │
     │   git push       │
     └────────┬─────────┘
              │
              ▼
     ┌──────────────────┐
     │  GitHub          │
     │  App Repo        │
     └────────┬─────────┘
              │
              ▼
     ┌──────────────────────────────────────┐
     │  Jenkins Job 1: devops-cicd-app      │
     │  Build → Test → Sonar → Docker →    │
     │  Trivy → Push → Trigger Job 2 → Email│
     └────────┬─────────────────────────────┘
              │
              ▼
     ┌──────────────────┐
     │  DockerHub       │
     └────────┬─────────┘
              │
              ▼
     ┌──────────────────────────────────────┐
     │  Jenkins Job 2: devops-mega-gitops   │
     │  Update deployment.yaml → Push →     │
     │  Trigger ArgoCD → Email              │
     └────────┬─────────────────────────────┘
              │
              ▼
     ┌──────────────────┐
     │  GitHub          │
     │  GitOps Repo     │
     └────────┬─────────┘
              │
              ▼
     ┌──────────────────┐
     │  ArgoCD          │
     └────────┬─────────┘
              │
              ▼
     ┌──────────────────┐
     │  K3s Cluster     │
     └────────┬─────────┘
              │
              ▼
     ┌──────────────────┐
     │  https://app.    │
     │  mechnomax.co.in │
     └──────────────────┘
```

**Components:**

- **App Repo** — Java Spring Boot code + Dockerfile + Jenkinsfile
- **GitOps Repo** — Kubernetes manifests + ArgoCD config
- **Jenkins** — runs two pipelines (app build + GitOps update)
- **SonarQube** — static code analysis with quality gate
- **Trivy** — container vulnerability scanning
- **DockerHub** — image registry with unique tags per build
- **ArgoCD** — watches GitOps repo, syncs to K3s
- **K3s** — lightweight Kubernetes with auto-scaling
- **cert-manager** — automatic SSL via Let's Encrypt
- **Nginx Ingress** — routes traffic to the app

---

## Prerequisites

### Servers Required

| Server | Purpose | Specs |
|--------|---------|-------|
| **Jenkins** | CI/CD orchestration | 2 vCPU, 4 GB RAM, 20 GB disk |
| **SonarQube** | Code quality analysis | 2 vCPU, 4 GB RAM, 20 GB disk |
| **K3s** | Kubernetes cluster | 2 vCPU, 4 GB RAM, 20 GB disk |

**OS:** Ubuntu 22.04 LTS or newer

### Accounts Required

- **GitHub** — free account with 2 repos
- **DockerHub** — free account
- **Gmail** — with 2-Step Verification enabled

### DNS Records

Add these A records in your DNS provider:

| Domain | Points To |
|--------|-----------|
| `jenkins.mechnomax.co.in` | Jenkins public IP |
| `sonarqube.mechnomax.co.in` | SonarQube public IP |
| `argocd.mechnomax.co.in` | K3s public IP |
| `app.mechnomax.co.in` | K3s public IP |

### Ports to Open

| Port | Protocol | Purpose |
|------|----------|---------|
| 22 | TCP | SSH |
| 80 | TCP | HTTP (Let's Encrypt + redirect) |
| 443 | TCP | HTTPS |
| 8080 | TCP | Jenkins |
| 9000 | TCP | SonarQube |
| 6443 | TCP | K3s API |

### Repositories

| Repo | URL |
|------|-----|
| **App Repo** | `https://github.com/Rohitz999/devops-cicd-app` |
| **GitOps Repo** | `https://github.com/Rohitz999/devops-mega-gitops` |

**You don't need to write any code — just clone, configure, and run.**

---

## Setup

### Step 1 — Jenkins Server (⏱ 30 min)

**Goal:** Install Jenkins, Java, Docker, Maven + plugins.

#### 1.1 Update system

```bash
sudo apt update && sudo apt upgrade -y
```

#### 1.2 Install Java 17 (Temurin)

```bash
wget -O - https://packages.adoptium.net/artifactory/api/gpg/key/public | sudo tee /etc/apt/keyrings/adoptium.asc
echo "deb [signed-by=/etc/apt/keyrings/adoptium.asc] https://packages.adoptium.net/artifactory/deb $(awk -F= '/^VERSION_CODENAME/{print$2}' /etc/os-release) main" | sudo tee /etc/apt/sources.list.d/adoptium.list
sudo apt update
sudo apt install temurin-17-jdk -y
```

**Verify:**

```bash
java -version
# Expected: openjdk version "17.x.x"
```

#### 1.3 Install Jenkins

```bash
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/" | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt update
sudo apt install jenkins -y
sudo systemctl start jenkins
sudo systemctl enable jenkins
```

**Verify:**

```bash
sudo systemctl status jenkins
# Expected: active (running)
```

#### 1.4 Unlock Jenkins

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

**Then:**
1. Open `http://<jenkins-ip>:8080` in browser
2. Paste the password
3. Click **Install suggested plugins**
4. Create admin user

#### 1.5 Install Docker

```bash
sudo apt install -y ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins
```

**Verify:**

```bash
sudo -u jenkins docker ps
# Expected: empty list (no permission error)
```

#### 1.6 Install Maven

```bash
sudo apt install maven -y
mvn -version
```

#### 1.7 Create Trivy cache directories

```bash
sudo mkdir -p /var/lib/jenkins/.trivy-cache
sudo mkdir -p /var/lib/jenkins/.trivy-tmp
sudo chown -R jenkins:jenkins /var/lib/jenkins/.trivy-cache
sudo chown -R jenkins:jenkins /var/lib/jenkins/.trivy-tmp
```

#### 1.8 Install Jenkins Plugins

**Manage Jenkins → Plugins → Available** — install these:

- Maven Integration
- Pipeline Maven Integration
- Eclipse Temurin installer
- SonarQube Scanner
- Docker
- Docker Pipeline
- Docker Commons
- Docker API
- CloudBees Docker Build and Publish
- docker-build-step
- Email Extension

Restart Jenkins after install.

#### 1.9 Configure Tools

**Manage Jenkins → Tools:**

| Tool | Name | Setup |
|------|------|-------|
| JDK | `temurin-17` | Auto-install Adoptium Temurin 17 |
| Maven | `maven3` | Auto-install Maven 3.9.x |
| SonarQube Scanner | `sonarqube-scanner-latest` | Auto-install latest |

#### 1.10 Configure SonarQube Server in Jenkins

**Manage Jenkins → System → SonarQube servers:**

- ☑ **Environment variables**
- **Name:** `Sonarqube-scanner`
- **Server URL:** `https://sonarqube.mechnomax.co.in`
- **Server auth token:** `jenkins-sonarqube-token` (created later)

**⚠️ If this fails:**
- Can't access `http://<jenkins-ip>:8080` → check firewall/security group
- Jenkins won't start → `sudo journalctl -u jenkins -n 50` for logs

---

### Step 2 — SonarQube Server (⏱ 30 min)

**Goal:** Install PostgreSQL + SonarQube on a separate server.

#### 2.1 Update + Install PostgreSQL

```bash
sudo apt update && sudo apt upgrade -y
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
wget -qO- https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo tee /etc/apt/trusted.gpg.d/pgdg.asc &>/dev/null
sudo apt update
sudo apt-get -y install postgresql postgresql-contrib
sudo systemctl enable postgresql
sudo systemctl start postgresql
```

#### 2.2 Create SonarQube database

```bash
sudo passwd postgres
sudo su - postgres
createuser sonar
psql
```

Inside `psql`:
```sql
ALTER USER sonar WITH ENCRYPTED password 'sonar';
CREATE DATABASE sonarqube OWNER sonar;
GRANT ALL PRIVILEGES ON DATABASE sonarqube TO sonar;
\q
```

Then:
```bash
exit
```

#### 2.3 Install Java 17

```bash
sudo bash
wget -O - https://packages.adoptium.net/artifactory/api/gpg/key/public | tee /etc/apt/keyrings/adoptium.asc
echo "deb [signed-by=/etc/apt/keyrings/adoptium.asc] https://packages.adoptium.net/artifactory/deb $(awk -F= '/^VERSION_CODENAME/{print$2}' /etc/os-release) main" | tee /etc/apt/sources.list.d/adoptium.list
apt update
apt install temurin-17-jdk -y
update-alternatives --config java
exit
```

#### 2.4 Kernel tuning

```bash
sudo tee -a /etc/security/limits.conf << 'EOF'
sonarqube   -   nofile   65536
sonarqube   -   nproc    4096
EOF

sudo tee -a /etc/sysctl.conf << 'EOF'
vm.max_map_count = 262144
EOF

sudo reboot
```

#### 2.5 Install SonarQube

```bash
sudo wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-9.9.0.65466.zip
sudo apt install unzip -y
sudo unzip sonarqube-9.9.0.65466.zip -d /opt
sudo mv /opt/sonarqube-9.9.0.65466 /opt/sonarqube
sudo groupadd sonar
sudo useradd -c "user to run SonarQube" -d /opt/sonarqube -g sonar sonar
sudo chown sonar:sonar /opt/sonarqube -R
```

#### 2.6 Configure database

```bash
sudo vim /opt/sonarqube/conf/sonar.properties
```

Add:
```
sonar.jdbc.username=sonar
sonar.jdbc.password=sonar
sonar.jdbc.url=jdbc:postgresql://localhost:5432/sonarqube
```

#### 2.7 Create systemd service

```bash
sudo tee /etc/systemd/system/sonar.service > /dev/null << 'EOF'
[Unit]
Description=SonarQube service
After=syslog.target network.target

[Service]
Type=forking
ExecStart=/opt/sonarqube/bin/linux-x86-64/sonar.sh start
ExecStop=/opt/sonarqube/bin/linux-x86-64/sonar.sh stop
User=sonar
Group=sonar
Restart=always
LimitNOFILE=65536
LimitNPROC=4096

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl start sonar
sudo systemctl enable sonar
```

**Verify:**

```bash
sudo systemctl status sonar
# Expected: active (running)

# Watch logs
sudo tail -f /opt/sonarqube/logs/sonar.log
# Wait until you see "SonarQube is operational"
```

#### 2.8 Access SonarQube

Open: `http://<sonarqube-ip>:9000`

- **Default login:** `admin` / `admin`
- **Change password to:** `admin123`

#### 2.9 Generate token

- SonarQube → **My Account** → **Security** → **Generate Tokens**
- **Name:** `jenkins-token`
- **Type:** `User Token`
- Click **Generate** → **Copy the token** (`squ_xxxxx`)

#### 2.10 Configure webhook

- SonarQube → **Administration** → **Configuration** → **Webhooks**
- **Name:** `Jenkins`
- **URL:** `https://jenkins.mechnomax.co.in/sonarqube-webhook/`
- Click **Create**

**⚠️ If this fails:**
- SonarQube won't start → check `vm.max_map_count` is set, then reboot
- `Connection refused` on port 9000 → check firewall
- PostgreSQL auth error → verify `sonar.jdbc.password` in `sonar.properties`

---

### Step 3 — K3s + ArgoCD + Ingress + SSL (⏱ 45 min)

**Goal:** Install K3s, ArgoCD, cert-manager, and Nginx Ingress on one server.

#### 3.1 Install K3s

```bash
sudo bash
curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="server" sh -s - --disable traefik
exit
```

#### 3.2 Configure kubectl

```bash
mkdir -p ~/.kube
sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
sudo chown $USER:$USER ~/.kube/config
chmod 600 ~/.kube/config
export KUBECONFIG=~/.kube/config
echo 'export KUBECONFIG=~/.kube/config' >> ~/.bashrc
source ~/.bashrc
```

**Verify:**

```bash
kubectl get nodes
# Expected: ip-xxx  Ready  control-plane,master  xxm
```

#### 3.3 Install Helm

```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
helm version
```

#### 3.4 Install ArgoCD

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
kubectl get pods -n argocd -w
```

Wait for all pods to be `Running`. Press Ctrl+C to exit.

#### 3.5 Expose ArgoCD

```bash
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "NodePort"}}'
kubectl get svc argocd-server -n argocd
```

#### 3.6 Get admin password

```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
```

Save this password — it's for ArgoCD admin login.

#### 3.7 Install ArgoCD CLI

```bash
curl -sSL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
sudo install -m 555 argocd-linux-amd64 /usr/local/bin/argocd
rm argocd-linux-amd64
```

#### 3.8 Install cert-manager

```bash
helm repo add jetstack https://charts.jetstack.io
helm repo update
helm install cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --set installCRDs=true

kubectl get pods -n cert-manager
# Wait for all pods Running
```

#### 3.9 Create Let's Encrypt ClusterIssuer

```bash
kubectl apply -f - << 'EOF'
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    email: rohitvishwakarma8082@gmail.com
    server: https://acme-v02.api.letsencrypt.org/directory
    privateKeySecretRef:
      name: letsencrypt-prod-account-key
    solvers:
      - http01:
          ingress:
            class: nginx
EOF
```

**Verify:**

```bash
kubectl get clusterissuer
# Expected: letsencrypt-prod   READY: True
```

#### 3.10 Install Nginx Ingress

```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm install ingress-nginx ingress-nginx/ingress-nginx \
  --namespace ingress-nginx \
  --create-namespace \
  --set controller.hostNetwork=true \
  --set controller.dnsPolicy=ClusterFirstWithHostNet \
  --set controller.service.enabled=false

kubectl get pods -n ingress-nginx
# Wait for 1 pod Running
```

#### 3.11 Enable ArgoCD API token

```bash
kubectl patch configmap argocd-cm -n argocd --type merge -p '
data:
  accounts.admin: "apiKey, login"
'

kubectl rollout restart deployment argocd-server -n argocd
kubectl rollout status deployment argocd-server -n argocd
```

#### 3.12 Generate ArgoCD token

```bash
argocd login argocd.mechnomax.co.in --username admin --password <admin-pass> --insecure
argocd account generate-token --account admin
```

**Copy the JWT token** — you'll use it as `argocd-token` credential in Jenkins.

**⚠️ If this fails:**
- `argocd: command not found` → check `/usr/local/bin/argocd` exists
- `permission denied` on kubeconfig → `chmod 600 ~/.kube/config`
- Certificate not issuing → check port 80 open in firewall

---

### Step 4 — DNS + Nginx Reverse Proxy (⏱ 20 min)

#### 4.1 Add DNS A records

In your DNS provider for `mechnomax.co.in`:

| Type | Name | Value | TTL |
|------|------|-------|-----|
| A | jenkins | Jenkins IP | 300 |
| A | sonarqube | SonarQube IP | 300 |
| A | argocd | K3s IP | 300 |
| A | app | K3s IP | 300 |

**Verify:**

```bash
dig jenkins.mechnomax.co.in +short
# Should return your Jenkins IP
```

#### 4.2 Nginx reverse proxy for Jenkins

On **Jenkins server**:

```bash
sudo apt install nginx certbot python3-certbot-nginx -y

sudo tee /etc/nginx/sites-available/jenkins.conf > /dev/null << 'EOF'
upstream jenkins { server 127.0.0.1:8080; }

server {
    listen 80;
    server_name jenkins.mechnomax.co.in;

    access_log /var/log/nginx/jenkins.access.log;
    error_log /var/log/nginx/jenkins.error.log;

    proxy_buffers 16 64k;
    proxy_buffer_size 128k;

    location / {
        proxy_pass http://jenkins;
        proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504;
        proxy_redirect off;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto https;
    }
}
EOF

sudo ln -s /etc/nginx/sites-available/jenkins.conf /etc/nginx/sites-enabled/
sudo rm -f /etc/nginx/sites-enabled/default
sudo nginx -t
sudo systemctl restart nginx

# SSL
sudo certbot --nginx -d jenkins.mechnomax.co.in
```

#### 4.3 Nginx reverse proxy for SonarQube

On **SonarQube server**:

```bash
sudo apt install nginx certbot python3-certbot-nginx -y

sudo tee /etc/nginx/sites-available/sonarqube.conf > /dev/null << 'EOF'
server {
    listen 80;
    server_name sonarqube.mechnomax.co.in;

    access_log /var/log/nginx/sonar.access.log;
    error_log /var/log/nginx/sonar.error.log;

    proxy_buffers 16 64k;
    proxy_buffer_size 128k;

    location / {
        proxy_pass http://127.0.0.1:9000;
        proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504;
        proxy_redirect off;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto http;
    }
}
EOF

sudo ln -s /etc/nginx/sites-available/sonarqube.conf /etc/nginx/sites-enabled/
sudo rm -f /etc/nginx/sites-enabled/default
sudo nginx -t && sudo systemctl restart nginx

# SSL
sudo certbot --nginx -d sonarqube.mechnomax.co.in
```

**Verify:**

```bash
curl -I https://jenkins.mechnomax.co.in
# Expected: HTTP/2 200 (or 403 which means Jenkins is behind)

curl -I https://sonarqube.mechnomax.co.in
# Expected: HTTP/2 200
```

---

### Step 5 — Gmail SMTP Setup (⏱ 10 min)

**Goal:** Configure Jenkins to send email notifications.

#### 5.1 Enable 2-Step Verification on Gmail

1. Open https://myaccount.google.com/security
2. Enable **2-Step Verification** if not already on

#### 5.2 Generate App Password

1. Open https://myaccount.google.com/apppasswords
2. App name: `Jenkins`
3. Click **Create**
4. **Copy the 16-character password** (e.g., `abcd efgh ijkl mnop`)
5. Save it — you won't see it again

#### 5.3 Configure Jenkins Extended E-mail Notification

**Manage Jenkins → System → Extended E-mail Notification:**

| Field | Value |
|-------|-------|
| SMTP server | `smtp.gmail.com` |
| SMTP Port | `587` |
| Use SSL | ☐ (unchecked) |
| Use TLS | ☑ |
| SMTP Username | `rohitvishwakarma8082@gmail.com` |
| SMTP Password | `<16-char App Password>` |
| Default user e-mail suffix | `@gmail.com` |
| Default Recipients | `rohitvishwakarma8082@gmail.com` |

**Advanced:**
- Reply-To: `rohitvishwakarma8082@gmail.com`
- Charset: `UTF-8`

#### 5.4 Configure Basic E-mail Notification

**Manage Jenkins → System → E-mail Notification:**

| Field | Value |
|-------|-------|
| SMTP server | `smtp.gmail.com` |
| Default user e-mail suffix | `@gmail.com` |
| Use SMTP Authentication | ☑ |
| User Name | `rohitvishwakarma8082@gmail.com` |
| Password | `<16-char App Password>` |
| Use TLS | ☑ |

**System Admin e-mail address:** `rohitvishwakarma8082@gmail.com`

#### 5.5 Test SMTP

Click **Test configuration** → enter `rohitvishwakarma8082@gmail.com` → **Test**

**Check your inbox** (also check Spam folder).

**⚠️ Common errors:**
- `535-5.7.8 Username and Password not accepted` → use App Password, not Gmail password
- `534-5.7.9 Application-specific password required` → enable 2-Step Verification
- `Connection refused` → try port `465` with "Use SSL" checked

---

### Step 6 — Jenkins Credentials (⏱ 15 min)

**Manage Jenkins → Credentials → System → Global credentials → Add Credentials**

Create all **6 credentials** below:

#### 6.1 `github-creds`

| Field | Value |
|-------|-------|
| Kind | Username with password |
| Scope | Global |
| Username | `Rohitz999` |
| Password | `<GitHub PAT ghp_xxxx (scope: repo)>` |
| ID | `github-creds` |
| Description | GitHub PAT for clone + push |

**Create PAT:** https://github.com/settings/tokens → Generate new (classic) → scope: ✅ `repo`

#### 6.2 `dockerhub-creds`

| Field | Value |
|-------|-------|
| Kind | Username with password |
| Username | `rohitdockerhub01` |
| Password | `<DockerHub token dckr_pat_xxxx>` |
| ID | `dockerhub-creds` |

**Create token:** https://hub.docker.com/settings/security → New Access Token

#### 6.3 `jenkins-sonarqube-token`

| Field | Value |
|-------|-------|
| Kind | Secret text |
| Secret | `<SonarQube token squ_xxxx>` |
| ID | `jenkins-sonarqube-token` |

#### 6.4 `jenkins-api-token`

| Field | Value |
|-------|-------|
| Kind | Username with password |
| Username | `admin` |
| Password | `<Jenkins API token 11xxxx>` |
| ID | `jenkins-api-token` |

**Create API token:** Jenkins → top-right → username → Configure → API Token → Add new Token

#### 6.5 `argocd-token`

| Field | Value |
|-------|-------|
| Kind | Secret text |
| Secret | `<ArgoCD JWT token>` |
| ID | `argocd-token` |

#### 6.6 `sftp-creds`

| Field | Value |
|-------|-------|
| Kind | Username with password |
| Username | `rohitvishwakarma8082@gmail.com` |
| Password | `<Gmail App Password>` |
| ID | `sftp-creds` |
| Description | Gmail credential for SFTP/email |

---

### Step 7 — Create Jenkins Jobs (⏱ 15 min)

#### Job 1 — `devops-cicd-app`

1. **Jenkins → New Item**
2. Name: `devops-cicd-app`
3. Type: **Pipeline**
4. Configure:

| Field | Value |
|-------|-------|
| Discard old builds | ✅ Max: 5 |
| Definition | Pipeline script from SCM |
| SCM | Git |
| Repository URL | `https://github.com/Rohitz999/devops-cicd-app.git` |
| Credentials | `github-creds` |
| Branch | `*/main` |
| Script Path | `Jenkinsfile` |

5. **Save**

#### Job 2 — `devops-mega-gitops`

1. **Jenkins → New Item**
2. Name: `devops-mega-gitops`
3. Type: **Pipeline**
4. Configure:

| Field | Value |
|-------|-------|
| Discard old builds | ✅ Max: 5 |
| This project is parameterized | ✅ |
| String Parameter Name | `IMAGE_TAG` |
| Default Value | `latest` |
| Trigger builds remotely | ✅ Token: `gitops-token` |
| Definition | Pipeline script from SCM |
| SCM | Git |
| Repository URL | `https://github.com/Rohitz999/devops-mega-gitops.git` |
| Credentials | `github-creds` |
| Branch | `*/main` |
| Script Path | `Jenkinsfile` |

5. **Save**

---

### Step 8 — Deploy ArgoCD Application (⏱ 5 min)

On **K3s server**:

```bash
kubectl apply -f https://raw.githubusercontent.com/Rohitz999/devops-mega-gitops/main/argocd/application.yaml
```

**Verify:**

```bash
kubectl get applications -n argocd
# Expected: devops-mega-app   Synced   Healthy

kubectl get pods -l app=devops-mega-app
# Expected: 2 pods Running
```

**Enable auto-sync:**

```bash
kubectl patch application devops-mega-app -n argocd --type merge -p '
spec:
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
'
```

**Verify:**

```bash
kubectl get application devops-mega-app -n argocd -o jsonpath='{.spec.syncPolicy}'
# Expected: {"automated":{"prune":true,"selfHeal":true},...}
```

---

## Verification

### 1. DNS Check

```bash
dig app.mechnomax.co.in +short
# Should return K3s public IP
```

### 2. Certificate Check

```bash
kubectl get certificate
# Expected: app-mechnomax-tls   READY: True
```

### 3. Ingress Check

```bash
kubectl get ingress
# Expected: devops-mega-app-ingress with ADDRESS assigned
```

### 4. ArgoCD API Check

```bash
export ARGOCD_TOKEN="<your-token>"
curl -s -o /dev/null -w "%{http_code}\n" \
  -H "Authorization: Bearer $ARGOCD_TOKEN" \
  --insecure \
  "https://argocd.mechnomax.co.in/api/v1/applications"
# Expected: 200
```

### 5. Run the Pipeline

**Jenkins → `devops-cicd-app` → Build Now**

Watch the console output. Expected stages:

```
✅ Checkout
✅ Build
✅ Unit Tests
✅ SonarQube Analysis
✅ Quality Gate
✅ Build Docker Image       → 1.0.0-<N>
✅ Trivy Scan
✅ Push Docker Image
✅ Trigger GitOps Update    → HTTP 201
```

Then check `devops-mega-gitops` job runs, GitOps repo gets a commit, ArgoCD syncs, K3s updates, and the live app reflects the change.

### 6. Live App Check

```bash
curl https://app.mechnomax.co.in/health
# Expected: OK

curl https://app.mechnomax.co.in/version
# Expected: v1.0.0

curl https://app.mechnomax.co.in/api
# Expected: DevOps Mega Project - CI/CD Pipeline Working!
```

### 7. Email Check

Open `rohitvishwakarma8082@gmail.com` — you should have an email with subject:
```
✅ [SUCCESS] devops-cicd-app #<N>
```

---

## Usage

### Making a Change

```bash
git clone https://github.com/Rohitz999/devops-cicd-app.git
cd devops-cicd-app

# Edit any file
nano src/main/resources/static/index.html

# Commit and push
git add .
git commit -m "Update landing page"
git push origin main
```

**That's it.** The pipeline takes over automatically.

### What Happens Automatically

1. Jenkins detects the push
2. Builds new image `1.0.0-<build#>`
3. Runs tests + SonarQube + Trivy
4. Pushes image to DockerHub
5. Triggers GitOps pipeline
6. GitOps updates `deployment.yaml`
7. ArgoCD syncs to K3s
8. New pods start with new image
9. Live app updates
10. Email notification sent

**Total time:** ~2 minutes from push to live.

### Rolling Back

```bash
cd devops-mega-gitops
# Edit manifests/deployment.yaml — change image tag to previous version
nano manifests/deployment.yaml

git add .
git commit -m "Rollback to previous version"
git push origin main
```

ArgoCD syncs within 3 minutes → pods roll back.

### Checking Pipeline Status

- **Jenkins:** https://jenkins.mechnomax.co.in
- **ArgoCD:** https://argocd.mechnomax.co.in
- **DockerHub:** https://hub.docker.com/r/rohitdockerhub01/devops-mega-app/tags
- **Live App:** https://app.mechnomax.co.in

---

## Troubleshooting

### Pipeline Errors

| Error | Cause | Fix |
|-------|-------|-----|
| `400 buildWithParameters` | GitOps job missing `IMAGE_TAG` parameter | Add String Parameter in job config |
| `401 from GitHub` | Invalid PAT | Regenerate PAT with `repo` scope |
| `403 CSRF` | Trigger token not set | Add `gitops-token` in job Build Triggers |
| `No plugin found for prefix 'sonar'` | Missing sonar-maven-plugin | Add plugin to `pom.xml` |
| `no space left on device` | Trivy cache fills disk | Add `--skip-java-db-update` + cleanup |
| `cannot perform interactive login` | DockerHub creds wrong | Recreate `dockerhub-creds` as Username + Password |
| GitOps "Nothing to push" | Same tag reused | Use unique tags (`1.0.0-<build#>`) |

### ArgoCD Errors

| Error | Cause | Fix |
|-------|-------|-----|
| `OutOfSync` forever | Auto-sync off | Enable via UI or `kubectl patch` |
| Shows old revision | Polling delayed | Click REFRESH; or add webhook |
| `401 Unauthorized` on API | Token wrong | Regenerate `argocd-token` |
| `admin does not have apiKey capability` | Token generation blocked | Patch `argocd-cm` to enable `apiKey` |

### Kubernetes Errors

| Error | Cause | Fix |
|-------|-------|-----|
| Pods `ImagePullBackOff` | Image missing or private | Verify on DockerHub; set `imagePullPolicy: Always` |
| Pods `CrashLoopBackOff` | App failing to start | `kubectl logs <pod>` to see error |
| Certificate `False` | DNS or port 80 | Check `dig`, open port 80 in firewall |
| Ingress 404 | Wrong service/class | Verify `ingressClassName: nginx` |

### Email Errors

| Error | Cause | Fix |
|-------|-------|-----|
| `535-5.7.8 Username and Password not accepted` | Using Gmail password | Use App Password |
| `Connection refused` | Port 587 blocked | Try port 465 with SSL |
| No email received | In spam folder | Check spam; verify `to` field |

### Quick Diagnostic Commands

```bash
# Jenkins pipeline
# → See Jenkins console output

# ArgoCD app status
kubectl get application devops-mega-app -n argocd -o yaml

# K3s pod status
kubectl describe pod -l app=devops-mega-app

# K3s pod logs
kubectl logs -l app=devops-mega-app --tail=50

# Certificate issues
kubectl describe certificate app-mechnomax-tls
kubectl describe challenge -A

# Ingress issues
kubectl describe ingress devops-mega-app-ingress

# Force restart pods
kubectl rollout restart deployment devops-mega-app
```

---

## Appendix

### Quick Reference Card

```
┌────────────────────────────────────────────────────────────┐
│              DEVOPS MEGA PROJECT — CHEAT SHEET             │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  URLs:                                                     │
│    Jenkins    : https://jenkins.mechnomax.co.in            │
│    SonarQube  : https://sonarqube.mechnomax.co.in          │
│    ArgoCD     : https://argocd.mechnomax.co.in             │
│    Live App   : https://app.mechnomax.co.in                │
│                                                            │
│  Repositories:                                             │
│    App    : github.com/Rohitz999/devops-cicd-app           │
│    GitOps : github.com/Rohitz999/devops-mega-gitops        │
│                                                            │
│  Key Commands:                                             │
│    Restart Jenkins   : sudo systemctl restart jenkins      │
│    View K3s pods     : kubectl get pods -A                 │
│    Sync ArgoCD       : argocd app sync devops-mega-app     │
│    Force redeploy    : kubectl rollout restart deployment  │
│                        devops-mega-app                     │
│                                                            │
│  Jenkins Credentials (6):                                  │
│    github-creds            — GitHub PAT                    │
│    dockerhub-creds         — DockerHub token               │
│    jenkins-sonarqube-token — SonarQube token               │
│    jenkins-api-token       — Jenkins API token             │
│    argocd-token            — ArgoCD JWT                    │
│    sftp-creds              — Gmail App Password            │
│                                                            │
│  Jenkins Jobs (2):                                         │
│    devops-cicd-app     (main pipeline)                     │
│    devops-mega-gitops  (GitOps updater)                    │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

### All-in-One Setup Commands

```bash
# ══════════════════════════════════════════════════════════════
# JENKINS SERVER
# ══════════════════════════════════════════════════════════════
sudo apt update && sudo apt upgrade -y
sudo apt install temurin-17-jdk jenkins docker.io maven -y
sudo usermod -aG docker jenkins
sudo systemctl enable --now jenkins
sudo mkdir -p /var/lib/jenkins/.trivy-cache /var/lib/jenkins/.trivy-tmp
sudo chown -R jenkins:jenkins /var/lib/jenkins/.trivy-cache /var/lib/jenkins/.trivy-tmp

# ══════════════════════════════════════════════════════════════
# K3S SERVER
# ══════════════════════════════════════════════════════════════
curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="server" sh -s - --disable traefik
mkdir -p ~/.kube && sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
sudo chown $USER:$USER ~/.kube/config
export KUBECONFIG=~/.kube/config

# Install Helm + ArgoCD + cert-manager + Ingress
curl -fsSL https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
helm repo add jetstack https://charts.jetstack.io && helm repo update
helm install cert-manager jetstack/cert-manager --namespace cert-manager --create-namespace --set installCRDs=true
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm install ingress-nginx ingress-nginx/ingress-nginx -n ingress-nginx --create-namespace \
  --set controller.hostNetwork=true --set controller.dnsPolicy=ClusterFirstWithHostNet \
  --set controller.service.enabled=false
```

### Glossary

| Term | Meaning |
|------|---------|
| **CI/CD** | Continuous Integration / Continuous Deployment |
| **GitOps** | Using Git as single source of truth for infra + app state |
| **ArgoCD** | GitOps continuous delivery tool for Kubernetes |
| **K3s** | Lightweight Kubernetes distribution by Rancher |
| **Trivy** | Vulnerability scanner for containers |
| **SonarQube** | Code quality + security analysis platform |
| **cert-manager** | Kubernetes add-on for automatic TLS certificates |
| **ClusterIssuer** | cert-manager resource for issuing certificates cluster-wide |
| **Pod** | Smallest deployable unit in Kubernetes |
| **Ingress** | Kubernetes object that exposes HTTP/HTTPS routes |
| **PAT** | Personal Access Token (GitHub) |
| **JWT** | JSON Web Token (used by ArgoCD) |
| **SMTP** | Simple Mail Transfer Protocol |

### Related Links

- Jenkins: https://www.jenkins.io/doc/
- SonarQube: https://docs.sonarqube.org/
- Docker: https://docs.docker.com/
- Trivy: https://aquasecurity.github.io/trivy/
- ArgoCD: https://argo-cd.readthedocs.io/
- K3s: https://docs.k3s.io/
- cert-manager: https://cert-manager.io/docs/
- Nginx Ingress: https://kubernetes.github.io/ingress-nginx/

### Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-09-14 | Initial release |

---

## 📄 License

MIT License — free to use, modify, and share.

---

## 🙏 Contributing

Found a bug or want to improve this guide? Open an issue or PR:
- [App Repo](https://github.com/Rohitz999/devops-cicd-app)
- [GitOps Repo](https://github.com/Rohitz999/devops-mega-gitops)

---

## 📧 Contact

**Rohit Vishwakarma**
- GitHub: [@Rohitz999](https://github.com/Rohitz999)
- Email: rohitvishwakarma8082@gmail.com

---

**⭐ If this guide helped you, star the repos!**
