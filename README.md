# Fault-tolerant-Auto-Scaling-cloud-application-
AWS project for my Cloud module deployed a web app on EC2, added an RDS database, then load balancing and auto scaling. Built across 3 iterations, fixing real issues along the way.
What it does:

A simple record-management app (add, edit, delete entries) deployed on AWS. The point of the project wasn't the app itself but the infrastructure underneath it, built up over three iterations:

Iteration 1 Got the app running on a single EC2 instance inside a custom VPC.
Iteration 2 Added an RDS (MySQL) database, with credentials pulled from Secrets Manager instead of hardcoded. This is where a missing IAM role caused the app to silently fall back to placeholder credentials took a while to track down.
Iteration 3 Put the app behind an Application Load Balancer with an Auto Scaling Group, locked down security groups so instances could only be reached through the load balancer, and built a CloudWatch dashboard to monitor it. Load-tested it and confirmed it scaled up under load and back down once traffic dropped.
Tools used
Amazon EC2
Amazon VPC (public/private subnets, route tables, security groups)
Amazon RDS (MySQL)
AWS Secrets Manager
AWS IAM
Application Load Balancer
Auto Scaling Groups
Amazon CloudWatch
GROUP MEMBERS:
X00221466 — Ivan Medel
X00219599 — Muhammad Abubaker
C23300041 — Daniel Ilesanmi
