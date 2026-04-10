---
layout: post
title: The Geography of Network Performance - What 99 Homes Taught Us
date: 2026-02-08
description: Why geographic diversity matters for network monitoring, even within a single city and ISP. Lessons from analyzing 99 residential devices across Chicago neighborhoods.
tags: digital-divide, network-planning, geographic-analysis, infrastructure
categories: research, policy
giscus_comments: true
related_posts: true
toc:
  sidebar: left
---

When we set out to optimize our network monitoring deployment, we expected to find redundancy. What we didn't expect was how *geographically* concentrated the problems would be.

**One zip code—out of 21—accounted for 25% of selected probes and 25% of unique anomalies.**

This wasn't random. This zip code (60649) overlaps with Chicago's South Shore neighborhood—a historically underserved, predominantly Black community with documented infrastructure challenges.

The data revealed something that policy documents often obscure: **the digital divide exists at hyperlocal scales, even within the same ISP**.

## The Surprising Finding: You Can't Skip Zip Codes

After running our optimization algorithm ([described in Post 1](link)), we expected the selected probe set to cluster around a few "representative" areas. Instead:

- **17 of 21 zip codes** (81%) retained at least one probe
- **Geographic spread was preserved** despite 55% probe reduction
- **Diversity remained critical** for capturing unique network issues

{% include figure.liquid path="assets/img/geographic_distribution.png" class="img-fluid rounded z-depth-1" %}

*Figure 1: Original vs. optimized probe distribution across Chicago zip codes. Colors indicate probe counts.*

Why does this matter? Because it suggests that **last-mile infrastructure heterogeneity is the norm, not the exception**—even when all devices connect through the same ISP.

## What Makes South Shore Different?

Let's zoom into zip code 60649:

| Metric | 60649 (South Shore) | Median Across All Zip Codes |
|--------|---------------------|----------------------------|
| **Original probes** | 17 | 2 |
| **Selected probes** | 11 | 1 |
| **Unique anomalies** | 1,653 | 270 |
| **Median download speed** | 284 Mbps | 347 Mbps |
| **Last-mile latency (min)** | 8.2 ms | 5.1 ms |

This area retained **65% of its probes** (11 of 17), compared to a 45% retention rate citywide.

**Translation**: Network performance in South Shore is more heterogeneous and less predictable than in other parts of Chicago. Removing probes here would blind you to problems you wouldn't see elsewhere.

### Demographic Context

South Shore's population is:
- **94% Black** (vs. 29% citywide)
- **Median household income**: $35,000 (vs. $65,000 citywide)
- **Broadband adoption**: 73% (vs. 83% citywide)

[Source: U.S. Census Bureau, 2020]

This isn't coincidence. Prior research has documented infrastructure underinvestment in historically redlined neighborhoods. Our data provides **network-level evidence** of this disparity.

## The Three Scales of Geographic Granularity

We tested probe selection at three geographic levels:

### 1. Census Tracts (Finest Granularity)

- **Total tracts in deployment**: 43
- **Tracts retaining ≥1 probe**: 1 (2.3%)

**Verdict**: Too fine-grained. Most tracts don't have enough diversity to justify dedicated monitoring.

### 2. Neighborhoods (Medium Granularity)

Chicago has 77 official community areas. Our deployment covered 17.

- **Neighborhoods retaining ≥1 probe**: 4.8%

**Verdict**: Better than census tracts, but still sparse. Good for understanding *which* communities differ, but not for optimal probe placement.

### 3. Zip Codes (Recommended Granularity)

- **Zip codes retaining ≥1 probe**: 81%

**Verdict**: ✅ **Sweet spot**. Large enough to capture meaningful heterogeneity, small enough to detect localized issues.

{% include figure.liquid path="assets/img/granularity_comparison.png" class="img-fluid rounded z-depth-1" %}

*Figure 2: Fraction of geographic units retaining at least one probe after optimization.*

### Why Zip Codes Work

1. **IP geolocation databases** (MaxMind, IPinfo) provide zip-level precision
2. **ISP infrastructure** often aligns with zip code boundaries (cable plant, CO locations)
3. **Policy and planning** already use zip codes (FCC broadband maps, NTIA grants)

**Practical implication**: If you're designing a new measurement deployment, **start with at least one probe per zip code** in your target area.

## Distance Effects: Where Anomalies Get Shared

We measured how the *destination* of measurements affects sharing patterns:

{% include figure.liquid path="assets/img/distance_sharing.png" class="img-fluid rounded z-depth-1" %}

*Figure 3: Fraction of shared anomaly impact vs. destination distance from Chicago.*

### Key Observations:

1. **Local targets** (Chicago M-Lab): 45% shared impact
   - **Why**: Diverse routing even within metro area

2. **Mid-range targets** (Denver, Seattle): 55-60% shared impact  
   - **Why**: Paths converge at regional peering points

3. **Distant targets** (Stockholm): **70% shared impact** ⬅ Peak sharing
   - **Why**: Maximum path convergence (all routes funnel through same international links)

4. **Very distant targets** (Johannesburg): 50% shared impact
   - **Why**: Fewer total anomalies detected (high baseline RTT masks small increases)

**Implication**: If your goal is to *minimize* probe count, measure to mid-distance targets (500-2000 km). Too close = too diverse. Too far = too noisy.

## The Pre-Deployment Planning Problem

**Scenario**: You want to deploy 50 probes across a city. How do you choose *where*?

