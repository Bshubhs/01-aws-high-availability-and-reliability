# 🛠️ HADR Setup – AWS Multi-Region (Mumbai & Virginia)

This guide sets up a high-availability, disaster recovery (HADR) architecture using EC2, ALB, ASG, and Route 53 across Mumbai and Virginia regions.

---

## 🏗️ Step 1: Create VPCs in Both Regions

1.1 Create one VPC per region  
- CIDR: `10.0.0.0/16`  
- Subnets:  
  - AZ A: `10.0.1.0/24`  
  - AZ B: `10.0.2.0/24`  

1.2 Attach Internet Gateway  
1.3 Create Route Table with `0.0.0.0/0` → associate with both subnets  
1.4 Ensure subnets are public (auto-assign public IP enabled)

✅ Use consistent naming for VPC, IGW, and Route Tables.

---

## ⚙️ Step 2: Launch EC2 & Configure Auto Scaling

2.1 Create Launch Template (per region)  
- AMI: Amazon Linux  
- Instance type: `t2.micro`  
- Key pair & SG: region-specific  
- User Data:
```
  #!/bin/bash
  yum update -y
  yum install -y httpd
  systemctl start httpd
  systemctl enable httpd
  echo "<h1>Hello from region: [Region]</h1><p><strong>EC2 IP:</strong> $(hostname -I | cut -d\" \" -f1)</p>" > /var/www/html/index.html
  systemctl restart httpd
```


2.2 Create Auto Scaling Group
- Launch template: use above
- Subnets: AZ A & AZ B
- Capacity: min 1, desired 2, max 3
- Attach to Target Group

✅ ASGs ensure availability across AZs. Launch templates simplify provisioning.

---

## 🌐 Step 3: Create Load Balancer & Target Group

3.1 Create Target Group (per region)
- Protocol: HTTP
- Port: 80
- Health check path: `/`

3.2 Create ALB
- Type: Internet-facing
- Subnets: AZ A & AZ B
- Listener: HTTP (port 80)
- Forward traffic to target group

✅ ALBs distribute traffic across EC2 instances. Health checks ensure reliability.

---

## 🌍 Step 4: Configure Route 53 for DNS Failover

4.1 Create Public Hosted Zone
- Domain: `example.com` (example)

4.2 Add A-records with failover routing
- Primary → Alias → Mumbai ALB
- Secondary → Alias → Virginia ALB

4.3 Attach Health Check to Mumbai ALB
- Protocol: HTTP
- Port: 80
- Path: `/`
- Failure threshold: 1

4.4 Test DNS behavior
- ✅ Mumbai ALB healthy → traffic goes to Mumbai
- ❌ Mumbai ALB fails → traffic reroutes to Virginia


---

## 🔁 Step 5: Perform DR Testing

5.1 (Pre‑step) In the Primary region (Mumbai), edit the Auto Scaling Group and set Desired capacity = 0 and Min capacity = 0.
(This prevents the ASG from launching replacement EC2 instances when you stop them. Without this, failover won’t trigger properly.)

5.2 Stop all EC2 instances in the Primary region (Mumbai) → Route 53 health checks will fail, and traffic will automatically failover to the Secondary region (Virginia).

5.3 Restart instances in the Mumbai region by resetting the ASG capacities (e.g., Desired = 2, Min = 1, Max = 3) → Route 53 will detect healthy endpoints again and direct traffic back to the Primary region


---

## 📊 Monitoring & Logging (Optional)

- Enable **CloudWatch Alarms** for CPU, Memory, and Instance Health.
- Enable **ALB Access Logs** to S3.
- Setup **SNS Alerts** for failures.

---

## 🧱 Final Architecture Summary

- ✅ 2 VPCs (Mumbai & Virginia)
- ✅ 2 public subnets per region
- ✅ EC2 + ASG per region
- ✅ ALB + TG per region
- ✅ Route 53 DNS failover
- ✅ Modular naming for automation & scaling
