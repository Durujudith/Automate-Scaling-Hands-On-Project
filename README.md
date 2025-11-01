
#  AWS Auto Scaling Demo Project

This project demonstrates how **AWS Auto Scaling** dynamically adjusts compute capacity to maintain performance and cost efficiency.  
I built the environment manually — from networking to EC2 and scaling policies, and used a stress test to trigger scaling behavior in real time.


##  Project Overview

The goal of this project was to test **AWS Auto Scaling** behavior using **CPU-based scaling policies**.  
When CPU utilization rises above 50%, the Auto Scaling Group launches a new instance automatically.  
When utilization drops below 35%, it scales in — maintaining performance and reducing cost.  

This lab helped me understand how **automation, elasticity, and monitoring** work together inside AWS.



##  Step 1: VPC & Subnet Setup

**VPC:**  
- Created `vpc-test` with CIDR `10.0.0.0/16`

**Subnet:**  
- Added `demo-subnet` in `eu-north-1a` with CIDR `10.0.1.0/24`

**Security Group (`demo-sg`):**  
- Inbound: SSH (22) from my IP and all traffic for testing  
- Outbound: All traffic  

**Network ACL (`demo-nacl`):**  
- Inbound: SSH from my IP and all traffic  
- Outbound: All traffic  
- Associated with the subnet  

**Internet Gateway & Route Table:**  
- Created and attached Internet Gateway to `vpc-test`  
- Created Route Table → `0.0.0.0/0` → IGW  
- Associated the Route Table with the subnet  


##  Step 2: Launch Template

Created a Launch Template named **`auto-scaling-demo`** with the following:

- **AMI:** Amazon Linux 2023  
- **Instance Type:** `t3.micro`  
- **Key Pair:** `scaling.pem`  
- **Subnet:** `demo-subnet`  
- **Security Group:** `demo-sg`  

This template stores all EC2 launch settings for the Auto Scaling Group to reuse automatically.


##  Step 3: Auto Scaling Group (ASG)

Created an Auto Scaling Group named **`auto-scaling-demo-group`** using the launch template.

**Settings:**
- **VPC:** `vpc-test`
- **Subnet:** `demo-subnet`
- **Minimum Capacity:** 1  
- **Desired Capacity:** 1  
- **Maximum Capacity:** 3  
- **Health Check Type:** EC2 (300 seconds)  
- **Instance Replacement:** Prioritize Availability  

**Scaling Policy:**
- **Type:** Target Tracking  
- **Metric:** Average CPU Utilization  
- **Target Value:** 50%  
- **Warm-up Period:** 300 seconds  
- **Policy Name:** `auto-scaling-policy`

##  Step 4: Connect to EC2 Instance

SSH into the EC2 instance:
ssh -i scaling.pem ec2-user@<your-ec2-16.170.140.92>
chmod 400 "scaling.pem"


## Step 5: Install Web Server
Once connected, install and configure Apache HTTP server:

sudo yum update -y
sudo yum install -y httpd
sudo systemctl start httpd
sudo systemctl enable httpd
echo "Hello, Auto Scaling Demo!" | sudo tee /var/www/html/index.html
This creates a simple webpage displaying a message when accessed via browser.

## Step 6: Install and Run Stress Tool
The stress tool simulates CPU load to trigger scaling actions.

sudo yum install -y stress
stress --cpu 1 --timeout 300
This runs a 5-minute CPU stress test to intentionally push utilization above 50%, which triggers scaling events.

## Step 7: Monitoring & Results
CloudWatch Alarms

•Alarm Name	Condition	Action
•AlarmHigh	CPU > 50%	Triggered scaling out (launching a new instance)
•AlarmLow	CPU < 35%	Stabilized system; prepares for scale-in

## Scaling Activity

•Launching a new EC2 instance” occurred when CPU exceeded 50%.
•After stress completed, CPU dropped and AlarmLow returned to “OK” state.

## Instance Health
All reachability checks passed ✅
Instance status: Healthy ✅
EBS and system checks: Passed ✅

## Lessons Learned
•AWS Auto Scaling can fully manage scaling without manual intervention.
•Launch Templates simplify repeatable infrastructure.
•CloudWatch + Target Tracking = seamless automation.
•Stress testing helps validate scaling thresholds.


