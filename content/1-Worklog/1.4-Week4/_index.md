---
title: "Week 4 Worklog"
date: 2026-05-15
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

### Week 4 Objectives:

* Understand AWS networking architectures and core routing models.
* Plan IP address allocations and deploy a custom Amazon Virtual Private Cloud (VPC).
* Implement subnet partitions (Public and Private), attach Internet Gateways (IGW), and configure NAT Gateways to support secure internet access.

### Tasks to be carried out this week:
| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 2 | - Study VPC networking theory: CIDR block subnetting formulas, IP address reservation, and public/private topologies | 05/11/2026 | 05/11/2026 | <https://000003.awsstudygroup.com/> |
| 3 | - Analyze routing concepts: Route tables, network boundaries, Internet Gateways (IGW), NAT Gateways, and Security Groups vs. NACLs | 05/12/2026 | 05/12/2026 | <https://000003.awsstudygroup.com/> |
| 4 | - **Practice:** Create a custom VPC with IP block 10.0.0.0/16 and partition it into 2 Public and 2 Private subnets across multiple Availability Zones | 05/13/2026 | 05/13/2026 | <https://000003.awsstudygroup.com/> |
| 5 | - **Practice:** Configure routing tables, attach an Internet Gateway, and deploy a NAT Gateway in the public subnet segment | 05/14/2026 | 05/14/2026 | <https://000003.awsstudygroup.com/> |
| 6 | - **Practice:** Deploy testing instances in both subnets and run route traces to verify that private instances can download updates through the NAT Gateway | 05/15/2026 | 05/15/2026 | <https://000003.awsstudygroup.com/> |

### Week 4 Achievements:

* Designed and built a custom VPC layout dividing network domains into secure subnets.
* Configured routing protocols separating external-facing web properties from internal backend assets.
* Implemented a NAT Gateway, allowing systems located inside private subnets to execute out-of-band updates without exposing their ports to incoming internet requests.
* Gained hands-on experience troubleshooting VPC network pathings, local routing configurations, and security firewall boundaries.
