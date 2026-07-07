---
title: "Week 2 Worklog"
date: 2026-05-01
weight: 2
chapter: false
pre: " <b> 1.2. </b> "
---

### Week 2 Objectives:

* Understand identity federations, user access scopes, and authorization protocols in AWS.
* Learn and apply AWS Identity and Access Management (IAM) configurations including Users, Groups, Policies, and Roles.
* Practice setting up resource boundaries and secure EC2 instance profiling to access cloud services.

### Tasks to be carried out this week:
| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 2 | - Study AWS IAM fundamentals: Key concepts of identity controls, Least Privilege principles, and credentials | 04/27/2026 | 04/27/2026 | <https://000002.awsstudygroup.com/> |
| 3 | - Analyze IAM Policy structure: JSON elements (Effect, Principal, Action, Resource, Condition parameters) | 04/28/2026 | 04/28/2026 | <https://000002.awsstudygroup.com/> |
| 4 | - **Practice:** Create IAM Users (Developer, Tester), organize them into IAM Groups, and bind restricted policies | 04/29/2026 | 04/29/2026 | <https://000002.awsstudygroup.com/> |
| 5 | - Learn about temporary security credentials and IAM Roles for AWS Services (EC2 Instance Profiles) | 04/30/2026 | 04/30/2026 | <https://000048.awsstudygroup.com/> |
| 6 | - **Practice:** Create an IAM Role with S3 ReadOnly permissions, attach it to an EC2 instance, and test S3 access | 05/01/2026 | 05/01/2026 | <https://000048.awsstudygroup.com/> |

### Week 2 Achievements:

* Deeply understood security delegation best practices, resolving user privilege overrides under standard operating principles.
* Formulated custom JSON policies from scratch, restricting data access dynamically based on action variables.
* Successfully decoupled static Access Keys from server instances by deploying EC2 Instance Profiles for secure AWS API authorization.
* Verified security bounds by attempting file updates from an instance having only S3 read-only roles, proving the execution of privilege boundaries.
