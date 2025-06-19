---
layout: post
title: "Be Cautious with AWS EFS Pricing Model"
date: 2025-06-19
categories: blog
tags: [AWS, cost-optimization, bioinformatics, cloud-computing, devops]
---

This experience highlights one of my strengths—**cost awareness and proactive optimization in cloud infrastructure**. By identifying inefficiencies in AWS EFS's pricing model, I was able to reduce the cost of running our bioinformatic pipeline from **~$260 per study plus storage (~$4/day)** to **just the baseline storage cost of ~$4/day** (*Figure 1*). This translates to an estimated **annual savings of over $7,000** for the company. While the number may seem modest in isolation, it reflects a mindset that compounds impact when applied across multiple areas of infrastructure.

---

## The Context

We recently completed initial development and deployment of a bioinformatic pipeline using **AWS ECS** and **EFS**, entering the testing phase shortly after. As someone who is consistently mindful of cloud spending, I monitored our AWS cost metrics closely during this stage.

To my surprise, EFS costs spiked to **~$90** for just a subset of a full test study—an unexpected and disproportionate expense. Further testing showed that this cost varied with each run and dropped to negligible amounts on idle days. Something wasn’t adding up.

---

## Diagnosing the Problem

Digging deeper, I found that our EFS file system was set to **Elastic Throughput mode**, which is AWS’s default and recommended configuration. However, this mode charges for both storage and **throughput (read/write I/O)**, making it expensive for workloads that involve large-scale, repetitive data access—like our sequence database searches (100–200 GB per query).

We store **390 GB** on EFS, and each full study accesses approximately **8,672 GB** of data. Using AWS's pricing model:

```text
Storage cost: 390 GB × $0.30/month ÷ 30 days = $3.90/day  
Throughput cost: 8,672 GB × $0.03/GB = ~$260 per study
```

That’s a significant operational cost per run.

## The Solution: Switch to Bursting Throughput Mode
Further analysis revealed that Bursting Throughput mode is much better suited to our workload. In this mode:
- Throughput is free as long as usage stays under a limit defined by the total storage size.
- Each GiB of data grants 50 KiBps of throughput, meaning our 390 GB allows up to ~19 MiBps.
- Even better, reads are discounted by a factor of 3, so 30 KiBps read is billed as just 10 KiBps, ideal for read-heavy workflows like ours.

CloudWatch confirmed that our actual throughput during search operations stayed well below the threshold (_Figure 2_). After switching to Bursting mode, our EFS costs dropped dramatically to just the daily storage fee—no additional charges for throughput.

## Takeaway
This was a clear win for cost-conscious engineering. By challenging the default AWS configuration and making data-driven adjustments, I helped eliminate unnecessary costs while preserving performance.

This experience reinforces my approach to infrastructure: never assume defaults are optimal, and always let data guide decisions. I'm proud of this optimization—not just for the savings, but for what it says about how I work.