Traditional approach:
- Uniform geographic distribution
- Sample ISP market share
- Target "representative" demographics

**Our data-driven approach:**

### Step 1: Identify High-Variance Areas

Use crowdsourced data (M-Lab, Ookla) to find zip codes with:
- High coefficient of variation in throughput
- Wide interquartile range in latency
- Frequent speed tier mismatches (provisioned vs. actual)
```python
def identify_high_variance_areas(crowdsourced_data):
    """
    Find zip codes with heterogeneous performance
    """
    variance_by_zip = {}
    
    for zipcode in crowdsourced_data.zipcodes:
        measurements = crowdsourced_data[zipcode]
        
        # Calculate variability
        cv_throughput = measurements.download_mbps.std() / measurements.download_mbps.mean()
        iqr_latency = measurements.latency_ms.quantile(0.75) - measurements.latency_ms.quantile(0.25)
        
        # Score combines multiple variance signals
        score = cv_throughput * np.log(1 + iqr_latency)
        variance_by_zip[zipcode] = score
    
    # Select top N% highest variance
    threshold = np.percentile(list(variance_by_zip.values()), 75)
    high_variance_zips = [z for z, s in variance_by_zip.items() if s > threshold]
    
    return high_variance_zips
```

### Step 2: Allocate Probes Proportionally

Don't just put one probe per zip code. Weight by expected heterogeneity:
```python
def allocate_probes(zipcode_scores, total_budget=50):
    """
    Distribute probes based on variance scores
    """
    # Normalize scores to probabilities
    total_score = sum(zipcode_scores.values())
    probabilities = {z: s/total_score for z, s in zipcode_scores.items()}
    
    # Allocate proportionally (with minimum 1 per zip)
    allocation = {}
    remaining_budget = total_budget - len(zipcode_scores)  # Reserve 1 per zip
    
    for zipcode, prob in probabilities.items():
        allocation[zipcode] = 1 + int(prob * remaining_budget)
    
    return allocation
```

### Step 3: Refine After Short Collection Period

After 1-2 weeks:
- Run our anomaly detection pipeline
- Identify redundant probes (high IoU, similar amplitude)
- Reallocate to under-monitored areas

**Result**: Data-driven deployment that adapts to real performance patterns.

## Interactive Tool: Pre-Deployment Planner

We built a Streamlit app that implements this workflow:

🔗 **[Try the Planner](https://your-planner-app.com)**

**Features**:
- Upload M-Lab/Ookla CSV data
- Visualize variance by zip code
- Get recommended probe allocation
- Simulate coverage under different budgets

{% include figure.liquid path="assets/img/planner_screenshot.png" class="img-fluid rounded z-depth-1" %}

*Figure 4: Screenshot of the pre-deployment planner showing variance heatmap and recommended allocations.*

## Policy Implications: Informing Broadband Measurement Programs

These findings have direct relevance for federal programs:

### FCC Broadband Data Collection (BDC)

The FCC requires ISPs to report coverage at granular levels. But **coverage ≠ performance**.

**Recommendation**: Supplement BDC with **performance diversity metrics** at zip-code level. High-variance areas should trigger enhanced scrutiny or verification testing.

### NTIA BEAD Program ($42.5B for broadband expansion)

States must submit plans for identifying unserved/underserved areas.

**Recommendation**: Use anomaly heterogeneity as a **signal for infrastructure investment prioritization**. Areas with high intra-zip variance may need targeted upgrades, not just capacity expansion.

### Measurement Lab & Academic Research

M-Lab's open data is invaluable, but lacks spatial coverage in many cities.

**Recommendation**: Partner with community organizations in high-variance areas (like South Shore) to deploy dedicated measurement infrastructure. Current sampling may **systematically under-represent** neighborhoods with the most heterogeneous—and likely worst—performance.

## Key Takeaways for Infrastructure Planners

1. **Geographic diversity is non-negotiable**: Even within single city + single ISP, you need broad coverage

2. **Zip codes are the right granularity**: Balance between specificity and practicality

3. **Expect heterogeneity in underserved areas**: Allocate more probes where you see high performance variance

4. **Target mid-distance destinations**: 500-2000 km from your deployment for maximum shared anomaly detection

5. **Use crowdsourced data for planning**: M-Lab/Ookla can guide initial probe placement

6. **Refine after 1-2 weeks**: Short collection period sufficient to optimize deployment

## What We're Building Next

Based on these insights, we're developing:

1. **Automated deployment optimizer**: Input budget + city → get probe allocation plan
2. **Real-time reallocation algorithm**: Dynamically move probes based on detected heterogeneity
3. **Policy dashboard**: Translate anomaly data into actionable infrastructure investment recommendations

Interested in collaborating or applying this to your city? Reach out.

---

**Discussion**: Have you observed similar geographic performance heterogeneity in your network measurements? How do you decide where to place monitoring infrastructure?

📧 **[Contact me](mailto:your.email@domain.com)** | 🔗 **[Try the Planner](https://your-planner-app.com)** | 💻 **[View Code](https://github.com/yourusername/network-optimizer)**

---

## References & Data Sources

- U.S. Census Bureau, American Community Survey (2020)
- FCC Broadband Data Collection: [https://www.fcc.gov/BroadbandData](https://www.fcc.gov/BroadbandData)
- Measurement Lab: [https://www.measurementlab.net](https://www.measurementlab.net)
- Digital redlining research: Saxon & Black (2022), "Measuring Internet Inequality in Chicago"