---
title: "Frontend Delivery and Real-Time Integrations"
date: 2026-07-05
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

# Frontend Delivery and Real-Time Integrations

The LiveCap frontend is a React 18, TypeScript, Vite, Tailwind CSS, and GSAP
application. `/` is a product landing page with scroll-driven presentation;
`/app` is the lightweight caption dashboard used during live sessions.

CloudFront is the single browser entrypoint. It serves static assets from a
private S3 bucket through OAC and routes `/api/*` and `/ws/*` to the ALB. The
backend then coordinates Transcribe, Translate, transcript export, and session
cleanup.

Heavy animation is limited to the landing page. The live dashboard prioritizes
stable transcript rendering, microphone controls, connection status, and the
session timer.
