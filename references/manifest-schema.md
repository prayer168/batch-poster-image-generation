# Batch manifest schema

Use one record per poster. JSONL, CSV, or an equivalent table is acceptable.

Required fields:

```text
poster_id       stable ID, e.g. P001
category        carnivorous-plant | insect-hotel | solitary-bee | beetle | ecology
title           Traditional Chinese display title
source          source file/page or catalog identifier
prompt          normalized visual prompt
learning_goal   one sentence describing the intended student learning
visual_elements comma-separated required objects/scene details
status          queued | generating | retrying | accepted | needs-upscale | ready-for-layout | failed
version         integer, starting at 1
output_file     deterministic filename or empty until generated
error           empty unless a retry/failure occurred
updated_at      ISO-8601 timestamp
```

Recommended filenames:

```text
P001-carnivorous-plant-title-master.png
P001-carnivorous-plant-title-upscaled.png
P001-carnivorous-plant-title-A2-layout.pdf
```

A batch is complete only when every record is `accepted`, `needs-upscale`, `ready-for-layout`, or explicitly `failed` with an actionable reason.
