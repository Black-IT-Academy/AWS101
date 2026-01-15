# Black IT Academy — Project 1: Host a Static Website on AWS

Welcome to **Project 1** for the Black IT Academy (BITA). In this lab, you’ll build a **real-world, production-style AWS architecture** that serves a **reachable website** you can add to your portfolio.

By the end, you’ll have:
- A website publicly accessible through a **Load Balancer**
- A **custom domain** (Route 53) and **HTTPS** (ACM)
- **Private EC2 web servers** behind an **Application Load Balancer**
- A secure **Bastion Host** pattern for SSH access
- **Auto Scaling** configured for resiliency and growth

> **Project Materials (Agenda + Slides):**  
> - Slide deck (PDF): docs/Slide Deck 
> - Agenda (PDF): : docs/Agenda  

---

## Reference Architecture

This project follows a typical two-AZ web architecture:
- **Route 53** receives DNS requests from end users
- **ACM** provides TLS/SSL certificates for HTTPS
- **ALB** distributes traffic across EC2 web servers
- **EC2 instances** run Apache in **private subnets**
- **Bastion Host** lives in a **public subnet** for secure SSH access
- **NAT Gateways** enable outbound internet access for private instances
- **S3** stores your static website files (HTML/CSS/JS)

> Put your architecture image in the repo at: `assets/reference-architecture.png`  
> (Recommended: use the diagram from the project materials/slides.)

---

## What You’ll Learn

Aligned to the project agenda and outcomes:

- **AWS Networking:** Create a VPC, public/private subnets, route tables, IGW, and NAT Gateway(s)  
- **Security Management:** Configure Security Groups and IAM roles for least-privilege access  
- **Secure Access:** SSH into private EC2 instances via a Bastion Host (no public exposure)  
- **Server Administration:** Launch EC2, install Apache, validate web connectivity  
- **Application Deployment:** Serve static files stored in S3 through Apache  
- **Load Balancing:** Configure an ALB + Target Group + health checks  
- **Scalability & Automation:** Launch Templates + Auto Scaling Groups (ASG)  
- **DNS & SSL:** Route 53 records + ACM certificate attached to ALB (HTTPS)

(These items match the provided agenda/slide deck.) :contentReference[oaicite:2]{index=2} :contentReference[oaicite:3]{index=3}

---

## Prerequisites

### Required
- AWS account (Free Tier is fine to start)
- Laptop/Desktop + stable internet
- Basic familiarity with:
  - VPC/subnets (conceptually)
  - SSH basics
  - EC2 fundamentals

### Strongly Recommended
- A text editor (VS Code)
- Terminal access (Mac/Linux terminal or Windows PowerShell)
- A simple static website ready to upload (or use the sample in `/website`)

---

## ⚠️ Cost Warning (Read This)
Some resources in this project are **not Free Tier**, especially:
- **NAT Gateway** (hourly + data processing)
- **Application Load Balancer** (hourly + LCU usage)
- **EC2** if you choose non-free-tier instances

✅ Do the **Cleanup** section at the end to avoid surprise charges.

---


---

## Project Checklist (Your Step-by-Step Build)

> This follows the project flow from the agenda deck:
> Network → Security → S3 → IAM → Bastion/EC2 → Apache → ALB → Route 53 + ACM → ASG → Wrap-up 

### 1) Network Setup (VPC + Subnets + Routes)
**Goal:** Build the foundational AWS network environment.

- [ ] Create a **VPC**
- [ ] Create an **Internet Gateway (IGW)** and attach it to the VPC
- [ ] Create **Public Subnets** (AZ-a, AZ-b)
- [ ] Create **Private Subnets** (AZ-a, AZ-b)
- [ ] Create a **Public Route Table**:
  - Route `0.0.0.0/0 → IGW`
  - Associate with public subnets
- [ ] Create **NAT Gateway(s)** in public subnet(s) (allocate Elastic IP)
- [ ] Create a **Private Route Table**:
  - Route `0.0.0.0/0 → NAT Gateway`
  - Associate with private subnets

---

### 2) Security Groups (Minimum Recommended Rules)
Create three security groups (as outlined in the slides):

1. **SSH Security Group**
   - Inbound: `22` from **Your IP**
2. **ALB Security Group**
   - Inbound: `80` and `443` from `0.0.0.0/0`
3. **Web Server Security Group**
   - Inbound: `80` and `443` from **ALB Security Group**
   - Inbound: `22` from **SSH Security Group** (for bastion→private access)

---

### 3) Upload Application Code (S3)
**Goal:** Store and prepare website files for hosting.

- [ ] Create an **S3 bucket**
- [ ] Upload your static website files (`HTML/CSS/JS/images`)
- [ ] (Optional but common) Put assets in a clean structure like:
  - `s3://your-bucket/site/index.html`

---

### 4) IAM Permissions (EC2 → S3 Access)
**Goal:** Enable secure access between AWS services.

- [ ] Create an **IAM Role** for EC2
- [ ] Attach permissions (starter approach in this lab): **S3 Full Access**
  - ✅ For a more “production” approach, scope this down to only your bucket.

---

### 5) Bastion Host + Private Web Servers
**Goal:** Launch and configure EC2 instances securely.

- [ ] Create an **SSH key pair** (`.pem`) and store it safely
- [ ] Launch a **Bastion Host EC2** in a **public subnet**
  - Associate a public IP
  - Attach **SSH Security Group**
- [ ] Launch **Web Server EC2** instance(s) in **private subnets**
  - No public IP
  - Attach **Web Server Security Group**
  - Attach the **IAM Role** for S3 access
