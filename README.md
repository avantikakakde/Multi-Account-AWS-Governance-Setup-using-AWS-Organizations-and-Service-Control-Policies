# Multi-Account-AWS-Governance-Setup-using-AWS-Organizations-and-Service-Control-Policies

## Project Overview

This project demonstrates how to implement **centralized governance across multiple AWS accounts** using **AWS Organizations and Service Control Policies (SCPs)**.

In enterprise environments, different teams such as **Development, Testing, and Production** operate separate AWS accounts. Without proper governance, users may create expensive resources or modify security configurations.

This project implements a governance framework that enforces **security, compliance, and cost control policies across all AWS accounts.**

---

## Architecture Overview

```
AWS Organization
│
├── Root
│
├── Dev OU
│   └── Dev Account
│
├── Test OU
│   └── Test Account
│
└── Prod OU
    └── Prod Account
```

Each Organizational Unit (OU) has **Service Control Policies (SCPs)** attached to enforce governance rules.

---

## Technologies Used

- AWS Organizations  
- Service Control Policies (SCP)  
- IAM  
- CloudTrail  
- AWS Management Console  

---

## Step 1: AWS Organizations Setup

### Create AWS Organization

1. Login to the **AWS Management Account**
2. Open **AWS Organizations**
3. Click **Create Organization**
4. Enable **All Features**

---

### Create Organizational Units (OUs)

Created three Organizational Units:

- Dev OU
- Test OU
- Prod OU

Steps:

1. Go to **AWS Organizations**
2. Click **Organize Accounts**
3. Create Organizational Units

Screenshot:
![](./images/Screenshot%202026-03-17%20122823.png)

---

### Move Accounts to OUs

Member accounts were placed in their respective OUs.

| Account | OU |
|------|------|
| Dev Account | Dev OU |
| Test Account | Test OU |
| Prod Account | Prod OU |



---

## Step 2: Service Control Policies (SCP)

Service Control Policies are used to enforce restrictions across AWS accounts.

![](/images/Screenshot%202026-03-17%20122954.png)

---

### SCP 1: Deny Large EC2 Instances in Dev

Purpose: Prevent developers from launching expensive EC2 instances.

#### Policy

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyLargeEC2Instances",
      "Effect": "Deny",
      "Action": "ec2:RunInstances",
      "Resource": "*",
      "Condition": {
        "StringLike": {
          "ec2:InstanceType": [
            "m5.*",
            "c5.*",
            "r5.*"
          ]
        }
      }
    }
  ]
}
```

Attached to: **Dev OU**

Screenshot:
![](/images/Screenshot%202026-03-17%20123347.png)


---

### SCP 2: Prevent Disabling CloudTrail

Purpose: Ensure logging is always enabled for auditing.

#### Policy

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyCloudTrailDisable",
      "Effect": "Deny",
      "Action": [
        "cloudtrail:StopLogging",
        "cloudtrail:DeleteTrail"
      ],
      "Resource": "*"
    }
  ]
}
```

Attached to: **Root OU**

Screenshot:
![](/images/Screenshot%202026-03-17%20123410.png)


---

### SCP 3: Restrict Region Usage

Purpose: Allow AWS usage only in approved regions.

#### Policy

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyUnapprovedRegions",
      "Effect": "Deny",
      "NotAction": [
        "iam:*",
        "route53:*",
        "cloudfront:*"
      ],
      "Resource": "*",
      "Condition": {
        "StringNotEquals": {
          "aws:RequestedRegion": [
            "ap-south-1"
          ]
        }
      }
    }
  ]
}
```

Attached to: **Root OU**

Screenshot:

![](/images/Screenshot%202026-03-17%20123428.png)


---

## Step 3: Validation

To validate the governance policies, restricted actions were tested.

---

### Test 1: Launch Large EC2 Instance in Dev Account

Attempted to launch:

```
m5.large
```

Result:

```
Access Denied
```

Screenshot:

![](/images/Screenshot%202026-03-17%20123444.png)


---


## Governance Benefits

### Security
Prevents disabling CloudTrail and ensures audit logs are always enabled.

### Cost Control
Restricts launching expensive EC2 instance types in development environments.

### Compliance
Enforces region restrictions and security policies.

### Centralized Governance
All policies are managed from a single **AWS Organizations management account**.

### Scalability
New accounts automatically inherit governance policies.

---

## Risk Mitigation Strategy

| Risk | Mitigation |
|-----|-----|
| Expensive resources created by developers | SCP blocking large EC2 instances |
| Logging disabled | CloudTrail protection policy |
| Usage of unapproved regions | Region restriction SCP |
| Security misconfigurations | Centralized governance |



---

## Conclusion

This project demonstrates how **AWS Organizations and Service Control Policies (SCPs)** can be used to enforce centralized governance across multiple AWS accounts.

By implementing these policies, organizations can improve:

- Security
- Cost control
- Compliance
- Multi-account management