
# 🗺️ AWS Broken Web App Challenge - Project 5



![2025-06-25 (27)](https://github.com/user-attachments/assets/4e40879f-e0ab-4d19-b0fc-4482a047644d)




![2025-06-25 (23)](https://github.com/user-attachments/assets/7d16175f-eddd-49ff-b18c-eed7b01a02f4)



[![AWS](https://img.shields.io/badge/AWS-Cloud-FF9900?logo=amazon-aws)](https://aws.amazon.com)
[![Challenge](https://img.shields.io/badge/Challenge-Debugging-red?logo=debug)](https://github.com)
[![Difficulty](https://img.shields.io/badge/Difficulty-Intermediate-orange)](https://github.com)
[![Instructor](https://img.shields.io/badge/Instructor-Chigozie%20O.-blue)](https://github.com)


---

## 🎯 Challenge Overview

Welcome to the AWS Broken Web App Challenge! This hands-on project is designed to test your AWS troubleshooting skills by presenting you with a **deliberately broken** web application infrastructure. Your mission is to identify, diagnose, and fix multiple issues across various AWS services. This guide walks you through deploying and troubleshooting a deliberately broken web application using AWS CloudFormation. The application runs on EC2 instances behind an Application Load Balancer (ALB), but contains four intentional issues that need to be identified and resolved.

#Architecture Components#
-The deployed infrastructure typically includes:

-Application Load Balancer (ALB) - Routes traffic to EC2 instances
-EC2 Instances - Host the web application (usually in Auto Scaling Group)
-Security Groups - Control network access
-Target Groups - Health checks and traffic routing
-VPC/Subnets - Network infrastructure
-IAM Roles - Instance permissions

## 🚀 Getting Started

### Prerequisites
-Before starting, ensure you have:

-AWS CLI configured or AWS Management Console access
-Permissions to create CloudFormation stacks, EC2 instances, ALBs, and related resources
-Access to the US-EAST-1 region specifically

#Part 1: Initial Deployment#
-Step 1: Deploy the CloudFormation Stack
-Using AWS Management Console:

#Navigate to CloudFormation#

-Log into AWS Management Console
-Switch to US-EAST-1 region (critical requirement)
-Go to Services → CloudFormation


#Create New Stack#

-Click "Create stack" → "With new resources (standard)"
-Select "Template is ready"
-Choose "Amazon S3 URL"
-Enter template URL: https://cloud-mastery-bootcamp.s3.amazonaws.com/lab-guides/solutions-architecture/broken-web-app-challenge.json


#Configure Stack#

-Stack name: Enter your first name (as specified in requirements)
-Review any parameters the template requires
-Click "Next"


#Configure Stack Options#

-Add tags if desired (optional)
-Leave other options as default unless specified
-Click "Next"


-Review and Deploy

-Review all settings
-Check "I acknowledge that AWS CloudFormation might create IAM resources"
-Click "Create stack"


### 🧩 What You'll Encounter

This challenge simulates real-world scenarios where applications fail due to:
- **Misconfigured security groups**
- **Broken load balancer routing**
- **DNS resolution issues**
- **IAM permission problems**
- **Network connectivity failures**
- **Application-level errors**

### 🎓 Learning Objectives

By completing this challenge, you will:
- ✅ **Master AWS debugging techniques** across multiple services
- ✅ **Develop systematic troubleshooting methodologies**
- ✅ **Learn to read and interpret AWS logs and metrics**
- ✅ **Practice real-world incident response scenarios**
- ✅ **Gain confidence in production environment troubleshooting**

---
![2025-06-25 (17)](https://github.com/user-attachments/assets/7ca949d5-056c-49d4-a00e-8faf5747971b)




## 🏗️ Broken Infrastructure Architecture

The challenge presents a "broken" multi-tier web application with the following (intentionally faulty) components:

```
Internet → Route 53 (❌ DNS Issues) → ALB (❌ Misconfigured) → EC2 Instances (❌ Multiple Problems)
                                                ↓
                                    Target Groups (❌ Health Check Failures)
                                                ↓
                                        Security Groups (❌ Wrong Rules)
```

### 🔧 Technology Stack (Broken State)

| Component | AWS Service | Status | Common Issues |
|-----------|-------------|---------|---------------|
| **DNS** | Route 53 | ❌ Broken | Wrong record types, missing records |
| **Load Balancer** | Application Load Balancer | ❌ Broken | Misconfigured listeners, wrong target groups |
| **Compute** | EC2 Instances | ❌ Broken | Wrong AMI, security group issues, user data errors |
| **Security** | Security Groups | ❌ Broken | Incorrect ports, wrong sources, missing rules |
| **Storage** | EBS Volumes | ❌ Broken | Mount issues, permission problems |
| **Monitoring** | CloudWatch | ⚠️ Limited | Logs not configured, missing metrics |

---


# AWS Application Load Balancer Troubleshooting Guide

## 🎯 Overview

This repository documents a comprehensive AWS Application Load Balancer (ALB) troubleshooting case study. The project showcases systematic debugging of a multi-layered infrastructure problem that resulted in `ERR_CONNECTION_TIMED_OUT` errors and complete website inaccessibility.

## 🚨 Problem Statement

**Initial Symptoms:**
- Website completely unreachable via ALB DNS name
- `ERR_CONNECTION_TIMED_OUT` in browser
- Application Load Balancer appearing "healthy" in AWS Console
- No clear error indicators in basic monitoring

**Environment:**
- **Region:** us-east-1
- **VPC:** vpc-09553cc714f43fb75
- **Application:** Web service with health check endpoint
- **Expected Result:** Publicly accessible website via ALB

## 🔍 Root Cause Analysis

Through systematic debugging, we identified **4 critical issues** that compounded to cause complete service failure:

### Issue #1: Auto Scaling Group Configuration
- **Problem:** ASG `DesiredCapacity` set to 0
- **Impact:** No EC2 instances running to serve traffic
- **Detection:** `aws autoscaling describe-auto-scaling-groups`

### Issue #2: Target Group Registration  
- **Problem:** Launched instances not registered to target group
- **Impact:** Target group showed 0 healthy targets
- **Detection:** `aws elbv2 describe-target-health`

### Issue #3: Health Check Configuration Error
- **Problem:** Typo in health check path: `/healthcheck.hmtl` (missing 't')
- **Impact:** Instances marked as unhealthy despite running web service
- **Detection:** Health check failures in target group monitoring

### Issue #4: Network Architecture Misconfiguration (Critical)
- **Problem:** ALB deployed in private subnets instead of public subnets
- **Impact:** Internet traffic unable to reach ALB
- **Detection:** Subnet route table analysis

## 🛠️ Solution Implementation

### Step 1: Fix Auto Scaling Group
```bash
# Update ASG to launch instances
aws autoscaling update-auto-scaling-group \
    --auto-scaling-group-name "Osenat-ASG-rWJH9GBHtcOh" \
    --desired-capacity 2 \
    --region us-east-1
```

### Step 2: Register Instances to Target Group
```bash
# Register running instances
aws elbv2 register-targets \
    --target-group-arn "arn:aws:elasticloadbalancing:us-east-1:139561979448:targetgroup/Osenat-ALBTa-3NXVLJGHLIRD/06f6852d224ba7d1" \
    --targets Id=i-0abc123def456789 Id=i-0def456abc123789 \
    --region us-east-1
```

### Step 3: Fix Health Check Path
```bash
# Update target group health check
aws elbv2 modify-target-group \
    --target-group-arn "arn:aws:elasticloadbalancing:us-east-1:139561979448:targetgroup/Osenat-ALBTa-3NXVLJGHLIRD/06f6852d224ba7d1" \
    --health-check-path "/healthcheck.html" \
    --region us-east-1
```

### Step 4: Move ALB to Public Subnets
**Before:**
- subnet-0b4b97970007c1bc1 (private)
- subnet-0d7d2ca09639f5cd7 (private)

**After:**
- subnet-0b2480f00dd3bc2a9 (public)  
- subnet-08c511dc7e750530e (public)

*Note: ALB subnet modification performed via AWS Console*

## 📊 Diagnostic Commands

### Essential Troubleshooting Commands
```bash
# Check ALB status and configuration
aws elbv2 describe-load-balancers --region us-east-1

# Verify target group health
aws elbv2 describe-target-health \
    --target-group-arn [TARGET-GROUP-ARN] \
    --region us-east-1

# Check Auto Scaling Group status
aws autoscaling describe-auto-scaling-groups \
    --region us-east-1

# Analyze subnet configuration
aws ec2 describe-subnets \
    --filters "Name=vpc-id,Values=[VPC-ID]" \
    --query 'Subnets[].{SubnetId:SubnetId,AvailabilityZone:AvailabilityZone,CidrBlock:CidrBlock,Tags:Tags}' \
    --region us-east-1

# Check route tables for public internet access
aws ec2 describe-route-tables \
    --filters "Name=vpc-id,Values=[VPC-ID]" \
    --region us-east-1
```

## 🏗️ Final Architecture

### Correct Configuration
```
Internet (0.0.0.0/0)
    ↓
Internet Gateway
    ↓
Application Load Balancer (Public Subnets)
    ↓
Target Group (2 healthy targets)
    ↓
EC2 Instances (Private Subnets)
    ↓
Web Application (/healthcheck.html)
```

### Architecture Best Practices Implemented
- ✅ **ALB in Public Subnets:** Enables internet accessibility
- ✅ **EC2 in Private Subnets:** Enhanced security posture
- ✅ **Multi-AZ Deployment:** High availability across us-east-1a and us-east-1b
- ✅ **Proper Health Checks:** Reliable service monitoring
- ✅ **Auto Scaling:** Dynamic capacity management

## 🎯 Key Learnings

### Technical Insights
1. **Subnet Placement is Critical:** Public ALBs must be in public subnets
2. **Health Check Accuracy:** Small typos can cause complete service failure
3. **Target Registration:** Instances don't auto-register to target groups
4. **Multi-Layer Debugging:** Infrastructure issues often have multiple root causes

### Debugging Methodology
1. **Start with Basic Connectivity:** Can traffic reach the load balancer?
2. **Verify Target Health:** Are backend instances healthy and registered?
3. **Check Configuration:** Are health checks, paths, and settings correct?
4. **Analyze Network Path:** Is traffic properly routed through all layers?

## 📈 Performance Metrics

**Resolution Statistics:**
- **Total Debug Time:** ~2 hours
- **Issues Identified:** 4 distinct problems
- **Commands Executed:** 15+ diagnostic commands
- **Final Result:** 100% service availability

**Before vs After:**
- **Before:** 0% availability (complete failure)
- **After:** 100% availability (full functionality)
- **Target Health:** 0/0 → 2/2 healthy targets
- **Response Time:** Timeout → <200ms

## 🔧 Tools and Technologies

### AWS Services Used
- **EC2:** Compute instances
- **ELB:** Application Load Balancer
- **Auto Scaling:** Dynamic capacity management
- **VPC:** Network isolation and routing
- **CloudFormation:** Infrastructure as Code

### CLI Tools
- **AWS CLI:** Primary diagnostic and configuration tool
- **AWS Console:** Subnet configuration management
- **Browser DevTools:** Connection testing and error analysis

## 📚 Files in This Repository

```
├── README.md                          # This documentation
├── architecture-diagram.html          # Visual architecture diagram
├── troubleshooting-commands.md        # Complete command reference
├── linkedin-post.md                   # Social media summary
└── lessons-learned.md                 # Detailed technical insights
```

## 🚀 Getting Started

### Prerequisites
- AWS CLI configured with appropriate permissions
- Access to target AWS account and region
- Basic understanding of AWS networking concepts

### Quick Diagnosis Checklist
1. **Verify ALB exists and is active**
2. **Check ALB subnet placement (public vs private)**
3. **Confirm target group has healthy instances**
4. **Validate health check configuration**
5. **Test internet connectivity to ALB DNS name**

## 🤝 Contributing

This case study serves as a reference for AWS infrastructure troubleshooting. Contributions for additional scenarios or improved diagnostic techniques are welcome.

## 📄 License

This documentation is provided under MIT License for educational and reference purposes.

## 📞 Support

For questions about this troubleshooting methodology or AWS best practices, please refer to:
- [AWS Documentation](https://docs.aws.amazon.com/)
- [AWS Support](https://aws.amazon.com/support/)
- [AWS Community Forums](https://forums.aws.amazon.com/)

---

**Last Updated:** June 2025  
**Status:** ✅ Resolved - Service Fully Operational
