---
title: "Week 8 Worklog"
date: 2026-06-12
weight: 8
chapter: false
pre: " <b> 1.8. </b> "
---

### Week 8 Objectives:

* Understand domain name management and routing configurations using Amazon Route53.
* Map domain records pointing to cloud properties like S3 hosted websites or EC2 instances.
* Design and construct an integrated hybrid DNS architecture forwarding queries between a simulated Local corporate network and AWS VPC.
* Develop browser-to-backend WebSocket streaming for PCM audio and integrate Amazon Transcribe Streaming.

### Tasks to be carried out this week:
| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 2 | - Study DNS fundamentals: Hosted Zones (Public vs. Private), record mapping types (A, CNAME, ALIAS, TXT), and Route53 routing algorithms | 06/08/2026 | 06/08/2026 | <https://000010.awsstudygroup.com/> |
| 3 | - Learn about hybrid cloud integrations and the challenge of cross-network DNS resolution between VPCs and local host servers | 06/09/2026 | 06/09/2026 | <https://000010.awsstudygroup.com/> |
| 4 | - **Practice:** Create a Private Hosted Zone matching a test internal domain, map resource records, and query from EC2 | 06/10/2026 | 06/10/2026 | <https://000010.awsstudygroup.com/> |
| 5 | - Study Route53 Resolver Endpoints: Structure of Inbound vs. Outbound Endpoints, and DNS forwarding rule setups | 06/11/2026 | 06/11/2026 | <https://000010.awsstudygroup.com/> |
| 6 | - **Practice:** Deploy Route53 Inbound and Outbound Endpoints, establish Resolver Rules, and verify cross-network address resolution from local server simulators | 06/12/2026 | 06/12/2026 | <https://000010.awsstudygroup.com/> |
| 6 | - **Capstone Project:** Develop AudioWorklet for 16kHz PCM browser recording and implement FastAPI WebSocket connection to Amazon Transcribe Streaming | 06/12/2026 | 06/12/2026 | <https://cloudjourney.awsstudygroup.com/> |

### Week 8 Achievements:

* Created Public and Private DNS Hosted Zones using Amazon Route53.
* Configured records pointing to web properties, resolving domain names directly into target compute endpoints.
* Deployed Route53 Inbound and Outbound Resolver Endpoints to support name resolution between simulated Local systems and AWS.
* Verified DNS query routing bi-directionally across the hybrid network boundaries, solving DNS loop problems.
* Built a working real-time speech-to-text pipeline utilizing AudioWorklet and Amazon Transcribe Streaming.
