  

# 🧪 HADR Test Explanation

  
---
## 🎯 Purpose  

---
  
To validate **High Availability (HA)** and **Disaster Recovery (DR)** by simulating failures in the primary region and ensuring Route 53 correctly fails over to the secondary region. Tests also confirm that Auto Scaling Groups (ASGs) and Application Load Balancers (ALBs) behave as expected.


---
## ✅ Test Steps

---

### 1. **EC2 + ASG Self-Healing**  

- **Action:** Stop both EC2 instances in the Mumbai region while ASG is active.  

- **Why:** To confirm that the ASG detects the failure and automatically launches a replacement instances.  

- **Validation:**  

  - ALB health check remains healthy.  

  - Traffic continues without interruption.  

  - New EC2 instances joins the target group automatically.  
  
---
### 2. **Regional Failover (Mumbai → Virginia)**  

- **Action:**  

  1. Scale down the Mumbai ASG to **min=0, desired=0**.  

  2. After scaling down, **manually stop all remaining EC2 instances** in Mumbai.  

- **Why:** This ensures the ASG does not replace instances during the test, and the region is fully unavailable — simulating a real outage.  

- **Validation:**  

  - Mumbai ALB becomes unhealthy.  

  - Route 53 detects the failure and reroutes traffic to the Virginia ALB.  

  - Users continue to access the application from the secondary region.  

---
### 3. **Failback (Virginia → Mumbai)**  

- **Action:**  

  1. Scale the Mumbai ASG back up to **desired=2**.  

  2. Allow EC2 instances to launch and register with the ALB.  

- **Why:** To restore the primary region and validate that traffic can shift back once it is healthy.  

- **Validation:**  

  - Mumbai ALB health checks pass.  

  - Route 53 routes traffic back to Mumbai.  

  - Application continues serving without errors.  


---
## 📊 Validation Summary  
  ---

- **ASG Behavior:** Replaces failed instances when enabled.  

- **Regional Failover:** Requires ASG scaled to 0 and EC2s stopped to simulate a true outage.  

- **Route 53:** Correctly switches traffic between regions during failover and failback.  


---
