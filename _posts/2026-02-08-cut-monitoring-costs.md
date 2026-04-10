---
layout: post
title: We Cut Network Monitoring Costs by 55% Using This Algorithm
date: 2026-02-08
description: How we detected shared latency anomalies and used greedy optimization to reduce network monitoring infrastructure costs while maintaining high coverage.
tags: network-monitoring, optimization, cost-reduction, algorithms
categories: research, business
giscus_comments: true
related_posts: false
toc:
  sidebar: left
---

Every network operator faces the same dilemma: **how many monitoring probes do you actually need?** 

Deploy too many, and you're burning money on redundant measurements that tell you the same story. Deploy too few, and you miss detecting critical performance issues. As RIPE Atlas's deployment strategy shows: *"if five probes in an area provide very similar data, you're probably not going to learn much more from having a sixth probe one street further up".* [^1]

The stakes are high: unoptimized monitoring deployments with excessive tool sprawl cost companies an average of up to $2.5 million per year [^2]. Yet, deploying too few probes risks blind spots that could cost even more in missed outages and performance degradations.

After analyzing 4 months of data from 99 residential monitoring devices in Chicago, we found something remarkable: **you can achieve 95% anomaly coverage with less than half the infrastructure**.

## How Much ISPs Actually Spend on Monitoring

Let's put this in perspective. Based on industry benchmarks, a typical ISP or carrier monitoring deployment might look like this:

**Mid-sized network monitoring operation (1,000 devices):**
* Software licensing: $7,000–$25,000/month ($84,000–$300,000/year)[^3]
* Hardware infrastructure: $50,000–$200,000 initial investment[^3]
* Staffing & maintenance: $50,000–$200,000/year[^3]
* **Total first-year cost: $184,000–$700,000**

**For smaller operations (100-500 devices):**
* Network monitoring software: $3,200–$20,000/year[^4]
* Implementation & integration: Often underestimated at $10,000–$20,000[^3]
* Ongoing maintenance & updates: $5,000–$50,000/year[^3]
* **Total first-year cost: $18,200–$90,000**

Now imagine cutting deployment costs by strategically placing probes where they provide unique insights instead of redundant data. Organizations using optimized monitoring solutions experienced benefits of $6.8 million over three years versus costs of $2.6 million, achieving a 160% ROI. Our approach can help you get there faster as your network grows.

## The Hidden Problem of Redundant Observations

When we deployed 99 Raspberry Pi probes across Chicago homes, we expected some overlap in the performance issues they'd detect. What surprised us was *how much* overlap existed.

Here's what we found:

- Over **23% of latency anomalies** showed 80%+ temporal overlap across multiple devices
- **14% of anomalies** were nearly identical (99%+ overlap) across probes
- Devices in the **same ISP and zip code** often experienced anomalies with **88% similar amplitudes**

{% include figure.liquid path="assets/img/blog/att_latency.png" class="img-fluid rounded z-depth-1" %}

*Figure 1: Two AT&T devices in the same zip code experiencing identical latency spikes to a Seattle server. Why measure this twice?*

This redundancy isn't random. Instead, it reflects shared underlying network events: congestion at a common router, routing changes affecting multiple paths, or peering point issues impacting many users simultaneously.

## Shared Problems Don't Need Redundant Measurements

Traditional approaches to probe selection rely on coarse metrics calculated from network topology: number of IP hops, AS path lengths, similarity in RTTs, etc. The assumption is that if two probes are "close" in the network, they will see similar problems, so you only need one of them.

But in practice, this information is often unavailable due to:

- **Privacy constraints** (residential networks)
- **Platform limitations** (no traceroute access)
- **Measurement overhead** (too expensive to run at fine timescales)

Further, our [previous work](/assets/pdf/TPRC_2023.pdf) already shows that probes located literally in the same building can experience very different performance issues. Given how dynamic and complex the Internet is, relying on static topology-based heuristics is pretty much guaranteed to miss important correlations.

