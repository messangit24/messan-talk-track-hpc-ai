# AI Infra Talk Track

[**View the deck**](https://storage.googleapis.com/williszhang-ai-infra-talk-track-slides/slides.html)

## Generate the slides

```bash
cd slides
npx @marp-team/marp-cli slides.md --html --theme-set theme.css --output slides.html
# optional: --pdf / --pptx (launches headless Chrome; can mangle CSS overlays in PDF)
```

---

## Opening

- Self-intro: name, role, prior background, certifications
- The deck covers HPC and AI workloads on Google Cloud — the workloads your on-prem cluster can't serve fast enough and the four pillars that change that

## Current conditions

- Acknowledge the audience's existing HPC environment (on-prem cluster, queue, GPU partition); we're extending it, not replacing it
- [$190B Alphabet 2026 CapEx](https://www.cnbc.com/2026/04/29/alphabet-googl-q1-2026-earnings.html), mostly AI
- [~90% of generative AI unicorns](https://cloud.google.com/ai-infrastructure) run on Google Cloud AI infrastructure

## What we add to NVIDIA

Every hyperscaler sells H100s. The differentiation is what surrounds them:

- [**Optical Circuit Switching**](https://cloud.google.com/blog/products/networking/introducing-virgo-megascale-data-center-fabric) — physical fabric rewires around failed chips in milliseconds, the job survives without restart; AWS/Azure non-blocking fabrics can't reroute around failure dynamically
- [**Node Health Prediction**](https://docs.cloud.google.com/ai-hypercomputer/docs/workloads/enable-node-health-prediction) — fleet-wide ML predicts which GPU nodes will degrade in the next 5h; scheduler drains before symptoms surface (AWS SageMaker HyperPod Deep Health Checks detect after the fact)
- [**Multi-Tier Checkpointing**](https://docs.cloud.google.com/kubernetes-engine/docs/how-to/machine-learning/training/multi-tier-checkpointing) — local RAM disk → peer node → Cloud Storage; restart pulls from the nearest tier. AWS and Azure don't ship an integrated equivalent — you build it with FSx+S3 or Blob
- [**Topology-aware Slurm via Cluster Director**](https://docs.cloud.google.com/cluster-director/docs/orchestration) — sub-block / block / cluster metadata of the physical network exposed to the Slurm scheduler; tightly-coupled jobs land on co-located nodes
- [**Rail-aligned hierarchical networking**](https://docs.cloud.google.com/ai-hypercomputer/docs/networking-overview) — sub-block (1 hop) / block (2 hops) / cluster (3 hops); orchestrators can request "same block" placement explicitly
- [**Goodput**](https://cloud.google.com/blog/products/ai-machine-learning/goodput-metric-as-measure-of-ml-productivity) — public metric: fraction of paid GPU-hours that productively moved a model forward; Cluster Director optimizes against it

AWS and Azure ship none of these as integrated managed features.

## Storage tier

Three tiers, three access patterns — pick by workload:

- [**Managed Lustre**](https://docs.cloud.google.com/managed-lustre/docs/overview) — built in collaboration with DDN; full POSIX, sub-ms latency, 10 TB/s; AWS FSx for Lustre caps around 2 TB/s per file system. **Use case:** hot scratch during the job
- [**Cloud Storage as POSIX**](https://cloud.google.com/kubernetes-engine/docs/how-to/persistent-volumes/cloud-storage-fuse-csi-driver) (GCSFuse + Hierarchical Namespace) — mount a bucket as a regular filesystem path; 9× faster model loads and 30× faster checkpoint writes vs stock driver. **Use case:** durable project data, multi-region accessible
- [**Rapid Storage**](https://docs.cloud.google.com/storage/docs/rapid/rapid-cache) — sub-ms latency, 6 TB/s zonal, ~5× faster than equivalents at other hyperscalers; ingest-on-write caches data as it uploads. **Use case:** object-storage durability with filesystem-grade latency at the same time

Lustre for hot scratch. Cloud Storage for durable project data. Rapid Storage when you need both.

## How you consume capacity

Four ways to land capacity without a multi-year CUD:

- [**DWS Flex Start**](https://docs.cloud.google.com/kubernetes-engine/docs/concepts/dws) — guaranteed GPU/TPU capacity for up to 7 days per request, no reservation contract; AWS Capacity Blocks require fixed duration and rigid sizing
- [**Calendar Mode**](https://docs.cloud.google.com/compute/docs/instances/future-reservations-calendar-mode-overview) — reserve specific date windows up to 90 days out; ideal for data-release or grant-deadline timelines
- **Multi-region Spot** — Google has thousands of GPU chips in Spot pools across CONUS at any moment; single-zone pockets routinely hundreds of H100 Mega chips. Pattern: single Slurm controller fans nodesets across multiple regions, daily poller books capacity wherever it surfaces, preemption triggers `--requeue` and resume from checkpoint on a different region. [Reference deployment](https://github.com/WandLZhang/slurm-multi-region-gpu) is open source
- [**BoltVMs**](https://cloud.google.com/kubernetes-engine/docs/concepts/fast-starting-nodes) — pre-initialized GPU nodes; H100 cold-start in ~2 minutes vs industry-typical 5–15 minutes

AWS raised H200 list prices ~15% in Jan 2026; Google's trajectory moves the other way.

## Reimagine what a node is

Seven features that change what a "node" means:

- [**Nextflow on Cloud Batch**](https://docs.cloud.google.com/batch/docs/nextflow) — nf-core pipelines run with DWS Flex GPU guarantees underneath
- [**G4 fractional GPUs**](https://docs.cloud.google.com/compute/docs/accelerator-optimized-machines#g4-series) — 1/8, 1/4, or 1/2 of an RTX PRO 6000 Blackwell; workloads that need a GPU but not a whole H100
- [**TPU + GPU in one GKE cluster**](https://docs.cloud.google.com/kubernetes-engine/docs/how-to/tpus) — mixed accelerators, no parallel infrastructure
- [**Image Streaming**](https://docs.cloud.google.com/kubernetes-engine/docs/how-to/image-streaming) — 50 GB CUDA images, pods start while the image streams; Azure's equivalent caps useful image streaming around 30 GB
- [**Cloud Run with GPUs**](https://cloud.google.com/run/docs/configuring/services/gpu) — model → REST endpoint, scale-to-zero, per-second billing, no Kubernetes
- [**Pod Snapshots**](https://docs.cloud.google.com/kubernetes-engine/docs/how-to/checkpoint-restore) — snapshot full pod state, restore in seconds; 80% faster warm restart for 70B models
- [**Inference Gateway**](https://docs.cloud.google.com/kubernetes-engine/docs/concepts/about-gke-inference-gateway) — model-aware routing by predicted GPU latency, cache hit, LoRA adapter locality; 70% lower time-to-first-token vs standard load balancing

The node is no longer the unit — the workload is.

## TPU

Every major frontier AI lab that had a real choice has landed on TPU for at least some production workload:

- [**Anthropic**](https://www.anthropic.com/news/expanding-our-use-of-google-cloud-tpus-and-services) — trains and serves Claude on up to 1M TPU chips; largest publicly-disclosed AI infrastructure deal
- [**OpenAI**](https://www.networkworld.com/article/4015386/openai-tests-google-tpus-amid-rising-inference-cost-concerns.html) — production ChatGPT inference on TPUs since mid-2025; multi-year commitment, not a pilot; industry analysts put TPU inference at ~20–40% cheaper than equivalent GPU. Relationship has deepened with Ironwood (TPU v7) capacity. Not exclusive — OpenAI also signed [Cerebras](https://www.networkworld.com/article/4117296/openai-turns-to-cerebras-in-a-mega-deal-to-scale-ai-inference-infrastructure.html) and is reportedly building its own silicon
- [**Apple**](https://arxiv.org/abs/2407.21075) — trained Apple Foundation Models on 8,192 TPUv4 chips at ~52% sustained MFU
- [**Midjourney**](https://medium.com/@kshitizrimal/tpu-vs-gpu-the-shift-from-general-purpose-to-pure-performance-da04cf39c75c) — monthly compute went from ~$2M to ~$700K after migrating to TPU
- [**Meta**](https://siliconangle.com/2026/02/26/google-meta-reportedly-strike-new-multibillion-dollar-ai-chip-deal/) — multi-year, multi-billion-dollar TPU lease signed Feb 2026 for Llama training and internal ranking/personalization. Meta runs the largest single NVIDIA cluster in the industry (100K+ H100s for Llama 4) — they're not switching, they're diversifying
- [**Recursion**](https://cloud.google.com/customers/recursion) — drug discovery, neural net training at scale on TPU
- [**TPUv8**] (https://blog.google/innovation-and-ai/infrastructure-and-cloud/google-cloud/eighth-generation-tpu-agentic-era/)
- [**TPUv8-Architecture**] (https://cloud.google.com/blog/products/compute/tpu-8t-and-tpu-8i-technical-deep-dive?e=48754805)
- [**TPUv8-Virgo**] (https://cloud.google.com/blog/products/networking/introducing-virgo-megascale-data-center-fabric?e=48754805)
- [**TPUv7-IronWood**] (https://docs.cloud.google.com/kubernetes-engine/docs/concepts/tpu-ironwood)
- [**TPU-TorchTPU**] (https://developers.googleblog.com/torchtpu-running-pytorch-natively-on-tpus-at-google-scale/)

~90% of generative AI unicorns run on Google Cloud AI infrastructure.

### Why the economics are structural

- **vs GB200** — ~30% lower TCO per hour ([SemiAnalysis](https://newsletter.semianalysis.com/p/tpuv7-google-takes-a-swing-at-the))
- **vs GB300** — ~41% lower TCO per hour (SemiAnalysis)
- **Realized MFU** — ~40% TPU vs ~30% GPU on transformer training → ~52% lower cost per effective PFLOP
- [**Anthropic Claude Opus**](https://www.anthropic.com/news/claude-opus-4-5) — 67% API price cut at Opus 4.5 launch (Nov 2025); maintained through 4.6 and 4.7. Direct downstream consequence of TPU economics
- NVIDIA recently [paid ~$20B for Groq's LPU](https://www.cnbc.com/2025/12/24/nvidia-groq-deal.html) — the systolic-array inference architecture Google has shipped since 2015

### NeuralGCM proof point

- PDE solvers were the canonical HPC workload; on JAX+TPU they're a different category of throughput
- [**NeuralGCM**](https://research.google/blog/fast-accurate-climate-modeling-with-neuralgcm/) — neural-network-augmented general circulation model from Google Research; simulates Earth's atmosphere
- Benchmark: **70,000 simulated atmosphere-days per 24h wall-clock on a single TPU** vs **19 simulated days per 24h on legacy HPC + 13,824 CPU cores** = **3,500× productivity gap**
- Generalizes: fluid dynamics, geophysics, climate, materials simulations, quantum dynamics — all see the same shape of speedup

### CIFM proof point

- 1B-parameter graph neural network for spatial genomics; funded by [Michael J. Fox Foundation](https://www.michaeljfox.org/) for Parkinson's research; ported from PyTorch on B200 to JAX on TPU
- **Inference per step:** H200 baseline 10 min → TPU v7x 1.4 min
- **Model retrain:** H200 baseline 1 month → TPU v5p 8.3 days
- **10,000 drug-screening experiments:** ~$53K on GPU → ~$8.9K on TPU
- Generalizes to any batch inference at biological scale — protein function prediction, multi-omic biomarker work, drug screening

### TorchTPU

- Historically TPU required JAX; that changed
- [**TorchTPU**](https://developers.googleblog.com/torchtpu-running-pytorch-natively-on-tpus-at-google-scale/) — set `device='tpu'` in existing PyTorch code, runs natively on TPU
- First-class in [Ray 2.55](https://discuss.google.dev/t/google-cloud-tpus-are-now-a-first-class-accelerator-in-ray/345281) — first hardware accelerator to earn first-class Ray support since NVIDIA GPUs

## Live demo

- Open browser at `http://35.201.74.96`; pre-warm `/api/ready` before any presentation
- Setup: live aerodynamic simulation, Hess-Smith panel method on F-22 mesh, 143,000 triangles. Two backends simultaneously — TPU v6e Trillium and an H100 8-GPU. Same JAX source, same physics, same coefficients
- Click "Simulate" — identical results, **TPU $0.000556/sec, H100 $0.009831/sec → 18× per-second cost difference for identical physics**
- Click "Run 100-case design sweep" — 100 flight conditions in a single batched call
- Detailed walkthrough notes: [aero-sim demo Google Doc](https://docs.google.com/document/d/1Vtz-UKe5E__SPvYQ5jkAwSdgxMVmNBtExa_NCqdNNVI/)

## Wrap

- Recap the four pillars: NVIDIA differentiation, capacity without a commit, TPU economics, HPC storage tier
- Contact: williszhang@google.com
