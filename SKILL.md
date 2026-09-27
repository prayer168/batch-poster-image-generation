---
name: batch-poster-image-generation
description: "Batch-generate repeated educational poster images from a prompt catalog with queueing, retries, deterministic names, metadata, and A2 visual-master/upscale/layout QA. Use when the user asks to generate many poster images without one-by-one interaction."
---

# Batch Poster Image Generation

Use this skill when a user wants a large set of poster images generated from an existing prompt catalog (for example, 71 Taiwan ecology prompts) as one managed job. The skill orchestrates the work in batches; it does not silently publish, print, or spend paid API credits unless the user explicitly asks to execute the generation.

## Operating contract

1. Locate the prompt source and build a manifest before generating. Accept JSON/CSV/Markdown or local HTML cards. Preserve the source order and assign stable IDs such as `P001`–`P071`.
2. Normalize every item into the schema in `references/manifest-schema.md`. Keep the educational subject, Taiwan-specific species names, category, learning objective, and required visual elements. Remove long paragraphs from the image prompt; request short labels only and plan to typeset Traditional Chinese later.
3. Use `gpt-image-2.5-sunburst` for the visual master, portrait A2 ratio, with a target around `2352 × 3328 px` (or the closest supported portrait size). Do not claim that the image model directly outputs `5031 × 7087 px`: generate the clean master first, then upscale and add bleed in a graphics/layout tool. Keep `transparent_background=false` unless the user explicitly requests a cutout.
4. Queue jobs in numeric order and process as many as the current turn can safely complete. A batch is a scheduling unit, not a requirement to stop after one image: use 6 as a normal checkpoint, but continue with the next batch automatically while time, token budget, and image-tool capacity remain. The user should receive one progress update per checkpoint, not a confirmation request for each image. If the underlying image tool only accepts one image per call, loop over the queue internally and continue automatically.
5. For each job, save status, prompt version, model, timestamp, output name, and any retry/error in the manifest. Retry a failed item at most twice with a targeted correction; do not regenerate accepted items.
6. After each batch, run a lightweight visual QA: correct subject/anatomy, readable focal composition, no accidental watermark/logo, enough quiet space for Traditional Chinese text, no important subject inside the 3 mm trim/bleed risk zone, and no invented factual claim embedded in the artwork. Flag uncertain species identification for human review.
7. Stage outputs using this structure:

```text
poster-batch/
  manifest.csv
  prompts/
  masters/2352x3328/
  upscaled/4961x7016/
  layout/5031x7087/
  previews/
  qa/
  failures/
```

8. Do not mark an item `print-ready` until it has a high-quality upscale, 3 mm bleed, final A2 canvas, and Traditional Chinese text re-typeset in Canva, PowerPoint, Affinity Publisher, or SVG. The final output is a layout artifact, not merely the AI master.

## Continue-until-handoff protocol

When the user explicitly asks to keep generating until the token/turn limit and then hand off:

1. Start from the manifest and resume only records not already `accepted`, `needs-upscale`, `ready-for-layout`, or explicitly `failed`.
2. Continue generating checkpoint batches without asking for per-image approval. Before each new image call, ensure enough remaining turn budget for the call and its bookkeeping; stop starting new calls when the remaining budget is too small for a safe completion.
3. At every checkpoint, persist the manifest, prompt version, output paths, retry counts, and a `continuation.json` file containing the next poster ID, completed IDs, failed IDs, timestamp, and target output root.
4. When the safe budget boundary is reached, do not claim the catalog is complete. Report the exact completed/remaining counts and hand off the continuation task with a concise instruction to resume from `continuation.json`. If the host supports thread handoff, use the handoff mechanism; otherwise emit the handoff packet for the next Codex turn.
5. The handoff must be resumable and idempotent: accepted files are never regenerated or overwritten, and only queued/retrying records are eligible for the next worker.

Suggested handoff packet:

```text
Resume batch-poster-image-generation from [OUTPUT_ROOT]/manifest/continuation.json.
Do not regenerate accepted files. Continue automatically in checkpoint batches until
the next safe budget boundary, update the manifest and continuation file, then hand
off again if items remain. Report completed, failed, and remaining poster IDs.
```

## Reusable master prompt

Use this template for each queued item, replacing the bracketed fields:

```text
Batch poster job [POSTER_ID], category [CATEGORY], title [TITLE].
Create a clean educational visual master for a Taiwan ecology A2 portrait poster.
Model profile: GPT-Image-2.5 Sunburst; portrait A2 composition; target master about
2352 × 3328 px; high detail, sharp edges, natural colors, print-oriented lighting.
Subject and scene: [SUBJECT_AND_SCENE].
Learning objective: [LEARNING_OBJECTIVE].
Required visual elements: [VISUAL_ELEMENTS].
Composition: clear focal subject, layered depth, generous quiet space for later
Traditional Chinese typesetting, safe margins, no critical detail at the trim edge.
Text policy: do not render paragraphs or dense Chinese text; at most a short,
placeholder-free title area. I will typeset all final text separately.
Accuracy policy: scientifically plausible Taiwan species morphology and habitat;
no extra species, labels, logos, watermark, QR code, or decorative pseudo-text.
Output: one portrait visual master only, no collage, no mockup, no frame.
```

## Execution prompt for Codex

When the user is ready to run the job, ask for or infer only the missing input path and output location, then use this instruction:

```text
Read the prompt catalog, validate that it contains [N] items, and create a stable
manifest. Queue all items and generate them in batches of 6 with the batch-poster-
image-generation workflow. Continue automatically between batches; do not ask me to
approve each image. Retry only failed items (maximum two retries), record every
status and output filename, and stop with a compact report if a tool/account limit
prevents continuation. After generation, produce a QA report and identify which
items still need upscale, bleed, or manual Traditional Chinese typesetting.
```

## Guardrails

- Never overwrite an accepted output; create a versioned filename such as `P014-v2`.
- Never place API keys in prompts, manifests, screenshots, or generated images.
- If a paid API backend is requested, confirm the provider/model and budget before execution; this skill itself does not configure credentials.
- If generation is interrupted, resume from manifest statuses rather than starting over.
- Keep factual species names and teaching claims separate from visual instructions so a later content review can correct them without regenerating the entire catalog.

For the exact manifest fields and status values, read `references/manifest-schema.md` before starting a batch.
