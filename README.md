# Willis Zhang — AI Infra Talk Track

Generic, citation-clean version of the HPC + AI on Google Cloud briefing. Intended for Google Cloud colleagues to adapt per engagement — swap in your own customer name, researcher examples, and workload anecdotes verbally; the slides themselves stay clean.

## Files

```
slides/
  slides.md            Marp deck — 16 slides
  theme.css            Google Public Sector theme
  visuals/             Title background, TorchTPU image, NeuralGCM video, CIFM gif, demo gif
docs/
  talk_track.md        Pure bullet outline per slide, ~20 min runtime
```

## Render

```bash
cd slides
npx @marp-team/marp-cli slides.md --html --theme-set theme.css --output slides.html
python3 -m http.server 8080
# open http://localhost:8080/slides.html
```

The deck uses inline HTML (cards, split layouts, custom CSS classes) — `--html` is required.

## Aero-sim live demo (slide 15)

Detailed segment walkthrough lives in a separate Google Doc: [aero-sim demo talk track](https://docs.google.com/document/d/1Vtz-UKe5E__SPvYQ5jkAwSdgxMVmNBtExa_NCqdNNVI/). The live demo runs at `http://35.201.74.96` — pre-warm before any presentation by hitting `/api/ready`.

## Adapting for an engagement

- Keep the slides generic — don't bake customer-specific researchers or cluster names into the slide content
- Use the talk track bullets as a spine; layer customer-specific framing in verbally
- For audiences with an existing on-prem cluster, slide 2 is the moment to say "we're not replacing this, we're extending it for the workloads it can't serve fast enough"
- Slide 8 (Reimagine what a node is) is the laundry list — pick 2–3 to land on per audience rather than walking all seven
- Every external URL in the deck has been verified live; please re-verify before reuse and replace any 404s with canonical Google sources via [google-dev-knowledge MCP](https://github.com/googleapis/google-dev-knowledge-mcp)
