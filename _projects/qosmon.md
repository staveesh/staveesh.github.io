---
layout: page
title: QoSMon
description: A real-time network measurement and monitoring system for community networks.
img: /assets/img/testbed.png
importance: 2
---

<img height="60%" class="img-fluid rounded z-depth-1" src="{{ '/assets/img/system.png' | relative_url }}" alt="" title="Roburse in action"/>

This project investigated the optimal design of distributed network measurement systems in a community network context. We first implemented five active measurement tests that could be executed from linux boxes and Android phones. Then, we implemented four different scheduling algorithms to minimize the contention between network and CPU resources for conflicting measurements. The later phase of the project involved empirical evaluation of the optimality of scheduling these measurements in under-resourced networks. We also developed a mininet-based SDN framework to demonstrate the system in a simulated environment.

Here's a [video](https://www.youtube.com/embed/jAK0qikvSPw?si=gIbssVXwaNCjf-0G) of me presenting QoSMon at UCT's School of IT Showcase in 2021.

Some glimpses of the monitoring system.

<img class="img-fluid rounded z-depth-1" src="{{ '/assets/img/viz1.png' | relative_url }}" alt=""/>
<img class="img-fluid rounded z-depth-1" src="{{ '/assets/img/viz3.png' | relative_url }}" alt=""/>
<img class="img-fluid rounded z-depth-1" src="{{ '/assets/img/viz4.png' | relative_url }}" alt=""/>
<img class="img-fluid rounded z-depth-1" src="{{ '/assets/img/viz2.png' | relative_url }}" alt=""/>