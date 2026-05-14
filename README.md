# AWS re/Start - Step-by-Step Build Guides
## All 3 Projects: Hands-On in Your AWS Account

---

# ✅ BEFORE YOU START (All Projects)

1. Log in to your AWS Console → https://aws.amazon.com
2. Make sure you are in the correct **Region** (e.g., us-east-1)
3. Go to **IAM** → Make sure your user has admin or required permissions
4. Open **AWS CloudShell** (top bar icon) for running commands

---

---

# 🌐 PROJECT 1: Static Website Hosting
## S3 + CloudFront + Route 53

**Time to complete:** ~45 minutes
**Difficulty:** ⭐⭐☆ Intermediate

---

## STEP 1 — Create Your S3 Bucket

1. Go to **S3** in the AWS Console
2. Click **Create Bucket**
3. Bucket name: `my-website-yourname` (must be globally unique)
4. Region: `us-east-1`
5. Uncheck **Block all public access** → Confirm the warning
6. Leave everything else as default → Click **Create Bucket**

---

## STEP 2 — Enable Static Website Hosting

1. Click on your new bucket
2. Go to **Properties** tab
3. Scroll down to **Static website hosting** → Click **Edit**
4. Select **Enable**
5. Index document: `index.html`
6. Error document: `error.html`
7. Click **Save changes**

---

## STEP 3 — Upload Your Website Files

1. Create a simple `index.html` file on your computer:
```html
<!DOCTYPE html>
<html>
<head><title>My AWS Website</title></head>
<body>
  <h1>Hello from AWS S3!</h1>
  <p>Hosted with S3 + CloudFront + Route 53</p>
</body>
</html>
```
2. Go to your bucket → **Objects** tab → Click **Upload**
3. Upload your `index.html` file
4. Click **Upload**

---

## STEP 4 — Add Bucket Policy (Make it Public)

