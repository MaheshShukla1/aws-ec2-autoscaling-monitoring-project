# 🚀 AWS Day 3 Hands-On Project: Scaling + Monitoring

**Author:** Mahesh Shukla  
**Goal:** Build a highly available, scalable, and monitored web infrastructure using **AWS EC2**, **EBS**, **AMI**, **Launch Template**, **Auto Scaling Group (ASG)**, **Application Load Balancer (ALB)**, and **CloudWatch**.  
**Duration:** Day 3 of my AWS DevSecOps Journey  
**Focus:** Automation + High Availability + Monitoring  

---

## 🧭 Project Overview

This hands-on project demonstrates how to:

1. Launch and configure an EC2 instance.
2. Attach and persist an EBS volume.
3. Create a custom AMI for reuse.
4. Deploy an Auto Scaling Group (ASG) using a Launch Template.
5. Integrate with an Application Load Balancer (ALB).
6. Configure CloudWatch alarms to monitor instance health and CPU utilization.
7. Test scaling behavior by simulating load.

---

## 🧱 Architecture Overview

**Architecture Components:**

- **EC2 Instance** (Web Server)
- **EBS Volume** (Persistent Storage)
- **AMI** (Golden Image)
- **Launch Template**
- **Auto Scaling Group**
- **Application Load Balancer**
- **CloudWatch Alarms + Metrics**

![AWS Architecture](screenshots/aws-architecture.png)

---

## ⚙️ Step 1: Launch EC2 Instance

### 🧩 Configuration
- **AMI:** Amazon Linux 2
- **Instance Type:** t2.micro
- **Security Group:** Allow HTTP (80) + SSH (22)
- **Key Pair:** `aws-day3-key`

### 🧰 Commands Used

```bash
# Connect to instance
ssh -i aws-day3-key.pem ec2-user@<EC2-Public-IP>

# Update system
sudo yum update -y

# Install Apache Web Server
sudo yum install httpd -y

# Start and enable service
sudo systemctl start httpd
sudo systemctl enable httpd

# Create a test webpage
echo "<h1>Hello from Mahesh Ec2 Webserver</h1>" | sudo tee /var/www/html/index.html
```

📸 **Screenshot:**  
![EC2 Launch](screenshots/01-ec2-launch.png)  
![Web Server Running](screenshots/02-webserver-running.png)

✅ **Expected Output:**  
Access the public IP in a browser →  
`http://<EC2-Public-IP>` → Page displays your custom message.

---

## 💾 Step 2: Attach and Mount EBS Volume

### 🧩 Configuration
- Create an **EBS Volume** (8 GB, same Availability Zone)
- Attach to your instance

### 🧰 Commands Used

```bash
# List available disks
lsblk

# Create partition and format
sudo mkfs -t xfs /dev/xvdf

# Create mount point
sudo mkdir /data

# Mount the volume
sudo mount /dev/xvdf /data

# Verify mount
df -h
```

📸 **Screenshot:**  
![EBS Attached](screenshots/03-ebs-attached.png)

### 🧪 Test Persistence Across Reboot

```bash
# Add entry to fstab
echo "/dev/xvdf /data xfs defaults,nofail 0 2" | sudo tee -a /etc/fstab

# Reboot instance
sudo reboot

# Verify after reboot
df -h
```

📸 **Screenshot:**  
![EBS Persistent](screenshots/04-ebs-persistent.png)

✅ **Expected Output:**  
Volume should auto-mount at `/data` after reboot.

---

## 🧬 Step 3: Create a Custom AMI

```bash
# Create AMI from EC2 (via AWS Console or CLI)
aws ec2 create-image --instance-id <instance-id> --name "webserver-ami-day3" --no-reboot
```

📸 **Screenshot:**  
![Custom AMI](screenshots/05-custom-ami.png)

✅ **Result:**  
Reusable AMI with web server and EBS config baked in.

---

## 📦 Step 4: Create a Launch Template

### 🧩 Configuration
- Use the **Custom AMI**
- Instance Type: `t2.micro`
- User Data Script: (automates web server setup)

### 🧰 User Data Script