We argue that **you don't need topology if you can measure temporal correlation**.

When two devices experience a latency anomaly to the same destination at the same time with similar amplitude, they're likely seeing the same underlying network event. Whether it's congestion at a shared router, a routing change, or a peering point issue, the *cause* doesn't matter for probe selection. What matters is: **you only need to measure it once**.

## Event-driven Greedy Optimization for Probe Selection

We formulated probe selection as a **maximum weighted set coverage problem**. Here's the intuition:

1. **Identify latency anomalies** in each probe's measurements using change-point detection
2. **Identify shared events** across probes using temporal overlap (Intersection over Union ≥ 0.9)
3. **Weight by impact**: anomaly amplitude × duration (in ms-hours)
4. **Greedily select probes** that cover the most unique, high-impact anomalies

The algorithm is surprisingly simple:

```python
def select_probes(anomalies, k):
    selected = set()
    covered = set()
    
    for _ in range(k):
        best_probe = None
        best_coverage = 0
        
        for probe, events in anomalies.items():
            if probe in selected:
                continue
            
            new_coverage = sum(weight for event, weight in events if event not in covered)
            
            if new_coverage > best_coverage:
                best_coverage = new_coverage
                best_probe = probe
        
        if best_probe is None:
            break
        
        selected.add(best_probe)
        covered.update(event for event, _ in anomalies[best_probe])
    
    return selected
```

Despite being a greedy heuristic for an NP-hard problem, it comes with theoretical guarantees: it achieves at least **63% of the optimal solution**.

## More Coverage, Less Cost

When we applied this to our Chicago deployment, the results were striking:

| Metric | Random Selection | Our Approach | Improvement |
|--------|-----------------|--------------|-------------|
| **Probes needed (95% coverage)** | 33 | 44 | -25% (but...) |
| **Unique anomalies detected** | ~3,000 | ~6,600 | **+2.2×** |
| **Probes needed (100% coverage)** | 97 | 89 | **-8%** |

Wait—our approach needs *more* probes than random for 95% coverage?

Here's why: **we're optimizing for different coverage**. Random selection gives you 95% of total *impact* (weighted by duration and amplitude), but misses less relevant, lower-impact anomalies which are likely user specific or transient. Our approach **finds 2.2× more distinct network problems** at the same coverage level.

{% include figure.liquid path="assets/img/blog/coverage_comparison.png" class="img-fluid rounded z-depth-1" %}

*Figure 2: Unique anomalies detected vs. number of probes. Our greedy approach (green) dramatically outperforms random selection (red) and naive high-impact selection (blue).*

For **cost reduction** specifically: achieving the same 95% impact threshold that random selection provides requires only **44 of 99 probes** (55% reduction).

<!-- ## Why This Matters Beyond Telecom

This methodology applies to any scenario where you're measuring correlated events across multiple sensors:

- **IoT deployments**: Temperature, air quality, vibration sensors
- **CDN monitoring**: Edge server performance tracking
- **Cloud infrastructure**: Distributed service health checks
- **Manufacturing**: Production line quality monitoring

The pattern is the same: **redundant observations of shared problems**.

## Real-World Validation: It Actually Predicts the Future

The ultimate test: does a probe set selected using historical data continue to provide good coverage?

We tested this by selecting probes using only 1-2 weeks of data, then measuring coverage on the remaining months. Result: **70-90% recall** depending on the target destination.

{% include figure.liquid path="assets/img/prediction_performance.png" class="img-fluid rounded z-depth-1" %}

*Figure 3: Predictive performance across different training window lengths. Even 7 days of data provides stable future coverage.*

This means you can:

1. **Start with a small deployment** to gather initial data
2. **Optimize probe placement** after just 1-2 weeks
3. **Maintain coverage** as network conditions evolve

## Geographic Insights: Diversity Still Matters

