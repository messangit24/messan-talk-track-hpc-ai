---
marp: true
html: true
theme: google-public-sector
paginate: true
title: HPC + AI on Google Cloud
---

<!-- _class: title -->

# Beyond the Queue

## HPC + AI on Google Cloud

Willis Zhang · Google Cloud

---

## Current conditions

<div class="hero">
  <div class="num">$190B</div>
  <div class="label">Alphabet 2026 CapEx, mostly AI</div>
  <div class="sub"><strong><a href="https://cloud.google.com/ai-infrastructure">90%</a></strong> of generative AI unicorns run on Google Cloud AI infrastructure.</div>
</div>

---

<!-- _class: section -->

# NVIDIA on Google Cloud

---

<!-- _class: compact -->

## Every cloud has H100s. We add this:

<div class="cards">
  <div class="card"><div class="name"><a href="https://cloud.google.com/blog/products/networking/introducing-virgo-megascale-data-center-fabric">Optical Circuit Switching</a></div><div class="note">Failed chip → job survives, no restart.</div></div>
  <div class="card yellow"><div class="name"><a href="https://docs.cloud.google.com/ai-hypercomputer/docs/workloads/enable-node-health-prediction">Node Health Prediction</a></div><div class="note">ML drains nodes 5h before degradation.</div></div>
  <div class="card green"><div class="name"><a href="https://docs.cloud.google.com/kubernetes-engine/docs/how-to/machine-learning/training/multi-tier-checkpointing">Multi-Tier Checkpointing</a></div><div class="note">RAM → peer node → Cloud Storage recovery.</div></div>
  <div class="card green"><div class="name"><a href="https://docs.cloud.google.com/cluster-director/docs/orchestration">Topology-aware Slurm</a></div><div class="note">Cluster Director exposes sub-block metadata to Slurm.</div></div>
  <div class="card red"><div class="name"><a href="https://docs.cloud.google.com/ai-hypercomputer/docs/networking-overview">Rail-aligned networking</a></div><div class="note">Predictable hops at thousand-GPU scale.</div></div>
  <div class="card"><div class="name"><a href="https://cloud.google.com/blog/products/ai-machine-learning/goodput-metric-as-measure-of-ml-productivity">Goodput</a></div><div class="note">Public metric for ML productivity.</div></div>
</div>

---

## and also storage

<div class="cards">
  <div class="card yellow"><div class="name"><a href="https://docs.cloud.google.com/managed-lustre/docs/overview">Managed Lustre</a></div><div class="stat">10 TB/s</div></div>
  <div class="card"><div class="name"><a href="https://cloud.google.com/kubernetes-engine/docs/how-to/persistent-volumes/cloud-storage-fuse-csi-driver">Cloud Storage as POSIX</a></div><div class="stat">9×<span class="unit">model loads</span></div><div class="stat">30×<span class="unit">checkpoint writes</span></div></div>
  <div class="card green"><div class="name"><a href="https://docs.cloud.google.com/storage/docs/rapid/rapid-cache">Rapid Storage</a></div><div class="stat">6 TB/s</div></div>
</div>

---

<!-- _class: section yellow -->

# How you consume

---

## Capacity without a commit

<div class="cards four">
  <div class="card yellow"><div class="name"><a href="https://docs.cloud.google.com/kubernetes-engine/docs/concepts/dws">DWS Flex Start</a></div><div class="note">7-day guaranteed. No contract.</div></div>
  <div class="card"><div class="name"><a href="https://docs.cloud.google.com/compute/docs/instances/future-reservations-calendar-mode-overview">Calendar Mode</a></div><div class="note">Reserve specific date windows up to 90 days out.</div></div>
  <div class="card red"><div class="name"><a href="https://github.com/WandLZhang/slurm-multi-region-gpu">Multi-region Spot</a></div><div class="note">Thousands of chips across CONUS. Preempt → resume elsewhere.</div></div>
  <div class="card green"><div class="name"><a href="https://cloud.google.com/kubernetes-engine/docs/concepts/fast-starting-nodes">BoltVMs</a></div><div class="note">H100 cold-start in ~2 minutes.</div></div>
</div>

---

<!-- _class: compact -->

## Reimagine what a node is

<div class="feature-list">
  <div class="feature-item"><div class="fname"><a href="https://docs.cloud.google.com/batch/docs/nextflow">Nextflow on Cloud Batch</a></div><div class="fdesc">DWS Flex guarantees GPUs for nf-core pipelines.</div></div>
  <div class="feature-item red"><div class="fname"><a href="https://docs.cloud.google.com/compute/docs/accelerator-optimized-machines#g4-series">G4 fractional GPUs</a></div><div class="fdesc">1/8, 1/4, or 1/2 of an RTX PRO 6000 Blackwell.</div></div>
  <div class="feature-item green"><div class="fname"><a href="https://docs.cloud.google.com/kubernetes-engine/docs/how-to/tpus">TPU + GPU in one cluster</a></div><div class="fdesc">Mixed accelerators, single GKE cluster.</div></div>
  <div class="feature-item yellow"><div class="fname"><a href="https://docs.cloud.google.com/kubernetes-engine/docs/how-to/image-streaming">Image Streaming</a></div><div class="fdesc">Pods start while 50 GB CUDA images download. No cold-start tax.</div></div>
  <div class="feature-item"><div class="fname"><a href="https://cloud.google.com/run/docs/configuring/services/gpu">Cloud Run with GPUs</a></div><div class="fdesc">Model → REST endpoint. Scale-to-zero, no Kubernetes.</div></div>
  <div class="feature-item red"><div class="fname"><a href="https://docs.cloud.google.com/kubernetes-engine/docs/how-to/checkpoint-restore">Pod Snapshots</a></div><div class="fdesc">80% faster warm restart for 70B models.</div></div>
  <div class="feature-item green"><div class="fname"><a href="https://docs.cloud.google.com/kubernetes-engine/docs/concepts/about-gke-inference-gateway">Inference Gateway</a></div><div class="fdesc">Model-aware routing. 70% faster time-to-first-token vs standard load balancing.</div></div>
