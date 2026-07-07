---
title: "Week 10 Worklog"
date: 2026-06-26
weight: 10
chapter: false
pre: " <b> 1.10. </b> "
---

### Week 10 Objectives:

* Understand system scalability, load balancing, and high-availability design patterns in AWS.
* Deploy Application Load Balancers (ALB) and set up backend target groups with health checks.
* Implement dynamic Auto Scaling Groups (ASG) and launch template automation.

### Tasks to be carried out this week:
| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 2 | - Study scaling models: Horizontal vs. vertical scaling, manual capacity adjustment vs. metric-based auto-scaling configurations | 06/22/2026 | 06/22/2026 | <https://000006.awsstudygroup.com/> |
| 3 | - Learn the roles of Application Load Balancers (ALB), Target Groups, health checks, and Launch Templates | 06/23/2026 | 06/23/2026 | <https://000006.awsstudygroup.com/> |
| 4 | - **Practice:** Create an ALB and configure a Target Group mapping health check parameters (ports, endpoints, retry thresholds) | 06/24/2026 | 06/24/2026 | <https://000006.awsstudygroup.com/> |
| 5 | - **Practice:** Create an EC2 Launch Template with custom bash script User Data to automate Apache web server installation | 06/25/2026 | 06/25/2026 | <https://000006.awsstudygroup.com/> |
| 6 | - **Practice:** Create an Auto Scaling Group (ASG) bound to target subnets, set scaling thresholds, and run stress tests to trigger scale-out | 06/26/2026 | 06/26/2026 | <https://000006.awsstudygroup.com/> |

### Week 10 Achievements:

* Built a load-balanced architecture using an Application Load Balancer to route traffic and execute backend health validation.
* Deployed Launch Templates containing script automations (User Data) to launch configured web servers automatically.
* Implemented an Auto Scaling Group that dynamically scales instance counts up and down based on target CPU utilization boundaries.
* Tested failover and recovery behavior by manually stopping instances, observing the ASG spin up replacement machines automatically.
