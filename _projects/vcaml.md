---
layout: page
title: vcaml
description: An ML pipeline for passively estimating QoE for WebRTC applications.
img: /assets/img/webrtc.png
importance: 1
---

<img class="img-fluid rounded z-depth-1" src="{{ '/assets/img/taveesh_at_IMC.JPG' | relative_url }}" alt="" title="Presentation at Montreal, 2023"/>

In this work, we implemented a technique for helping network operators gain an increased visibility into the quality of video conferencing applications running over their networks. Our technique assumes no access to application layer headers and is able to use an untuned Random Forest regressor to accurately infer Quality of Experience (QoE) at 1-second granularity. We found an increased importance for features like *number of unique packet lengths*, and flow statistics over packet sizes and interarrival times in our trained models. Our technique generalized to three conferencing applications -- Google Meet, Microsoft Teams, and Cisco Webex running over in-lab as well as residential networks.

Our code, datasets and paper can be found [here](https://github.com/noise-lab/vcaml).