</div>

---

<!-- _class: section green -->

# TPU

---

## Who runs on TPUs

<div class="cards">
  <div class="card"><div class="name"><a href="https://www.anthropic.com/news/expanding-our-use-of-google-cloud-tpus-and-services">Anthropic</a></div><div class="stat">1M chips</div><div class="note">For Claude.</div></div>
  <div class="card yellow"><div class="name"><a href="https://www.networkworld.com/article/4015386/openai-tests-google-tpus-amid-rising-inference-cost-concerns.html">OpenAI</a></div><div class="stat">20–40% cheaper</div><div class="note">ChatGPT inference vs GPU.</div></div>
  <div class="card green"><div class="name"><a href="https://arxiv.org/abs/2407.21075">Apple</a></div><div class="stat">8,192 chips</div><div class="note">Foundation Models on TPUv4.</div></div>
</div>

<div class="cards">
  <div class="card green"><div class="name"><a href="https://medium.com/@kshitizrimal/tpu-vs-gpu-the-shift-from-general-purpose-to-pure-performance-da04cf39c75c">Midjourney</a></div><div class="stat">$2M → $700K</div><div class="note">Monthly compute.</div></div>
  <div class="card red"><div class="name"><a href="https://siliconangle.com/2026/02/26/google-meta-reportedly-strike-new-multibillion-dollar-ai-chip-deal/">Meta</a></div><div class="stat">Llama + ranking</div><div class="note">Multi-year TPU lease, Feb 2026.</div></div>
  <div class="card"><div class="name"><a href="https://cloud.google.com/customers/recursion">Recursion Pharma</a></div><div class="stat">Drug discovery</div><div class="note">Neural net training at scale.</div></div>
</div>

---

## Why TPU economics are structural

<div class="cards four">
  <div class="card"><div class="name">vs GB200</div><div class="stat">30%</div><div class="note">Lower TCO/hr.</div></div>
  <div class="card red"><div class="name">vs GB300</div><div class="stat">41%</div><div class="note">Lower TCO/hr.</div></div>
  <div class="card yellow"><div class="name">Realized MFU</div><div class="stat">40 / 30</div><div class="note">52% lower cost per effective PFLOP.</div></div>
  <div class="card green"><div class="name"><a href="https://www.anthropic.com/news/claude-opus-4-5">Claude Opus</a></div><div class="stat">67%</div><div class="note">API price cut.</div></div>
</div>

NVIDIA [paid ~$20B for Groq's LPU](https://www.cnbc.com/2025/12/24/nvidia-groq-deal.html) — the inference architecture Google has had since 2015.

---

## NeuralGCM

<div class="split">
  <div class="cards stack">
    <div class="card red"><div class="name">Legacy HPC</div><div class="stat">19 sims</div><div class="note">13,824 CPU cores.</div></div>
    <div class="card green"><div class="name">JAX</div><div class="stat">70,000 sims</div><div class="note">Single TPU chip.</div></div>
  </div>
  <div class="split-img">
    <video src="visuals/NeuralGCM-MP4-3.mp4" autoplay loop muted playsinline></video>
    <p class="caption"><span class="big">3,500×</span> faster atmosphere simulation</p>
  </div>
</div>

---

<!-- _class: compact -->

<h2 style="color: var(--gps-ink)">Cellular Interaction Foundation Model · <span style="color: #FF6C0C">Caltech</span></h2>

1B GNN, Parkinson's simulations for [Michael J. Fox Foundation](https://www.michaeljfox.org/)

<div class="split">
  <div>
    <h3>Inference results</h3>
    <table>
      <tr><td>Baseline on-prem</td><td>H200</td><td><span style="color:var(--gps-red);font-weight:500">10 min</span></td></tr>
      <tr><td>Google Cloud</td><td>TPU v7x</td><td><span style="color:var(--gps-green);font-weight:500">1.4 min</span></td></tr>
    </table>
    <h3>Planned model retrain</h3>
    <table>
      <tr><td>Baseline on-prem</td><td>H200</td><td><span style="color:var(--gps-red);font-weight:500">1 month</span></td></tr>
      <tr><td>Google Cloud</td><td>TPU v5p</td><td><span style="color:var(--gps-green);font-weight:500">8.3 days</span></td></tr>
    </table>
  </div>
  <div class="split-img">
    <img src="visuals/cifm_tissue.gif" alt="CIFM tissue simulation"/>
  </div>
</div>

<div class="chips">
  <span class="chip solid-red">10K experiments · GPU · $53,000</span>
  <span class="chip solid-green">10K experiments · TPU · $8,900</span>
</div>

---

<!-- _class: image -->

![w:780](visuals/torchtpu.png)

---

## Live demo

<a href="http://35.201.74.96"><img src="visuals/demo_overview.gif" style="max-width:90%;max-height:440px;object-fit:contain;border-radius:8px;" /></a>

---

<!-- _class: title -->

# Wrap

Four pillars covered:

<div class="chips">
  <span class="chip solid-blue">NVIDIA, integrated</span>
  <span class="chip solid-yellow">Capacity without a commit</span>
  <span class="chip solid-green">TPU economics</span>
  <span class="chip solid-red">HPC storage tier</span>
</div>

williszhang@google.com

<script>document.querySelectorAll('a[href^="http"]').forEach(a=>a.target='_blank')</script>
