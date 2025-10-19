# 🚀 Deploying a Scalable AI Chatbot Infrastructure on Google Cloud

![GitHub last commit](https://img.shields.io/badge/Status-Completed-success)
![Platform](https://img.shields.io/badge/Platform-Google%20Cloud-blue)
![Stack](https://img.shields.io/badge/Stack-Docker%20%7C%20NGINX%20%7C%20OpenWebUI%20v0.6.26%20%7C%20OpenRouter%20%7C%20Debian-lightgrey)
![Category](https://img.shields.io/badge/Focus-AI%20Engineering%20%7C%20Cloud%20Infrastructure%20%7C%20DevOps-orange)

---

## 🧭 Table of Contents

1. [Project Overview](#project-overview)
2. [Objectives](#objectives)
3. [Architecture Overview](#architecture-overview)
4. [Cloud Network Architecture](#cloud-network-architecture)
5. [VPS Instance Configuration](#vps-instance-configuration)
6. [Technical Stack](#technical-stack)
7. [Security & Network Configuration](#security--network-configuration)
8. [AI Integration](#ai-integration)
9. [Operations & Monitoring](#operations--monitoring)
10. [Performance & Results](#performance--results)
11. [Challenges & Solutions](#challenges--solutions)
12. [Lessons Learned](#lessons-learned)
13. [Future Improvements](#future-improvements)
14. [Setup Guide (Reproducible Deployment)](#setup-guide-reproducible-deployment)
15. [Testing](#testing)
16. [Screenshots](#screenshots)
17. [Acknowledgments](#acknowledgments)
18. [Author](#author)

---

## 📘 Project Overview

This project involved researching, integrating, and deploying a **secure, scalable AI chatbot infrastructure** on **Google Cloud Platform (GCP)**.  

The goal was to create a **cost-efficient and transparent AI assistant** using **OpenWebUI** integrated with **OpenRouter API**, enabling multiple LLM providers under a unified, white-label web interface.

The system operates under a **pay-per-token model**, enabling transparency and eliminating fixed subscription costs.  

Initially developed as a personal project, it later evolved into a technical demonstration of **AI infrastructure, cloud automation, and DevOps integration** for future commercial use.

---

## 🎯 Objectives

- Deploy a self-hosted AI chatbot using containerized services.  
- Ensure secure HTTPS access via NGINX reverse proxy and Certbot.  
- Integrate multiple AI models from OpenRouter (OpenAI, Anthropic, DeepInfra, xAI).  
- Build a maintainable, scalable cloud setup using **Debian** VPS in **Google Cloud Compute Engine**.  
- Demonstrate end-to-end ownership from research to production deployment.

---

## 🧠 Architecture Overview

```
                        ┌────────────────────────┐
                        │   User Web Browser     │
                        │  (chatbot.domain.com)  │
                        └────────────┬───────────┘
                                     │ HTTPS (443)
                                     ▼
                            ┌─────────────────┐
                            │     NGINX       │
                            │  Reverse Proxy  │
                            └────────┬────────┘
                                     │
                                     ▼
                     ┌───────────────────────────────────────┐
                     │  Docker Container: OpenWebUI v0.6.26  │
                     │  Host Port: 3001  →  Container 8080   │
                     └───────────────────────────────────────┘
                                     │
                                     ▼
                     ┌──────────────────────────────────┐
                     │  OpenRouter API (External)       │
                     │  Multi-LLM Providers Integration │
                     └──────────────────────────────────┘
```

**Summary**
- One container: `open-webui`  
- NGINX reverse proxy handles routing, SSL termination, CORS, and WebSockets.  
- HTTPS via Let’s Encrypt / Certbot.  
- Static IP + firewall rules via `ufw` and VPC rules.  
- VPS on GCP (e2-small, Debian 12).

---

## ☁️ Cloud Network Architecture

The deployment uses a **custom VPC** named `webhosting` inside **Google Cloud Platform (GCP)** for isolated and secure network configuration.

### Subnet: `prod1-subnet`
| Attribute | Value |
|------------|--------|
| Description | Production subnet |
| Region | us-east1 |
| CIDR Range | 10.10.0.0/24 |
| Gateway | 10.10.0.1 |
| Purpose | PURPOSE_PRIVATE |
| Stack Type | IPV4_ONLY |

**Key Notes**
- The private subnet provides internal IP routing within the `webhosting` VPC.  
- Outbound traffic is NAT-routed securely to the internet.  
- CIDR block (10.10.0.0/24) provides 256 IPv4 addresses.  
- The configuration ensures low latency and isolation for production-grade workloads.

---

## 🖥️ VPS Instance Configuration

The deployed virtual machine (`prod1`) runs as a **Compute Engine e2-small instance** under the GCP project `gcpprojectname`.

| Property | Value |
|-----------|--------|
| Status | RUNNING |
| Machine Type | e2-small (2 vCPU, 2 GB RAM) |
| CPU Platform | Intel Broadwell |
| Disk | 20 GB Persistent SSD (Debian 12 Bookworm, UEFI compatible) |
| Network | VPC: webhosting → Subnet: prod1-subnet |
| External IP | XX.XXX.XX.XXX |
| Internal IP | 10.10.0.2 |
| Shielded VM | Enabled (Secure Boot, vTPM, Integrity Monitoring) |
| Auto Restart | Enabled |
| Confidential Compute | Disabled |
| Zone | us-east1-b |
| Deletion Protection | Enabled |
| SSH Access | Custom user SSH keys |

Additional disk features: `VIRTIO_SCSI_MULTIQUEUE`, `GVNIC`, and live-migratable capabilities as per GCE features.  
Service account scope: `cloud-platform` (broad access for ops tooling).

---

## ⚙️ Technical Stack

| Component        | Technology / Version |
|------------------|----------------------|
| OS               | Debian 12.12 (Bookworm) |
| Reverse Proxy    | NGINX 1.22.1 |
| Containerization | Docker 28.4.0 |
| Web Framework    | OpenWebUI v0.6.26 |
| AI Provider      | OpenRouter API (OpenAI, Anthropic, DeepInfra, xAI) |
| Scripting / CLI  | Python 3.11.2 |
| Firewall         | UFW 0.36.2 |
| Cloud Platform   | Google Cloud Compute Engine |
| Domain Provider  | Namecheap |
| SSL Certificates | Let’s Encrypt (Certbot) |

---

## 🔒 Security & Network Configuration

### UFW Firewall Rules
```
45700/tcp   ALLOW IN  Anywhere    # Custom SSH
80/tcp      ALLOW IN  Anywhere    # HTTP
443/tcp     ALLOW IN  Anywhere    # HTTPS
10.10.0.0/24 ALLOW IN  Anywhere   # VCN Internal
```

### Google Cloud VPC (Security List Summary)
- Ingress allowed: ICMP (0, 3, 8, 11), TCP (22, 80, 443, 45700) 
- Public CIDR: `0.0.0.0/0` for required ports only  
- Local subnet: `10.10.0.0/24` for intra-VPC communication

### NGINX Configuration (with SSL redirection)
*(Maintained as working version)*

---

## 🤖 AI Integration

- **OpenWebUI v0.6.26** connected to **OpenRouter API** for multi-provider LLMs.  
- Allowed providers: **OpenAI**, **Anthropic**, **xAI**, **DeepInfra**.  
- Fallback model configured: **gpt-5-nano** (via OpenRouter settings).  
- Admin flow: first launch creates admin user; API key stored securely in the Admin panel.  
- No local Ollama server: all model calls routed through OpenRouter.  
- Verified full API functionality through CLI `curl` requests and live interface tests.

---

## 🧩 Operations & Monitoring

- Services managed via `systemctl` and Docker CLI.  
- Manual log reading on VPS (`docker logs`, `journalctl`).  
- Manual incremental snapshots + full-disk backups to **Google Cloud Storage** (S3-compatible).  
- Runtime stability observed over ~5 weeks.  
- Estimated real cost: **~$15 USD/month (VPS) + ~$10 initial API usage.**

---

## 📊 Performance & Results

- **Deployment Duration:** ~4 weeks from research to production.  
- **Environment:** GCE e2-small (2 vCPU, 2 GB RAM).  
- **Functional Validation:** Verified correct model response, model fallback, and end-to-end availability.  
- **Outcome:** Fully functional, production-ready AI chatbot (white-label).  
- **Feedback:** Positive peer review (stable and consistent responses).

---

## ⚠️ Challenges & Solutions

| Challenge | Resolution |
|------------|-------------|
| `SyntaxError: unexpected token 'i', "internal s"... is not valid JSON` | Enabled webhooks in NGINX, aligned Node.js version with deployment environment, corrected API route, and disabled internal Ollama API. |
| SSL routing & reverse proxy conflicts | Isolated `/`, `/api/`, and `/ws/` with proper upgrade headers and timeouts. |
| CORS issues | Explicit `Access-Control-*` headers in NGINX for UI and API routes. |

---

## 💡 Lessons Learned

- Reinforced **operational discipline** for containerized workloads (setup, backups, logs, uptime).  
- Refined **NGINX routing logic** for modern AI web apps (WebSockets, API, HTTPS).  
- Gained deeper hands-on experience in **self-hosted LLM environments** and **OpenRouter integration**.  
- Improved understanding of **DevOps workflows and cloud networking**.

---

## 🔮 Future Improvements

- Upgrade to latest OpenWebUI; enable web search tools.  
- Add custom branding and tenant-aware configuration.  
- Implement persistent storage and uptime monitoring.  

---

## 🧱 Setup Guide (Reproducible Deployment)

### 1) Prerequisites
- Google Cloud account (Compute Engine enabled)  
- Debian 12+ VM (2 vCPU, 2 GB RAM minimum)  
- Domain with DNS A record pointing to external static IP (`chatbot.domain.com`)  

### 2) Update Base System
```bash
sudo apt update && sudo apt upgrade -y
```

### 3) System Packages Installed
Below is the complete list of packages used in the production setup for monitoring, networking, and deployment:

| Category | Packages |
|-----------|-----------|
| Locale & Shell | locales, zsh, unzip, perl |
| Development Tools | git, curl, build-essential, lsof, tree |
| Docker Stack | docker-ce, docker-ce-cli, containerd.io, docker-buildx-plugin, docker-compose-plugin |
| Web & Security | nginx, apache2-utils, certbot, python3-certbot-nginx, ufw |
| Monitoring | bpytop |
| Networking | telnet, apt-transport-https, ca-certificates, gnupg, lsb-release |
| Cloud CLI | google-cloud-cli |

> Tip: group them in one command:
```bash
sudo apt install -y locales bpytop zsh unzip perl git curl build-essential lsof docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin ufw nginx apache2-utils certbot python3-certbot-nginx google-cloud-cli apt-transport-https ca-certificates gnupg lsb-release tree telnet
```

### 4) Containers
```bash
docker run -d -p 3001:8080 --name open-webui ghcr.io/open-webui/open-webui:main
```

### 5) Configure Reverse Proxy (NGINX)
Place the configuration file at:
```
/etc/nginx/sites-available/chatbot.conf
```
Then enable and reload:
```bash
ln -s /etc/nginx/sites-available/chatbot.conf /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

### 6) Secure Sockets Layer
```bash
sudo certbot --nginx -d chatbot.domain.com -d www.chatbot.domain.com
```

### 7) Security & Network Configuration
```bash
sudo ufw allow 80,443,45700/tcp
sudo ufw enable
sudo ufw status
```

---

### 🌐 Domain and DNS Configuration (Namecheap)

To make the AI chatbot accessible publicly through a secure HTTPS domain, a custom domain and DNS configuration were set up using **Namecheap**.  
This configuration maps the domain name to the public static IP of the Google Cloud VPS where the application is hosted.

### 🔧 Example DNS Records

| Type | Host | Value | TTL | Description |
|------|------|--------|------|-------------|
| A Record | @ | 203.0.113.45 | Automatic | Points the root domain to the VPS public IP |
| A Record | chatbot | 203.0.113.45 | Automatic | Points the chatbot subdomain to the same server |
| CNAME Record | www | exampleprojects.ai. | Automatic | Redirects `www` to the main domain |

> **Note:**  
> Replace `203.0.113.45` with your actual Google Cloud static IP, and replace `exampleprojects.ai` with your real domain name.

---

### 🔐 Configuring the OpenRouter API in OpenWebUI

This section explains how to obtain and configure your **OpenRouter API key** for use with **OpenWebUI**, enabling LLM interactions within your local or cloud deployment.

#### 1. Obtain Your OpenRouter API Key
1. Go to the official OpenRouter website: [https://openrouter.ai](https://openrouter.ai)
2. Create an account (if you don’t have one yet).
3. Navigate to your **API Keys** page: [https://openrouter.ai/keys](https://openrouter.ai/keys)
4. Click **“Create new key”** and copy the generated token.
5. Keep this key private — it’s your authentication credential.

#### 2. Purchase Credits and Set Spending Limit
1. In your OpenRouter account, open the **Billing** section.
2. Purchase credits according to your usage needs.
3. For testing and personal projects, a **$10 limit** is recommended.
4. You can set a spending cap to prevent accidental overuse and maintain cost transparency.

#### 3. Configure the API Key in OpenWebUI
1. Start your OpenWebUI container (either locally or on your cloud instance).
2. Access the web interface (default: `http://localhost:3000` or your public IP).  
3. Open the **Settings** panel from the sidebar.  
4. Go to the **Connections** tab.  
5. Select **“Add new connection”** → choose **Custom API**.  
6. Fill the fields as follows:  
   * **API Base URL:** `https://openrouter.ai/api/v1`  
   * **API Key:** paste the key you copied earlier.  
7. Click **Save** or **Connect** to confirm the configuration.

#### 4. Test the Connection
1. Return to the chat interface in OpenWebUI.  
2. Choose an available model (e.g., `openrouter/anthropic/claude-3`, `openrouter/openai/gpt-4o`).  
3. Send a short test message to confirm the integration.  
4. If the connection is successful, you’ll see a valid model response.

**✅ Summary:**  
Your OpenWebUI instance is now integrated with OpenRouter for multi-model access, ideal for **testing, prototyping, and learning about LLM deployments** in a reproducible environment.

---

## 🧠 Testing

**Internal (Server CLI)**
```bash
curl http://127.0.0.1:3001/api/v1/chat/completions -H "Content-Type: application/json" -H "Authorization: Bearer USER-OPENWEBUI-APIKEY" -d '{
  "model": "openai/gpt-4.1-nano",
  "messages": [
    { "role": "user", "content": "Hello AI, which GPT are you?" }
  ]
}'
```

**External (Internet)**
```bash
curl https://chatbot.domain.com/api/v1/chat/completions -H "Content-Type: application/json" -H "Authorization: Bearer USER-OPENWEBUI-APIKEY" -d '{
  "model": "openai/gpt-4.1-nano",
  "messages": [
    { "role": "user", "content": "Hello AI, which GPT are you?" }
  ]
}'
```

---

## 🖼️ Screenshots

![Screenshot #1](./assets/signin.png)
![Screenshot #2](./assets/model-selection.png)
![Screenshot #3](./assets/chat-example.png)

---

## 🙌 Acknowledgments

- **OpenWebUI** — Open-source web interface for LLMs.  
- **OpenRouter** — API aggregation platform for AI model access.  
- **Google Cloud Platform** — Compute Engine and cloud networking.  
- **Docker, Inc.** — Containerization stack.  
- **Namecheap** — Domain and DNS provider.  
- **Let’s Encrypt** — SSL/TLS certificates for encryption.

---

## 👨‍💻 Author

**Developed by [@dev-mikel](https://github.com/dev-mikel)**  
*Miguel Ladines | Electronics Engineer *  

Location: LATAM  
Contact: [ladinesmiguel770@gmail.com](mailto:ladinesmiguel770@gmail.com)