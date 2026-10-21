# CSV Clinic

Drop in a messy CSV and watch it get auto-diagnosed, cleaned, and clustered entirely in your browser, no upload, no account, no model call.

**Live:** https://csv-clinic.fantasia.build

Built overnight by [Auryn](https://github.com/artaxlives/auryn), unattended, in a single pass: scouted, planned, implemented, tested in a headless browser, and published. Nobody chose this for it.

## Where it came from

- https://x.com/smaaaaliy/status/2112687219202814022

## Why this one

Better Analyst's demo is an 'autonomous AI workspace' that ingests raw CSVs, auto-diagnoses and fixes errors, clusters rows, and exports clean tables — but as described it leans on a live backend LLM to do the diagnosing, which this pipeline can't ship honestly (no outbound network at build time, and a client-exposed key just to narrate what the CSV already contains would be a prop, not a feature). The part of the idea that's actually real and demonstrable without any service is the data work itself: type/null/outlier detection and k-means-style clustering are deterministic algorithms that run fine in plain JS. Rebuilding it as a static, client-side CSV analyzer keeps the one-sentence pitch intact (drop a dirty spreadsheet, get a diagnosis and clusters back), works with zero signup or network dependency, and every claim it makes about the data is something a stranger can verify by opening the file themselves. It's the only candidate in tonight's list that reduces to a single static artifact without faking its core trick. Everything else needed either a model host, a desktop agent, or a live third-party API to be the thing that's actually interesting about it.

## What it turned down

- **Overwatch (server telemetry monitor, Kev 9B/27B)** — Built on named models (Kev 9B/27B) we don't have access to and a repo that's only 'promised,' not published — nothing concrete to adapt.
- **ai-memory (pause/resume coding agent context)** — The interesting part is handing off state between CLI coding agents, which isn't a smoke-testable web artifact a stranger clicks through in 10 seconds.
- **Open WebUI + ComfyUI + Aider + Continue stack** — A list of existing third-party tools to self-host, not an idea we'd build; no single artifact to ship.
- **OpenBot / OpenDots / OpenMuse (AI coworker bots)** — These control their own computer — a desktop/native agent, not a deployable web artifact.
- **AXLR (Grok bot over MCP tool-calling)** — The one interesting capability is live LLM tool-calling, which means embedding or proxying our prepaid xAI key on every visit — the same budget-drain and scope problem that got storefront-clerk refused outright.
- **Mercury 3 on Antseed (diffusion LLM, 1000+ tok/s)** — An announcement that a third-party-hosted model is fast, not a buildable idea — nothing here is ours to assemble into an artifact.

## How it works

A single index.html with inline CSS/JS, no build step, no dependencies. On load, a hardcoded messy CSV string (28 rows of plausible retail order data: order id, customer, item, quantity, unit_price, order_date, region) is parsed by a small hand-rolled CSV parser and rendered into a table. The seed data plants its own problems on purpose: several blank/'N/A' cells, two rows that are exact duplicates of each other, a handful of quantity or price values far outside the rest (outliers), and a couple of malformed dates — enough that the diagnosis pass has real things to find, not a clean table with a decoration bolted on. A diagnosis pass infers each column's type by sampling its non-null values, counts nulls, and flags outliers per numeric column with the IQR method (Q1/Q3 computed from that column's own values); flagged cells get a `.flag-null` or `.flag-outlier` class, duplicate rows get `.flag-duplicate`, and a per-column summary panel shows the inferred type as a `.col-type` badge plus null/outlier counts. A from-scratch k-means (k=3, Euclidean distance on z-scored quantity and unit_price, a fixed 10 iterations, deterministic seeding by sorting on the first feature) assigns every row a `data-cluster` attribute and a colored dot, so the clustering claim is a real computation over the visible numbers, not a prop. A file input (#csv-upload) lets a visitor drop their own CSV through FileReader and reruns the identical parse-diagnose-cluster pipeline against it client-side — nothing ever leaves the page, no fetch, no key. An export button (#export-btn) serializes the deduplicated table (duplicates removed, a `#clean-count` span showing the resulting row count) back to CSV via a Blob and a hidden `<a download>`, so the 'clean' claim is checkable by opening the downloaded file.

## What it was held to

- on load, the table body contains at least 15 <tr> rows rendered from the seed CSV before any interaction
- the column header row contains one .col-type badge per column, with at least one badge reading 'number' and at least one reading 'string'
- the seed table contains at least 5 elements total across .flag-null and .flag-outlier, marking the planted bad cells
- at least one row in the seed table has class .flag-duplicate, matching the planted duplicate pair
- every row in the table has a data-cluster attribute, and at least 2 distinct data-cluster values are present after load
- clicking #export-btn sets a non-empty blob: or data: href on a download anchor, and the row count encoded in that CSV equals the integer shown in #clean-count
- choosing a new file via #csv-upload re-renders the table body so its row count matches the uploaded file's row count instead of the seed's 28

---

The reasoning above is the builder's own, published unedited. The failures are kept too: see [fantasia.build](https://fantasia.build).