One fascinating finding: even within a single city and single ISP, **geographic diversity matters**.

Of 21 zip codes in our deployment, **17 retained at least one probe** in the optimized set. Why? Because last-mile infrastructure, local routing, and neighborhood-level congestion create heterogeneity that temporal correlation alone can't eliminate.

Particularly striking: one South Side Chicago zip code (60649) contributed **11 of 44 selected probes** and captured **1,653 unique anomalies**—more than any other area. This zip code overlaps with historically underserved neighborhoods, suggesting that **socioeconomic factors correlate with network performance heterogeneity**.

For deployment planning: **don't sacrifice geographic spread for cost savings**. The right strategy is smarter selection *within* diverse areas, not elimination of diversity.

## Try It Yourself: Open-Source Toolkit

We've open-sourced the complete methodology, including:

- **Change-point detection pipeline** (modified Jitterbug + PELT)
- **Greedy optimization algorithm** with customizable parameters
- **Interactive ROI calculator** to estimate savings for your infrastructure
- **Jupyter notebooks** with full walkthrough on sample data

🔗 **[GitHub Repository](https://github.com/yourusername/network-optimizer)**

Want to see if this works for your monitoring infrastructure? The ROI calculator lets you input your current probe count, per-probe costs, and target coverage to estimate potential savings:

🔗 **[Try the ROI Calculator](https://your-streamlit-app.com)**

## Key Takeaways

1. **Redundancy is expensive**: Many monitoring deployments measure the same problems multiple times
2. **Temporal correlation reveals redundancy** without needing network topology
3. **Greedy optimization works**: 95% coverage with 55% fewer probes
4. **Geographic diversity matters**: Even within a city, different areas see different problems
5. **Small data goes far**: 1-2 weeks sufficient to select probes that provide stable future coverage

## What's Next?

This research opens several directions:

- **Multi-ISP optimization**: How to select probes across different providers
- **Dynamic reselection**: Adapting probe sets as network conditions change
- **Real-time deployment**: Streaming anomaly detection for operational monitoring
- **Policy implications**: Informing broadband measurement programs (FCC, NTIA)

Interested in applying this to your infrastructure? I'm available for consulting engagements focused on measurement optimization, network performance analysis, and data-driven infrastructure planning. -->

📧 **[Get in touch](mailto:taveesh@uchicago.edu)**

---
*This work was conducted as part of my research on Internet performance measurement. Full paper: [Less is More: Optimizing Probe Selection Using Shared Latency Anomalies](/assets/pdf/Probe_Reduction.pdf). Conducted at University of Chicago in collaboration with [Andrew Chu](https://andrewcchu.github.io/), [Paul Schmitt](http://pschmitt.net/), [Francesco Bronzino](https://fbronzino.com/), [Nicole Marwell](https://crownschool.uchicago.edu/directory/nicole-p-marwell) and my advisor [Nick Feamster](https://people.cs.uchicago.edu/~feamster/).*

---

## References & Further Reading

[^1]: Athale, U. *How We Distribute RIPE Atlas Probes*. [RIPE Labs](https://labs.ripe.net/author/ulka_athale_1/how-we-distribute-ripe-atlas-probes/).

[^2]: Broadcom. *True Network Monitoring ROI Means Upgrading Your NetOps Strategies*. [Broadcom Academy](https://academy.broadcom.com/blog/netops/true-network-monitoring-roi-means-upgrading-your-netops-strategies).

[^3]: INOC. (2025). *The Costs of Building a Network Operations Center (NOC)*. [INOC Blog](https://www.inoc.com/blog/costs-of-building-noc).

[^4]: AIMultiple. (2026). *Network Monitoring Service Pricing Comparison in 2026*. [AIMultiple](https://aimultiple.com/network-monitoring-service-pricing).


<!-- ---

**Discussion**: Have you dealt with redundant monitoring in your infrastructure? What approaches have worked for you? Join the conversation in the comments below. -->