- [ ] SSH flow validation:
  - [ ] SSH from your laptop → Bastion Host
  - [ ] SSH from Bastion Host → Private Web Server

> Tip: Use SSH agent forwarding or securely copy the key to bastion temporarily (then remove it). Document what you did in `docs/build-notes.md`.

---

### 6) Install Apache + Deploy Website Files
**Goal:** Install and configure the web application.

You can do this manually or via **User Data**. Recommended: use a bootstrap script.

**Example user data concept (Amazon Linux style):**
- Install Apache
- Start + enable service
- Pull website files from S3
- Copy to web root (`/var/www/html`)

Put your script in: `scripts/user-data-apache.sh`  
(Keep it simple, then improve over time.)

- [ ] Confirm Apache is running on each web server
- [ ] Confirm files exist in `/var/www/html`
- [ ] Confirm local curl test from inside the instance works:
  - `curl http://localhost`

---

### 7) Load Balancing Setup (ALB + Target Group)
**Goal:** Make the website reachable and resilient.

- [ ] Create a **Target Group**
  - Target type: Instances
  - Health check path: `/` (or `/index.html`)
- [ ] Create an **Application Load Balancer (ALB)**
  - Internet-facing
  - Public subnets in **two AZs**
  - Attach **ALB Security Group**
- [ ] Register targets (your private web servers) into the target group
- [ ] Validate:
  - [ ] Targets show **Healthy**
  - [ ] ALB DNS name loads your website

---

### 8) Domain + SSL (Route 53 + ACM)
**Goal:** Make it portfolio-ready.

- [ ] Register or use a domain in **Route 53**
- [ ] Request a public certificate in **AWS Certificate Manager (ACM)**
  - Validate via DNS (recommended)
- [ ] Add an **HTTPS (443) listener** on the ALB using your ACM cert
- [ ] Create Route 53 record:
  - A/AAAA Alias → ALB  
  - (Or CNAME if needed, depending on setup)

✅ Final validation:
- [ ] `https://yourdomain.com` loads successfully
- [ ] Browser shows a valid certificate (no warnings)

---

### 9) Auto Scaling (Launch Template + ASG)
**Goal:** Make your setup scalable and closer to production patterns.

- [ ] Create a **Launch Template**
  - Includes IAM role, security group, user data, AMI, instance type
- [ ] Create an **Auto Scaling Group**
  - Place instances across private subnets (AZ-a, AZ-b)
  - Attach to the ALB target group
- [ ] Test scaling trigger (basic):
  - Adjust desired capacity temporarily
  - Confirm new instances become healthy targets

---

## Acceptance Criteria (What “Done” Looks Like)

You’re complete when you can prove:
- [ ] Website loads through **ALB DNS name**
- [ ] Website loads through **your domain**
- [ ] Website loads over **HTTPS**
- [ ] Web servers are in **private subnets**
- [ ] SSH access uses **Bastion Host** (no direct public SSH to private instances)
- [ ] Target group health checks show **Healthy**
- [ ] ASG can add/remove instances successfully

---

## What to Submit (Portfolio Deliverables)

Create a folder: `docs/screenshots/` and include:

1. VPC + subnet layout (console screenshot)
2. Security Groups rules (showing inbound sources)
3. S3 bucket objects (website files)
4. ALB listeners (HTTP + HTTPS)
5. Target group health (Healthy targets)
6. Route 53 record pointing to ALB
7. Browser screenshot of your live site (domain + lock icon)

Also update `docs/build-notes.md` with:
- What you built
- What broke
- How you fixed it
- What you’d improve in a “real production” setup

---

## Common Troubleshooting

**Targets unhealthy**
- Check Security Groups (ALB → WebServer on 80/443)
- Confirm Apache is running
- Confirm health check path exists

**ALB loads but shows default page**
- Website files not copied to `/var/www/html`
- Confirm S3 sync/copy succeeded
- Confirm correct bucket/path

**Can’t SSH into private instance**
- Verify bastion can reach private subnet (routing + NACLs)
- Verify Web Server SG allows `22` from SSH SG
- Verify you’re SSH’ing from bastion, not your laptop

**HTTPS not working**
- ACM cert must be in the same region as ALB
- Cert validation must be issued/complete
- ALB listener 443 must reference the correct cert

---

## Cleanup (Do This To Avoid Charges)

When finished, delete resources in this order:

- [ ] Auto Scaling Group (set desired=0, then delete)
- [ ] Launch Template (optional)
- [ ] ALB + Target Group
- [ ] NAT Gateway(s) + release Elastic IP(s)
- [ ] EC2 instances (bastion + any standalones)
- [ ] Route 53 records (and hosted zone if created for this project)
- [ ] S3 bucket objects + bucket (if not needed)
- [ ] Security Groups (if unused)
- [ ] VPC (last)

---

## Stretch Goals (Optional Enhancements)

If you want to go beyond:
- Replace “S3 Full Access” with **least-privilege bucket policy**
- Add **CloudWatch alarms** to trigger scaling
- Use **SSM Session Manager** instead of SSH for private access
- Add a `/health` endpoint and tighten health checks
- Add CI/CD (GitHub Actions) to push website updates to S3

---

## BITA Community Notes

- Ask questions early. Document your blockers.
- Pair up and compare architectures.
- If you get stuck, submit:
  - Your error message
  - A screenshot
  - What you tried
  - Where in the checklist you are

---

## License

This repo is for educational use within the Black IT Academy community unless otherwise noted.

---

## Acknowledgements

This lab is based on a “build it live” style AWS project format and adapted for BITA students to create a portfolio-ready deployment. Project flow and learning outcomes align with the provided agenda/slide materials. 