1. Go to **Permissions** tab in your bucket
2. Click **Bucket Policy** → Edit
3. Paste this policy (replace YOUR-BUCKET-NAME):
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::YOUR-BUCKET-NAME/*"
    }
  ]
}
```
4. Click **Save changes**

---

## STEP 5 — Create CloudFront Distribution

1. Go to **CloudFront** in the AWS Console
2. Click **Create Distribution**
3. Origin domain: Select your S3 bucket from the dropdown
4. Under **Viewer protocol policy**: Select **Redirect HTTP to HTTPS**
5. Default root object: `index.html`
6. Click **Create Distribution**
7. Wait 5-10 minutes for deployment (Status changes to "Enabled")
8. Copy your **Distribution domain name** (e.g., `abc123.cloudfront.net`)
9. Open it in your browser — your website should load! ✅

---

## STEP 6 — Test Your Website

1. Copy the CloudFront domain name
2. Paste it in your browser
3. You should see your **Hello from AWS S3!** page
4. Screenshot this for your portfolio! 📸

---

## ✅ PROJECT 1 COMPLETE!
**What you built:** A globally delivered static website with HTTPS

---
---

# ⚖️ PROJECT 2: Auto Scaling Web Application
## EC2 + VPC + ALB + Auto Scaling + RDS

**Time to complete:** ~90 minutes
**Difficulty:** ⭐⭐⭐ Intermediate-High

---

## STEP 1 — Create a VPC

1. Go to **VPC** in the AWS Console
2. Click **Create VPC** → Select **VPC and more**
3. Name: `my-webapp-vpc`
4. IPv4 CIDR: `10.0.0.0/16`
5. Number of Availability Zones: **2**
6. Number of public subnets: **2**
7. Number of private subnets: **2**
8. NAT Gateways: **None** (to save cost)
9. Click **Create VPC**
10. Wait for creation to complete ✅

---

## STEP 2 — Create Security Groups

### Web Server Security Group
1. Go to **VPC** → **Security Groups** → Create
2. Name: `web-server-sg`
3. VPC: Select `my-webapp-vpc`
4. Inbound Rules:
   - Type: HTTP | Port: 80 | Source: 0.0.0.0/0
   - Type: HTTPS | Port: 443 | Source: 0.0.0.0/0
   - Type: SSH | Port: 22 | Source: My IP
5. Click **Create**

### Database Security Group
1. Create another Security Group
2. Name: `database-sg`
3. Inbound Rules:
   - Type: MySQL/Aurora | Port: 3306 | Source: `web-server-sg`
4. Click **Create**

---

## STEP 3 — Launch EC2 Instance (Template)

1. Go to **EC2** → **Launch Templates** → Create
2. Name: `webapp-template`
3. AMI: **Amazon Linux 2023**
4. Instance type: `t2.micro` (free tier)
5. Key pair: Create new or use existing
6. Security group: Select `web-server-sg`
7. Under **Advanced details** → User data, paste:
```bash
#!/bin/bash
yum update -y
yum install -y httpd
systemctl start httpd
systemctl enable httpd
echo "<h1>Hello from $(hostname -f)</h1>" > /var/www/html/index.html
```
8. Click **Create Launch Template**

---

## STEP 4 — Create Application Load Balancer

1. Go to **EC2** → **Load Balancers** → Create
2. Choose **Application Load Balancer**
3. Name: `my-webapp-alb`
4. Scheme: **Internet-facing**
5. VPC: `my-webapp-vpc`
6. Subnets: Select both **public subnets**
7. Security group: `web-server-sg`
8. Create a **Target Group**:
   - Name: `webapp-target-group`
   - Target type: Instances
   - Protocol: HTTP | Port: 80
   - Health check path: `/`
9. Click **Create Load Balancer**
10. Copy the **DNS name** of your ALB

---

## STEP 5 — Create Auto Scaling Group

1. Go to **EC2** → **Auto Scaling Groups** → Create
2. Name: `webapp-asg`
3. Launch template: `webapp-template`
4. VPC: `my-webapp-vpc`
5. Subnets: Select both **public subnets**
6. Attach to **existing load balancer** → Select `webapp-target-group`
7. Group size:
   - Minimum: **1**
   - Desired: **2**
   - Maximum: **4**
8. Scaling policy:
   - Target tracking → CPU Utilization → Target: **70%**
9. Click **Create Auto Scaling Group**

---

## STEP 6 — Create RDS Database

1. Go to **RDS** → **Create Database**
2. Engine: **MySQL**
3. Template: **Free tier**
4. DB instance identifier: `my-webapp-db`
5. Master username: `admin`
6. Master password: Create a strong password
7. Instance class: `db.t3.micro`
8. VPC: `my-webapp-vpc`
9. Subnet group: Create new (select private subnets)
10. Public access: **No**
11. Security group: `database-sg`
12. Click **Create Database** (takes ~5 minutes)

---

## STEP 7 — Test Your Setup

1. Go to **EC2** → **Load Balancers**
2. Copy the **DNS name** of your ALB
3. Paste it in your browser
4. You should see **"Hello from ip-10-0-x-x..."**
5. Refresh the page — notice the hostname changes! (Load balancing working ✅)
6. Screenshot this for your portfolio! 📸

---

## ✅ PROJECT 2 COMPLETE!
**What you built:** A fault-tolerant, auto-scaling web application

---
---

# 🤖 PROJECT 3: Automated S3 Backup System
## Lambda + S3 + CloudWatch + IAM (Python Boto3)

**Time to complete:** ~60 minutes
**Difficulty:** ⭐⭐☆ Intermediate

---

## STEP 1 — Create Two S3 Buckets

1. Go to **S3** → Create Bucket
2. First bucket name: `my-source-files-yourname`
3. Region: `us-east-1` → Create
4. Second bucket name: `my-backup-files-yourname`
5. Region: `us-east-1` → Create

### Enable Versioning on Backup Bucket
1. Click `my-backup-files-yourname`
2. Go to **Properties** → **Bucket Versioning** → Edit
3. Select **Enable** → Save

---

## STEP 2 — Upload a Test File to Source Bucket

1. Click `my-source-files-yourname`
2. Click **Upload** → Add a simple text file from your computer
3. Click **Upload** ✅

---

## STEP 3 — Create IAM Role for Lambda

1. Go to **IAM** → **Roles** → Create Role
2. Trusted entity: **AWS Service** → Lambda
3. Click **Next**
4. Search and attach: `AmazonS3FullAccess` (for testing)
5. Search and attach: `CloudWatchLogsFullAccess`
6. Role name: `lambda-s3-backup-role`
7. Click **Create Role**

---

## STEP 4 — Create Lambda Function

1. Go to **Lambda** → **Create Function**
2. Select **Author from scratch**
3. Function name: `s3-backup-function`
4. Runtime: **Python 3.11**
5. Execution role: **Use existing role** → `lambda-s3-backup-role`
6. Click **Create Function**

---

## STEP 5 — Write the Backup Code

1. In the Lambda console, scroll to **Code source**
2. Delete the existing code
3. Paste this Python code:
```python
import boto3
import os
from datetime import datetime

def lambda_handler(event, context):
    s3 = boto3.client('s3')
    
    source_bucket = 'my-source-files-yourname'   # Change this
    backup_bucket = 'my-backup-files-yourname'   # Change this
    timestamp = datetime.now().strftime('%Y-%m-%d_%H-%M-%S')
    
    response = s3.list_objects_v2(Bucket=source_bucket)
    
    if 'Contents' not in response:
        print("No files found to backup.")
        return {'statusCode': 200, 'body': 'No files to backup'}
    
    backed_up = 0
    for obj in response['Contents']:
        source_key = obj['Key']
        backup_key = f"backups/{timestamp}/{source_key}"
        
        s3.copy_object(
            CopySource={'Bucket': source_bucket, 'Key': source_key},
            Bucket=backup_bucket,
            Key=backup_key
        )
        backed_up += 1
        print(f"Backed up: {source_key}")
    
    print(f"Done! {backed_up} files backed up at {timestamp}")
    return {'statusCode': 200, 'body': f'Backed up {backed_up} files'}
```
4. Click **Deploy**

---

## STEP 6 — Test the Lambda Function

1. Click **Test** → Configure test event
2. Event name: `TestBackup`
3. Leave default JSON → Click **Save**
4. Click **Test** again
5. You should see **"Done! 1 files backed up"** in the logs ✅
6. Go to your backup bucket → You should see a `backups/` folder with your file! 📸

---

## STEP 7 — Schedule with CloudWatch (Auto Daily Backup)

1. Go to **CloudWatch** → **Rules** → **Create Rule**
   (Or search for **EventBridge** → Rules → Create)
2. Event source: **Schedule**
3. Cron expression: `0 0 * * ? *` (runs every day at midnight)
4. Target: **Lambda function** → Select `s3-backup-function`
5. Click **Create Rule** ✅

---

## STEP 8 — Verify in CloudWatch Logs

1. Go to **CloudWatch** → **Log Groups**
2. Find `/aws/lambda/s3-backup-function`
3. Click the latest log stream
4. You should see your backup logs 📸

---

## ✅ PROJECT 3 COMPLETE!
**What you built:** A fully automated serverless backup system

---
---

# 🏆 ALL 3 PROJECTS COMPLETE!

## What You've Accomplished:
| Project | Services Used | Skill Demonstrated |
|---------|--------------|-------------------|
| Static Website | S3, CloudFront, Route 53, IAM | Hosting & CDN |
| Auto Scaling App | EC2, VPC, ALB, ASG, RDS, IAM | High Availability |
| Lambda Backup | Lambda, S3, CloudWatch, IAM | Serverless & Automation |

## Next Steps:
- 📸 Take screenshots of each completed project
- 📝 Add screenshots to your GitHub README files
- 🔗 Push code to GitHub repositories
- 💼 Add projects to your LinkedIn profile
- 🎯 Start applying for Cloud Support / Junior Cloud Engineer roles!

---
*AWS re/Start Program — Keep building, keep growing! ☁️*
