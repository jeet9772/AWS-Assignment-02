# Assignment 2 – Deployment Strategies & Amazon S3

## Objective

The objective of this assignment is to deploy the **Spring3Hibernate** application using two deployment strategies:

1. Recreate Deployment
2. Rolling Deployment

The assignment also includes storing static assets in **Amazon S3** and configuring the EC2 instance to retrieve and use the assets.

---

# Architecture

```text
                         Amazon S3
              ┌──────────────────────────┐
              │ Assignment 2 Bucket      │
              │                          │
              │  artifacts/              │
              │  assets/                 │
              │  configs/                │
              │  logs/                   │
              └────────────┬─────────────┘
                           │
                    IAM Role / S3 Read
                           │
                           ▼
                  ┌─────────────────┐
                  │      EC2        │
                  │  Amazon Linux   │
                  │                 │
                  │ Java + Maven    │
                  │ Apache Tomcat   │
                  └────────┬────────┘
                           │
                           ▼
                 Spring3Hibernate App
```

---

# Environment

| Component             | Details               |
| --------------------- | --------------------- |
| Cloud Provider        | AWS                   |
| Region                | ap-south-1 (Mumbai)   |
| OS                    | Amazon Linux 2023     |
| Instance Type         | t3.micro              |
| Java                  | Corretto 17           |
| Maven                 | 3.8.4                 |
| Application Server    | Apache Tomcat 9.0.113 |
| Application           | Spring3Hibernate      |
| Storage               | Amazon S3             |
| Deployment Strategies | Recreate, Rolling     |

---

# Application Repository

GitHub Repository:

```text
https://github.com/opstree/spring3hibernate.git
```

Application:

```text
Spring3Hibernate
```

WAR file:

```text
Spring3HibernateApp.war
```

---

# 1. Recreate Deployment

## Step 1: Launch EC2 Instance

An EC2 instance was launched for the initial application deployment.

Instance configuration:

```text
Instance Type: t3.micro
OS: Amazon Linux 2023
Region: ap-south-1
```


<img width="1440" height="900" alt="Screenshot 2026-09-24 at 5 54 00 PM" src="https://github.com/user-attachments/assets/d8ffb46b-a804-4326-91a9-35ad16b51d6f" />

---

## Step 2: Install Required Packages

```bash
sudo dnf install git java-17-amazon-corretto maven -y
```

Verify:

```bash
java -version
mvn -version
git --version
```


<img width="1440" height="900" alt="Screenshot 2026-09-24 at 5 56 03 PM" src="https://github.com/user-attachments/assets/ec1a3eb1-9eb9-40bc-8bb0-0ad68a0f20ec" />

---

## Step 3: Clone Application Repository

```bash
git clone https://github.com/opstree/spring3hibernate.git
cd spring3hibernate
```

<img width="1440" height="900" alt="Screenshot 2026-09-24 at 6 00 18 PM" src="https://github.com/user-attachments/assets/401f2477-e6cd-4b73-bb08-9ddfcadde6c0" />

---

## Step 4: Build the Application

The Maven configuration was updated for compatibility with the current Java and Maven environment.

Build the application:

```bash
mvn clean package -DskipTests
```

The generated WAR file:

```text
target/Spring3HibernateApp.war


<img width="1440" height="900" alt="Screenshot 2026-09-24 at 6 03 49 PM" src="https://github.com/user-attachments/assets/71a90ed9-ca07-4839-9810-663d3a3912b2" />

```

---

## Step 5: Install Apache Tomcat

Tomcat was installed under:

```text
/opt/tomcat/apache-tomcat-9.0.113
```

Start Tomcat:

```bash
cd /opt/tomcat/apache-tomcat-9.0.113
./bin/startup.sh
```

Verify port:

```bash
ss -lntp | grep 8080
```

---

## Step 6: Deploy WAR File

```bash
sudo cp target/Spring3HibernateApp.war \
/opt/tomcat/apache-tomcat-9.0.113/webapps/
```

Application URL:

```text
http://<EC2-PUBLIC-IP>:8080/Spring3HibernateApp/
```

---

## Step 7: Create AMI

After successfully deploying and verifying the application, an AMI was created.

AMI:

```text
assignment2-recreate-v1-ami
```

AMI ID:

```text
ami-063e784ee916137e6
```

The AMI contains the configured application environment, Java, Maven, Tomcat and deployed application.

---

## Step 8: Recreate Deployment

A new EC2 instance was launched from the created AMI.

The old application instance can be replaced with the new instance.

This demonstrates the **Recreate Deployment** strategy:

```text
Old Instance
     │
     ▼
Stop/Replace
     │
     ▼
New Instance from AMI
     │
     ▼
Application Running
```

Application was verified successfully after recreation.

---

# 2. Rolling Deployment

## Step 1: Create Launch Template

A Launch Template was created for the rolling deployment.

Launch Template:

```text
assignment2-rolling-v1
```

Instance type:

```text
t3.micro
```

The Launch Template uses the application AMI.

---

## Step 2: Create Auto Scaling Group

ASG configuration:

```text
Name: assignment2-rolling-asg
Minimum Capacity: 1
Desired Capacity: 1
Maximum Capacity: 2
```

The Auto Scaling Group manages the application instances.

---

# Rolling Deployment – V1

The first version of the application was deployed using the initial AMI.

V1 application output:

```text
Sample WebApp CRUD Example for CI
```

The application was verified on the EC2 instance.

---

# Rolling Deployment – V2

## Step 1: Update Application

The application was updated to Version 2.

V2 page contains:

```text
Sample WebApp CRUD Example for CI - V2
Rolling Deployment - Version 2
```

---

## Step 2: Build V2

```bash
mvn clean package -DskipTests
```

Generated WAR:

