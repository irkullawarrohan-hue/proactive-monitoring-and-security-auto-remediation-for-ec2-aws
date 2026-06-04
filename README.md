CloudGuard — Monitoring and Auto-Remediation on AWS

Proactive cloud monitoring, serverless auto-remediation, and real-world security threat response — built for a financial services environment on AWS.


Overview:

CloudGuard is a hands-on AWS project that simulates a real-world scenario where a financial services company — CloudGuard Inc. — suffered a security breach because their operations team failed to detect unusual system behavior in time. The result was significant downtime and potential data exposure.
As the Cloud Support Engineer on this project, I designed and implemented a fully automated monitoring and incident response system that detects performance issues, triggers serverless remediation, and responds to active security threats — all without manual intervention.

The Problem:

No proactive monitoring on EC2 instances
No automated response to performance degradation (CPU spikes, disk exhaustion)
No threat detection for malicious network activity
Incident response was entirely manual and reactive


What I Built:

A multi-layer cloud operations system covering three areas:

Performance Monitoring and Auto-Remediation — CloudWatch alarms detecting CPU and disk issues, triggering Lambda to automatically tag affected instances
Security Threat Detection — GuardDuty identifying active port scanning and suspicious credential usage
Incident Response and Containment — VPC NACL deployed to block malicious traffic, with full incident documentation



Architecture:

<img width="1004" height="499" alt="image" src="https://github.com/user-attachments/assets/d8b020f4-0843-48bf-8ef7-ff63d1a4d0cd" />


EC2 Instances (Dev + Prod)
        │
        ▼
CloudWatch Agent → Custom Metrics (CPU, Disk, Memory)
        │
        ▼
CloudWatch Alarms (CPUUtilization ≥ 85%, disk_used_percent ≥ 80%)
        │
        ▼
SNS Topic (EC2-Alarms)
        ├──► Email Notification
        └──► Lambda Function (EC2-AutoRemediation)
                    │
                    ▼
            Tags EC2 Instance (Issue=HighCPU / Issue=LowDisk)

─────────────────────────────────────────────

Simulated Attack (nmap port scan from Dev → Prod)
        │
        ▼
AWS GuardDuty → Recon:EC2/PortProbeUnprotectedPort (MEDIUM)
        │
        ▼
Manual Investigation (journalctl logs)
        │
        ▼
VPC NACL Deployed → Blocked Dev IP on ports 1–1000
        │
        ▼
Formal Incident Report Created (SEC-2025-001)

AWS Services Used:

ServicePurposeAmazon EC2Dev and Prod instances hosting the environmentAmazon CloudWatchMetrics collection, alarms, and dashboardsCloudWatch AgentCustom metric collection (disk, memory, CPU)AWS LambdaServerless auto-remediation function (Python)Amazon SNSAlert routing to email and LambdaAWS GuardDutyIntelligent threat detectionAWS IAMRoles and least-privilege permissionsAWS VPC NACLsNetwork-level threat containment


Implementation Breakdown:

Step 1 — EC2 Environment Setup
Launched two EC2 instances on Amazon Linux 2023 (t2.micro):

Dev-Server — tagged Environment=Dev, installed stress tool for CPU simulation
Prod-Server — tagged Environment=Production, installed util-linux for disk simulation

Step 2 — CloudWatch Agent Installation
Installed and configured amazon-cloudwatch-agent on both instances using the configuration wizard. Set 60-second collection intervals for CPU, disk, and memory metrics. Verified agent running as active (running) on both instances.

Step 3 — IAM Role Configuration
Resolved a NoCredentialProviders error by creating EC2CloudWatchAgentRole with CloudWatchAgentServerPolicy. Attached the role to both instances. Created a custom agent config at /opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json to collect disk_used_percent, mem_used_percent, and CPU metrics.

Step 4 — CloudWatch Alarms

DevInstance-HighCPU — CPUUtilization ≥ 85%, 1-minute period, Average
ProdInstance-LowDisk — disk_used_percent ≥ 80%, 1-minute period, Average
Both alarms connected to SNS topic EC2-Alarms with email subscription

Step 5 — Lambda Auto-Remediation
Created EC2-AutoRemediation Lambda function in Python 3.10. The function:

Receives SNS alarm notifications as triggers
Extracts instance ID from alarm dimensions (handles both InstanceId and host formats)
Identifies issue type from alarm name (HighCPU or LowDisk)
Tags the affected instance with Key=Issue, Value=HighCPU or LowDisk

IAM permissions: AmazonEC2ReadOnlyAccess + custom inline policy EC2TaggingPermissions for ec2:CreateTags.

Step 6 — Alarm Testing and Validation
CPU Test: SSH into Dev, ran sudo stress --cpu 8 --timeout 300. Alarm transitioned from OK → In Alarm. Email received. Lambda executed and tagged instance with Issue=HighCPU.
Disk Test: SSH into Prod, ran fallocate -l 6G /home/ec2-user/fakefile. Alarm triggered. Lambda tagged instance with Issue=LowDisk. Cleaned up with sudo rm.

Step 7 — GuardDuty Setup and Security Simulation
Enabled GuardDuty in us-east-1. From Dev instance, ran a port scan against Prod:
bashsudo nmap -Pn -p 1-1000 -T4 -A [PROD-IP]
Scan completed in 121 seconds. GuardDuty detected:

Recon:EC2/PortProbeUnprotectedPort — Severity: MEDIUM
Policy:IAMUser/RootCredentialUsage — detected from earlier AWS CLI configuration

Step 8 — Incident Response and Containment
Investigated attack using journalctl system logs. Created and deployed CloudGuard-Prod-Protection-NACL:

Inbound Rule 100: Deny TCP ports 1–1000 from Dev IP
Inbound Rule 200: Allow all other traffic
Outbound Rule 100: Allow all

Associated NACL with Production subnet. Validated by re-running nmap — scan blocked. Produced formal Incident Report SEC-2025-001.


Key Results:

Reduced incident detection and response time from hours to seconds through full automation
Successfully detected and contained a simulated port scan attack using GuardDuty and VPC NACLs
Delivered a complete incident response lifecycle — detection, investigation, containment, and documentation
Built across two isolated environments (Dev and Prod) simulating real financial services infrastructure



Skills Demonstrated:

Cloud monitoring and observability (CloudWatch, CloudWatch Agent)
Serverless automation and event-driven architecture (Lambda, SNS)
Cloud security and threat detection (GuardDuty)
Incident response and network containment (VPC NACLs)
IAM least-privilege design
Linux system administration and log investigation


Author
Rohan — Cloud Support Engineer
