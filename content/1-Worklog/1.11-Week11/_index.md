---
title: "Week 11 Worklog"
date: 2026-07-03
weight: 11
chapter: false
pre: " <b> 1.11. </b> "
---

### Week 11 Objectives:

* Master cloud resource observability, metrics dashboard setups, and centralized logging via Amazon CloudWatch.
* Configure log shippers on servers to centralize application log outputs.
* Build automated alert notification rules using CloudWatch Alarms and Amazon Simple Notification Service (SNS).

### Tasks to be carried out this week:
| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 2 | - Study CloudWatch components: Host metrics, custom metrics, dashboards, alarms, and unified log collectors | 06/29/2026 | 06/29/2026 | <https://000008.awsstudygroup.com/> |
| 3 | - **Practice:** Install the unified CloudWatch Agent on EC2 instances, configure `config.json` rules, and start collecting memory/disk usage statistics | 06/30/2026 | 06/30/2026 | <https://000008.awsstudygroup.com/> |
| 4 | - **Practice:** Construct a custom CloudWatch Dashboard containing line and number widgets representing CPU, Memory, and Disk stats of your ASG instances | 07/01/2026 | 07/01/2026 | <https://000008.awsstudygroup.com/> |
| 5 | - **Practice:** Set up an Amazon SNS Topic, subscribe developer email, create a CloudWatch Alarm triggered at 80% CPU usage, and bind it to the SNS topic | 07/02/2026 | 07/02/2026 | <https://000008.awsstudygroup.com/> |
| 6 | - **Practice:** Configure CloudWatch Agent log rules to ship Apache access and system error logs to CloudWatch Log Groups, and write filter queries | 07/03/2026 | 07/03/2026 | <https://000008.awsstudygroup.com/> |

### Week 11 Achievements:

* Configured the Amazon CloudWatch Unified Agent on Linux server environments, exposing system resources metrics.
* Designed dynamic operational dashboards to visualize cross-infrastructure performance data under single views.
* Implemented automatic alerts using CloudWatch Alarms linked with SNS, sending email alerts to administrator endpoints under high CPU stress conditions.
* Centralized web application access/error logs inside CloudWatch Log Groups, enabling analysis using CloudWatch Logs Insights.