```bash
#!/bin/bash
yum update -y
yum install -y httpd stress
systemctl start httpd
systemctl enable httpd
echo "<h1>Auto Scaling Instance - $(hostname)</h1>" > /var/www/html/index.html
```

📸 **Screenshot:**  
![Launch Template](screenshots/06-launch-template.png)

✅ **Expected Output:**  
Every new instance automatically runs a web server on boot.

---

## 🧩 Step 5: Configure Load Balancer (ALB)

### 🧩 Steps
1. Create a **Target Group** for HTTP (port 80)
2. Register no instances yet (ASG will attach automatically)
3. Create an **Application Load Balancer**
4. Add the target group as listener target

📸 **Screenshot:**  
![Target Group](screenshots/07-alb-targetgroup.png)

✅ **Result:**  
ALB DNS name will serve traffic across instances.

---

## 🔁 Step 6: Create Auto Scaling Group (ASG)

### 🧩 Configuration
- Use the Launch Template
- Target Group: connect ALB target group
- Desired Capacity: 2  
- Min: 1  
- Max: 3  
- Health Check: ELB + EC2

📸 **Screenshot:**  
![ASG Created](screenshots/08-asg-created.png)

✅ **Expected Output:**  
Two instances automatically launched and registered in the ALB target group.

---

## 📊 Step 7: Set Up CloudWatch Monitoring

### 🧩 Configuration
- Metric: **CPUUtilization**
- Threshold: **>= 70%**
- Action: Scale out (+1 instance)
- Notification: SNS topic (optional)

📸 **Screenshot:**  
![CloudWatch Alarm](screenshots/09-cloudwatch-alarm.png)

### 🧰 Commands Used

```bash
# Install stress tool if not already
sudo yum install stress -y

# Generate CPU load
stress --cpu 2 --timeout 300
```

✅ **Expected Output:**  
- CPU utilization spikes in CloudWatch.
- ASG scales out automatically to add a new instance.

---

## 🌐 Step 8: Verify Load Balancer Access

Visit the ALB DNS name in your browser:  
`http://<ALB-DNS-Name>`  

📸 **Screenshot:**  
![ALB Working](screenshots/10-alb-working.png)

✅ **Expected Output:**  
Requests should round-robin between multiple instances (hostname changes).

---

## 🧾 Verification Checklist

| Step | Description | Status | Screenshot |
|------|--------------|--------|-------------|
| 1 | EC2 Instance launched and web server configured | ✅ | 01, 02 |
| 2 | EBS volume attached and persistent | ✅ | 03, 04 |
| 3 | Custom AMI created | ✅ | 05 |
| 4 | Launch Template created with User Data | ✅ | 06 |
| 5 | ALB + Target Group configured | ✅ | 07 |
| 6 | Auto Scaling Group created | ✅ | 08 |
| 7 | CloudWatch alarm configured | ✅ | 09 |
| 8 | ALB tested and working | ✅ | 10 |

---

## 💡 Key Learnings

- Understood EC2 lifecycle and persistent storage configuration.
- Learned to automate setup using User Data scripts.
- Built reusable AMIs for fast deployment.
- Configured scalable infrastructure with ASG and ALB.
- Monitored infrastructure health with CloudWatch metrics and alarms.
- Tested scaling events in real time using load simulation.

---

## 🔮 Future Enhancements

- [ ] Add HTTPS support using AWS ACM + ALB  
- [ ] Integrate CloudWatch Logs and Dashboard visualizations  
- [ ] Use Terraform to automate the entire setup  
- [ ] Add SNS notification for scaling events  
- [ ] Implement CI/CD pipeline for web app updates  

---

## 🧰 Tech Stack

- **AWS EC2**, **EBS**, **AMI**
- **Launch Template**, **ASG**, **ALB**
- **CloudWatch**, **SNS**
- **Amazon Linux 2**, **Apache HTTPD**

---

## 🧑‍💻 Author

**Mahesh Shukla**  
_Aspiring AWS DevSecOps Engineer_  
📍 Mumbai, India  
🔗 [LinkedIn](http://linkedin.com/in/maheshsh)  

---

> _“Scaling is not a feature, it’s a mindset. Monitor everything, automate everything.”_
