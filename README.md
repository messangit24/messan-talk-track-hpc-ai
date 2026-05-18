# Willis Zhang — AI Infra Talk Track

Generic, citation-clean version of the HPC + AI on Google Cloud briefing. Source slides in `slides/slides.md` (Marp). This README is the talk track — pure bullet outline, one bulleted block per slide. Fill in customer-specific framing, researcher name-checks, and workload anecdotes **verbally** per engagement; the slides themselves stay clean. Target runtime: ~20 minutes.

## Generate the slides

```bash
cd slides
npx @marp-team/marp-cli slides.md --html --theme-set theme.css --output slides.html        # web view
npx @marp-team/marp-cli slides.md --html --theme-set theme.css --pdf  --output slides.pdf  # PDF (needs Chrome installed)
npx @marp-team/marp-cli slides.md --html --theme-set theme.css --pptx --output slides.pptx # PPTX
```

`--html` is required (the deck uses inline HTML cards / split layouts). For PDF/PPTX, Marp launches a headless Chrome; set `CHROME_PATH=/path/to/chrome` if Marp can't find one.

## Aero-sim live demo (slide 15)

Detailed segment walkthrough lives in a separate Google Doc: [aero-sim demo talk track](https://docs.google.com/document/d/1Vtz-UKe5E__SPvYQ5jkAwSdgxMVmNBtExa_NCqdNNVI/). Demo URL: `http://35.201.74.96` — pre-warm `/api/ready` before any presentation.

---

## Slide 1 — Title (0:20)

- Self-intro: name, role, prior background, certifications
- One-line frame: what the deck covers and why it matters today

---

## Slide 2 — Current conditions (1:40)

- Empathy: acknowledge the audience's existing HPC environment (on-prem cluster, queue, GPU partition)
- Frame: not replacing the on-prem environment, extending it for workloads the on-prem cluster can't serve fast enough
- Google AI infra credentials — `$190B` Alphabet 2026 CapEx, `90%` of generative AI unicorns on Google Cloud
- Set up: "the rest of this deck is what that capacity actually looks like"

---

## Slide 3 — Section divider: NVIDIA on Google Cloud (0:10)

- Transition: "let's start with the familiar — NVIDIA GPUs"

---

## Slide 4 — Every cloud has H100s. We add this. (3:20)

- Frame: every hyperscaler sells H100s; the differentiation is what surrounds them
- **Optical Circuit Switching** — physical fabric rewires around failed chips in ms, job survives without restart (AWS/Azure non-blocking fabrics can't reroute around failure dynamically)
- **Node Health Prediction** — Google fleet-wide ML predicts which GPU nodes will degrade in the next 5h; scheduler drains them before symptoms surface (AWS SageMaker HyperPod Deep Health Checks detect after the fact)
- **Multi-Tier Checkpointing** — writes to local RAM disk, replicates to peer node, async-uploads to Cloud Storage; restart pulls from the nearest tier (AWS/Azure: build it yourself with FSx+S3 or Blob)
- **Topology-aware Slurm via Cluster Director** — sub-block / block / cluster metadata of the physical network exposed to the scheduler; tightly-coupled jobs land on co-located nodes
- **Rail-aligned hierarchical networking** — sub-block (single hop) / block (2 hops) / cluster (3 hops); orchestrators can request "same block" placement explicitly
- **Goodput** — public metric: fraction of paid GPU-hours that productively moved a model forward (Google publishes it, optimizes Cluster Director against it)
- Close: "AWS and Azure ship none of these as integrated managed features"

---

## Slide 5 — and also storage (1:00)

- Frame: three tiers, three access patterns — pick by workload
- **Managed Lustre** — built in collaboration with DDN; full POSIX, sub-ms latency, 10 TB/s; AWS FSx for Lustre caps around 2 TB/s per file system. Use case: hot scratch during the job
- **Cloud Storage as POSIX** (GCSFuse + Hierarchical Namespace) — mount a bucket as a filesystem path; 9× faster model loads and 30× faster checkpoint writes vs stock driver. Use case: durable project data, multi-region accessible
- **Rapid Storage** — sub-ms latency, 6 TB/s zonal, ~5× faster than equivalents at other hyperscalers; ingest-on-write caches data as it uploads. Use case: object-storage durability with filesystem-grade latency simultaneously
- Close: "Lustre for hot scratch, Cloud Storage for durable project data, Rapid Storage when you need both"

---

## Slide 6 — Section divider: How you consume (0:10)

- Transition: "hardware availability is the consumption story"

---

## Slide 7 — Capacity without a commit (2:50)

- Frame: four ways to land capacity without a multi-year CUD
- **DWS Flex Start** — guaranteed GPU/TPU capacity for up to 7 days per request, no reservation contract; AWS Capacity Blocks require fixed duration + rigid sizing
- **Calendar Mode** — reserve specific date windows up to 90 days out; ideal for known data-release or grant-deadline timelines
- **Multi-region Spot** — Google has thousands of GPU chips in Spot pools across CONUS at any moment; single-zone pockets routinely hundreds of H100 Mega chips. Pattern: single Slurm controller fans nodesets across multiple regions, daily poller books capacity wherever it surfaces, preemption → `--requeue` → resume from checkpoint on a different region. [Reference deployment open source](https://github.com/WandLZhang/slurm-multi-region-gpu)
- **BoltVMs** — pre-initialized GPU nodes; H100 cold-start in ~2 minutes vs industry-typical 5–15 minutes
- Close: "AWS raised H200 list prices ~15% in Jan 2026; Google's trajectory moves the other way"

---

## Slide 8 — Reimagine what a node is (1:30)

- Frame: seven features that change what a "node" means on Google Cloud
- **Nextflow on Cloud Batch** — nf-core pipelines run with DWS Flex GPU guarantees underneath
- **G4 fractional GPUs** — 1/8, 1/4, or 1/2 of an RTX PRO 6000 Blackwell; workloads that need a GPU but not a whole H100
- **TPU + GPU in one GKE cluster** — mixed accelerators, no parallel infrastructure
- **Image Streaming** — 50 GB CUDA images, pods start while the image streams; Azure's equivalent caps useful image streaming around 30 GB
- **Cloud Run with GPUs** — model → REST endpoint, scale-to-zero, per-second billing, no Kubernetes
- **Pod Snapshots** — snapshot full pod state, restore in seconds; 80% faster warm restart for 70B models; stop paying for idle accelerators between bursts
- **Inference Gateway** — model-aware routing by predicted GPU latency, cache hit, LoRA adapter locality; 70% lower time-to-first-token vs standard load balancing
- Close: "the node is no longer the unit — the workload is"

---

## Slide 9 — Section divider: TPU (0:10)

- Transition / ice-breaker: "show of hands or chat — who's heard of TPUs?"

---

## Slide 10 — Who runs on TPUs (1:50)

- Frame: every major frontier AI lab that had a real choice has landed on TPU for at least some production workload
- **Anthropic** — trains and serves Claude on up to 1 million TPU chips (largest publicly-disclosed AI infrastructure deal)
- **OpenAI** — production ChatGPT inference on TPUs since mid-2025; multi-year commitment, not a pilot; industry analysts put TPU inference at ~20–40% cheaper than equivalent GPU. Relationship has deepened with Ironwood (TPU v7) capacity. Not exclusive — OpenAI also signed Cerebras and is reportedly building own silicon
- **Apple** — trained Apple Foundation Models on 8,192 TPUv4 chips at ~52% sustained MFU (arXiv:2407.21075)
- **Midjourney** — monthly compute went from ~$2M to ~$700K after migrating to TPU
- **Meta** — signed multi-year, multi-billion-dollar TPU lease in February 2026 for Llama training + internal ranking and personalization. Notable because Meta runs the largest single NVIDIA cluster in the industry (100K+ H100s for Llama 4). They're not switching, they're diversifying
- **Recursion Pharmaceuticals** — drug discovery, neural net training at scale on TPU
- Close: "~90% of generative AI unicorns run on Google Cloud AI infrastructure"

---

## Slide 11 — Why TPU economics are structural (1:00)

- Frame: TPU isn't a cheaper-GPU substitute; the economics flip because of how the silicon is designed
- **vs GB200** — ~30% lower TCO per hour (SemiAnalysis)
- **vs GB300** — ~41% lower TCO per hour (SemiAnalysis)
- **Realized MFU** — ~40% TPU vs ~30% GPU on transformer training → ~52% lower cost per effective PFLOP
- **Anthropic Claude Opus** — 67% API price cut at Opus 4.5 launch (Nov 2025); maintained through 4.6 and 4.7. Direct downstream consequence of TPU economics
- Close: "NVIDIA paid ~$20B for Groq's LPU — the systolic-array inference architecture Google has shipped since 2015"

---

## Slide 12 — NeuralGCM (1:00)

- Frame: PDE solvers were the canonical HPC workload; on JAX+TPU they're a different category of throughput
- **NeuralGCM** = neural-network-augmented general circulation model from Google Research; simulates Earth's atmosphere
- Benchmark: **70,000 simulated atmosphere-days per 24h wall-clock on a single TPU** vs **19 simulated days per 24h on legacy HPC + 13,824 CPU cores** = **3,500× productivity gap per researcher-day**
- Generalizes: anyone running PDE solvers — fluid dynamics, geophysics, climate, materials simulations, quantum dynamics — sees the same shape of speedup
- Close: "the chart on the right is from the Google Research blog"

---

## Slide 13 — Caltech Cellular Interaction FM (1:00)

- Frame: 1B-parameter graph neural network for spatial genomics; funded by Michael J. Fox Foundation for Parkinson's research; ported from PyTorch on B200 to JAX on TPU
- **Inference time per step**: H200 baseline 10 min → TPU v7x **1.4 min**
- **Model retrain**: H200 baseline 1 month → TPU v5p **8.3 days**
- **10,000 drug-screening experiments**: ~$53K on GPU → ~$8.9K on TPU
- Generalizes: any batch inference at biological scale — protein function prediction, multi-omic biomarker work, drug screening — flips the same way at production scale
- Close: "training time was comparable per epoch; the economics flip because you run thousands of these"

---

## Slide 14 — TorchTPU (0:30)

- Frame: historically TPU required JAX; that's changed
- **TorchTPU** — set `device='tpu'` in existing PyTorch code, runs natively on TPU
- First-class in **Ray 2.55** — first hardware accelerator to earn first-class Ray support since NVIDIA GPUs
- Close: "if your code already runs in PyTorch, the migration is one line"

---

## Slide 15 — Live demo (2:30)

- Open browser at `http://35.201.74.96`
- Setup: live aerodynamic simulation, Hess-Smith panel method on an F-22 mesh, 143,000 triangles. Two backends simultaneously — TPU v6e Trillium and an H100 8-GPU. Same JAX source, same physics, same coefficients
- Click "Simulate" — identical results, **TPU $0.000556/sec, H100 $0.009831/sec → 18× difference in per-second cost for identical physics**
- Click "Run 100-case design sweep" — 100 flight conditions in a single batched call
- Detailed walkthrough notes: see the [aero-sim demo Google Doc](https://docs.google.com/document/d/1Vtz-UKe5E__SPvYQ5jkAwSdgxMVmNBtExa_NCqdNNVI/)
- Close: "URL is on the slide — try it after this session"

---

## Slide 16 — Wrap (1:00)

- One-sentence recap of each of the four pillars: NVIDIA differentiation, capacity without a commit, TPU economics, HPC storage tier
- Contact: williszhang@google.com
- Handoff to the next speaker / open for questions