```text
target/Spring3HibernateApp.war
```

---

## Step 3: Upload V2 Artifact to S3

The V2 WAR file was uploaded to the S3 bucket.

S3 location:

```text
s3://jeetendra-assignment2-deployment-2026/artifacts/Spring3HibernateApp-V2.war
```

Upload command:

```bash
aws s3 cp Spring3HibernateApp-V2.war \
s3://jeetendra-assignment2-deployment-2026/artifacts/
```

---

## Step 4: Create V2 AMI

A new AMI was created containing the V2 application.

AMI:

```text
assignment2-rolling-v2-ami
```

AMI ID:

```text
ami-0bbbc761f802b4f33
```

---

## Step 5: Create New Launch Template Version

A new version of the Launch Template was created using the V2 AMI.

```text
Launch Template: assignment2-rolling-v1
Version: 2
AMI: ami-0bbbc761f802b4f33
Instance Type: t3.micro
```

---

## Step 6: Update Auto Scaling Group

The Auto Scaling Group was updated to use Launch Template Version 2.

```text
ASG:
assignment2-rolling-asg

Launch Template:
assignment2-rolling-v1

Version:
2
```

---

## Step 7: Start Instance Refresh

An Instance Refresh was started to replace the old V1 instance with the V2 instance.

Process:

```text
V1 Instance
     │
     ▼
Instance Refresh
     │
     ▼
Launch V2 Instance
     │
     ▼
Terminate V1 Instance
     │
     ▼
V2 Instance Running
```

Instance Refresh completed successfully.

---

## Step 8: Verify V2

The new V2 instance was verified successfully.

Application URL:

```text
http://3.110.99.147:8080/Spring3HibernateApp/
```

Expected output:

```text
Sample WebApp CRUD Example for CI - V2
Rolling Deployment - Version 2
```

---

# 3. Amazon S3 – Static Assets

Amazon S3 was used to store application artifacts and static assets.

Bucket:

```text
jeetendra-assignment2-deployment-2026
```

Region:

```text
ap-south-1
```

Bucket structure:

```text
jeetendra-assignment2-deployment-2026/
├── artifacts/
├── assets/
├── configs/
└── logs/
```

---

## S3 Asset

CSS file:

```text
assets/style.css
```

CSS content:

```css
body {
    font-family: Arial, sans-serif;
}

h2 {
    margin: 20px;
}
```

---

# IAM Role for S3 Access

An IAM role was attached to the EC2 instance.

Role:

```text
Assignment2-EC2-S3-ReadOnly
```

Policy:

```text
AmazonS3ReadOnlyAccess
```

The role allows the EC2 instance to retrieve objects from S3 without storing AWS access keys on the server.

Verify AWS identity:

```bash
aws sts get-caller-identity
```

---

# Retrieve CSS from S3

The EC2 instance retrieved the CSS file from S3:

```bash
aws s3 cp \
s3://jeetendra-assignment2-deployment-2026/assets/style.css \
/tmp/style.css
```

Verify:

```bash
cat /tmp/style.css
```

---

# Configure Application to Use CSS

The CSS file retrieved from S3 was copied into the deployed application:

```bash
sudo cp /tmp/style.css \
/opt/tomcat/apache-tomcat-9.0.113/webapps/Spring3HibernateApp/style.css
```

The application `index.html` was configured with:

```html
<link rel="stylesheet" href="style.css">
```

---

# Verify Static Asset

Verify that the application references the CSS:

```bash
curl -s \
http://localhost:8080/Spring3HibernateApp/ | grep "style.css"
```

Expected:

```html
<link rel="stylesheet" href="style.css">
```

Verify CSS is served by the application:

```bash
curl -s \
http://localhost:8080/Spring3HibernateApp/style.css
```

Expected:

```css
body {
    font-family: Arial, sans-serif;
}

h2 {
    margin: 20px;
}
```

---

# Final Deployment Flow

```text
                 GitHub
                    │
                    ▼
             Spring3Hibernate
                    │
                    ▼
               Maven Build
                    │
                    ▼
                  WAR
                    │
          ┌─────────┴─────────┐
          │                   │
          ▼                   ▼
   Recreate Deployment   Rolling Deployment
          │                   │
          ▼                   ▼
        AMI V1          Launch Template V1
                              │
                              ▼
                       Application V1
                              │
                         Update Code
                              │
                              ▼
                         Build V2
                              │
                              ▼
                         S3 Artifact
                              │
                              ▼
                           AMI V2
                              │
                              ▼
                    Launch Template V2
                              │
                              ▼
                       ASG Instance Refresh
                              │
                              ▼
                         Application V2
```

---

# Verification Checklist

* [x] EC2 instance configured
* [x] Java installed
* [x] Maven installed
* [x] Git installed
* [x] Spring3Hibernate application cloned
* [x] Application built successfully
* [x] Tomcat configured
* [x] Recreate Deployment completed
* [x] Recreate AMI created
* [x] Rolling Deployment V1 completed
* [x] V2 application created
* [x] V2 WAR uploaded to S3
* [x] V2 AMI created
* [x] Launch Template Version 2 created
* [x] Auto Scaling Group updated
* [x] Instance Refresh completed successfully
* [x] Rolling Deployment V2 verified
* [x] S3 bucket created
* [x] Static CSS uploaded to S3
* [x] IAM role attached to EC2
* [x] EC2 successfully retrieved CSS from S3
* [x] Application successfully served the CSS

---

# Conclusion

The Spring3Hibernate application was successfully deployed using:

1. **Recreate Deployment**
2. **Rolling Deployment**

Amazon S3 was also integrated for storing application artifacts and static assets. The EC2 instance used an IAM role to securely retrieve the CSS asset from S3.

Both deployment strategies and the S3 static asset requirement were successfully implemented and verified.

