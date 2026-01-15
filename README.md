# Black IT Academy — Project 1: Host a Static Website on AWS

Welcome to **Project 1** for the Black IT Academy (BITA). In this lab, you’ll build a **real-world, production-style AWS architecture** that serves a **reachable website** you can add to your portfolio.

By the end, you’ll have:
- A website publicly accessible through a **Load Balancer**
- A **custom domain** (Route 53) and **HTTPS** (ACM)
- **Private EC2 web servers** behind an **Application Load Balancer**
- A secure **Bastion Host** pattern for SSH access
- **Auto Scaling** configured for resiliency and growth

> **Project Materials (Agenda + Slides):**  
> - Slide deck (PDF): :contentReference[oaicite:0]{index=0}  
> - Agenda (PDF): :contentReference[oaicite:1]{index=1}  

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

## Repo Layout (Suggested)


