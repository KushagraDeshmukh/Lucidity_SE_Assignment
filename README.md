# Using AWS Disk Utilization Monitoring — Lucidity Technical Assignment - Kushagra

This repository provides a minimal, working example of a disk utilization monitoring solution for Amazon EC2 instances running across multiple AWS accounts. The approach uses:

- **AWS Systems Manager (SSM)** for secure, auditable access  
- **CloudWatch Agent** for collecting disk metrics  
- **CloudWatch Alarms** for disk usage thresholds  
- **Ansible** for configuration management and automated rollout  
- **Tag-based discovery** to onboard instances at scale  

The goal is to detect low disk space early, centralize visibility, and ensure scalability as more accounts and instances are added.

---

## **1. Architecture Summary**

### **Access & Management**
- EC2 instances are managed using **AWS Systems Manager**.  
- No SSH keys or opened ports are required.  
- Each instance uses an **IAM instance profile** that allows:
  - SSM connectivity
  - CloudWatch metrics publishing

### **Metric Collection**
- **CloudWatch Agent** runs on each instance.  
- It collects:
  - `disk_used_percent`
  - `disk_free_bytes`
  - `disk_total_bytes`
- Metrics are published every **60 seconds** to the `CWAgent` namespace.

### **Aggregation & Monitoring**
- CloudWatch Metrics and Alarms live in each member AWS account.
- A central monitoring account:
  - can view cross-account metrics using **AWS Managed Grafana**, or  
  - can receive alarms via **SNS + EventBridge** for unified alert routing.

### **Scalability**
Instances are included automatically using a tag such as:

```
monitoring:disk=true
```

Ansible’s dynamic AWS inventory discovers these instances and deploys or updates CloudWatch Agent accordingly.

---

## **2. Repository Contents**

```
.
├── README.md
├── diagrams/
│   ├── architecture.mmd
│   └── architecture.png
│
├── ansible/
│   ├── inventories/
│   │   └── aws_ec2.yaml
│   ├── playbooks/
│   │   ├── bootstrap-cw-agent.yml
│   │   └── create-alarms.yml
│   └── roles/
│       └── cw_agent/
│           └── templates/
│               └── cw-config.json.j2
│
└── iam/
    └── instance_role_policy.json
```

---

## **3. Quick Start Guide**

### **Step 1 — Tag the instance(s)**  

```
monitoring:disk=true
```

### **Step 2 — Run Ansible Bootstrap Playbook**

```bash
ansible-playbook -i ansible/inventories/aws_ec2.yaml   ansible/playbooks/bootstrap-cw-agent.yml   --extra-vars "account_id=123456789012 team=platform"
```

### **Step 3 — Validate Metrics in CloudWatch**

```bash
aws cloudwatch list-metrics   --namespace CWAgent   --metric-name disk_used_percent
```

### **Step 4 — (Optional) Create an Alarm**

```bash
ansible-playbook ansible/playbooks/create-alarms.yml
```

---

## **4. IAM Requirements**

Instances require permissions for:

- SSM connectivity
- CloudWatch metrics publishing
- CloudWatch Logs publishing

A minimal example is provided in:

```
iam/instance_role_policy.json
```

---

## **5. Production Considerations**

For production deployments, consider adding:

- AWS Managed Grafana dashboards
- CloudWatch Metric Streams for long‑term data storage
- Central alert routing using EventBridge + Lambda
- Automated remediation workflows (e.g., EBS expansion via SSM Automation)
- Stricter IAM scoping and guardrails across accounts

---

## **6. Purpose of This Submission**

This repository demonstrates:

- Secure, scalable cross-account VM access  
- Reliable disk metric collection  
- Practical alerting  
- Infrastructure-as-code principles  
- A design suitable for customer‑facing PoCs  

Prepared for **Lucidity – Sales Engineer Technical Assignment**.
