---
title: "Week 7 Worklog"
date: 2026-06-05
weight: 7
chapter: false
pre: " <b> 1.7. </b> "
---

### Week 7 Objectives:

* Understand AWS managed relational database options using Amazon Relational Database Service (RDS).
* Secure database clusters by hosting instances within private VPC subnets.
* Implement database subnet groups and configure custom firewall boundaries linking EC2 app servers directly to RDS.
* Kick off the LiveCap Capstone Project, analyzing requirements and sketching the system design.

### Tasks to be carried out this week:
| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 2 | - Study Amazon RDS architecture: Supported DB engines, database replication models, backup routines, and parameter groups | 06/01/2026 | 06/01/2026 | <https://000005.awsstudygroup.com/> |
| 3 | - Learn about database isolation patterns: Subnet groups, routing guidelines, Multi-AZ deployments, and network firewall policies | 06/02/2026 | 06/02/2026 | <https://000005.awsstudygroup.com/> |
| 4 | - **Practice:** Create a DB Subnet Group selecting the private subnets of your custom VPC deployed in Week 4 | 06/03/2026 | 06/03/2026 | <https://000005.awsstudygroup.com/> |
| 5 | - **Practice:** Launch an Amazon RDS database instance (MySQL/PostgreSQL) and configure credentials, snapshots, and maintenance hours | 06/04/2026 | 06/04/2026 | <https://000005.awsstudygroup.com/> |
| 6 | - **Practice:** Configure RDS Security Groups restricting port access exclusively to the EC2 instance, install a database client, and execute query commands | 06/05/2026 | 06/05/2026 | <https://000005.awsstudygroup.com/> |
| 6 | - **Capstone Project:** Kick off LiveCap, analyze requirements, and sketch the system data flow (WebSocket, PCM audio) | 06/05/2026 | 06/05/2026 | <https://cloudjourney.awsstudygroup.com/> |

### Week 7 Achievements:

* Deployed and configured a secure relational database instance using Amazon RDS.
* Successfully isolated the database cluster inside VPC Private Subnets, protecting backend data resources from direct public route access.
* Implemented cross-tier Security Group references, restricting MySQL port 3306 queries to web-application instances only.
* Installed database clients on EC2 and verified successful client authentication and schema operations on the remote RDS instance.
* Initiated the LiveCap project, defining core requirements and proposing the WebSocket communication flow.
