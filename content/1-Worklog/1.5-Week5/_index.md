---
title: "Week 5 Worklog"
date: 2026-05-22
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---

### Week 5 Objectives:

* Master virtual server computing concepts on AWS using Amazon Elastic Compute Cloud (EC2).
* Deploy, configure, and secure Linux web application servers in custom subnet environments.
* Learn about block storage drives, practice creating, formatting, and attaching Amazon EBS volumes.

### Tasks to be carried out this week:
| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 2 | - Study EC2 service architecture: Instance families (General Purpose, Compute, Memory), AMI selection, and purchasing models | 05/18/2026 | 05/18/2026 | <https://000004.awsstudygroup.com/> |
| 3 | - **Practice:** Launch a Linux EC2 instance (t2.micro/t3.micro) inside the public subnet of the custom VPC created in Week 4 | 05/19/2026 | 05/19/2026 | <https://000004.awsstudygroup.com/> |
| 4 | - **Practice:** Configure Security Groups to allow SSH (port 22) and HTTP (port 80) traffic, download key pairs, and connect to the instance | 05/20/2026 | 05/20/2026 | <https://000004.awsstudygroup.com/> |
| 5 | - Study Elastic Block Store (EBS) volumes types (gp2, gp3, io2) and practice creating a 10GB volume and attaching it to the instance | 05/21/2026 | 05/21/2026 | <https://000004.awsstudygroup.com/> |
| 6 | - **Practice:** Partition, format (ext4), and mount the new EBS volume to a custom Linux folder, install Apache, and host a test site | 05/22/2026 | 05/22/2026 | <https://000004.awsstudygroup.com/> |

### Week 5 Achievements:

* Successfully launched and managed Amazon EC2 instances within a designated custom VPC subnet.
* Implemented host security boundaries by defining fine-grained inbound rules on Security Groups.
* Created and managed persistent block storage volumes using Amazon EBS.
* Gained experience operating with Linux storage commands (`lsblk`, `mkfs`, `mount`, and writing to `/etc/fstab` for auto-mounting).
* Deployed an Apache web server serving web traffic from the newly mounted block storage drive.
