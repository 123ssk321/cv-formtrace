# FormTrace execution plan

Planning baseline: 2026-10-20. Scope: ONLY FormTrace, quick track. Source brief: “Quick-to-product computer-vision projects for a solo builder,” research dated 2026-10-16, FormTrace section. This standalone plan overrides its month-3 recommendation and its mistaken restrictions on existing human-created public labels; the source report is not needed to execute this plan.

This is a specification for later implementation. Every metric below is a target or measurement procedure, never an observed result. The only deliverables in this planning pass are README.md, this document and planning/backlog.json.

## 1. Product, audience and delivery boundary

Target workflow: a small administrative/bookkeeping operator transcribes invoice header and totals into a spreadsheet, checks the source, and exports a reviewed record. Initial value hypothesis: source-linked review reduces entry time without silently accepting incorrect records. Demand, accuracy on real invoices and willingness to pay remain unproven.

Support one family: single-page, printed Portuguese-language invoices in EUR with explicit field captions. Eight required fields: supplier_name, supplier_tax_id, invoice_number, issue_date, currency, net_total, vat_total, gross_total. Supplier tax ID is an opaque string with a documented syntactic check, not a claim of legal validity. Currency must appear as EUR or a euro symbol; never silently supply it. Net and tax totals must be explicit and add to gross within EUR 0.01. No tax-law interpretation.

MVP inputs: one PDF page, PNG or JPEG; maximum 10 MiB upload, 20 megapixels decoded image and 300 DPI raster output. Crop/scale rasterization further if needed to meet the pixel cap. Reject encrypted/multipage PDFs, malformed media, unsupported formats and oversized inputs with useful errors. Rasterize PDFs before OCR; do not exploit embedded text or generator labels during benchmark inference.

MVP actions: select a fictional public sample or upload locally, process, inspect evidence, correct uncertain values, explicitly confirm a reviewed record, export JSON/CSV, delete session. Public mode has server-enforced sample IDs and no arbitrary upload route. This provides an interactive functional demo and a downloadable package capable of processing supported user files locally.

Excluded: handwriting, receipts, credit notes, mixed currencies, multipage invoices, line-item extraction, attachments, email intake, ERP integration, payments, tax compliance, autonomous bookkeeping, general document classification, model training and public customer-file hosting. Unknown layouts or ambiguous fields must abstain. Do not claim reliable rejection of every out-of-distribution document.

## 2. Operating and publication rules

No project is activated by this plan. When FormTrace is activated, it is the only active project. Other projects remain parked. One builder has 16 hours/week globally, including implementation, reviews, operations, documentation and defects. Work one issue at a time in dependency order; there is no assumed parallel team or free operations time.

Public repository: [123ssk321/cv-formtrace](https://github.com/123ssk321/cv-formtrace), confirmed created public by the parent via gh. Maintain public planning from day one. Do not create another repository. Repository publication is a planning milestone and does not establish functional product delivery.

Luna will later import backlog issues using gh. The importer must preserve FT IDs, estimates, milestone, dependencies, phase and body; match existing FT IDs before creating anything; create missing labels/milestones only within the separately authorized import. A local mapping from FT ID to remote issue number should make retries idempotent. All output paths in backlog bodies are relative to the FormTrace root. They describe FUTURE files, not existing implementations.

The default OCR engine and bounded fallback are specified below. No interview or outreach is required; do not contact people. Optional owner-arranged feedback can supplement the product decision without becoming a dependency.

### Conservative implementation handoff

The parent supplies `AGENTS.md`; this planning task does not create it. After explicit activation, read that file if present, this plan and the selected issue. Confirm dependencies and inspect existing work before editing. Implement only that issue's bounded outputs, run its acceptance checks, and hand back changed paths, commands/results, measured hours, remaining reserve and any unresolved limitation. Do not mark a task complete solely because code exists. Luna's later gh import is bookkeeping, not activation or implementation.

Use the fixed defaults and contracts here for routine implementation choices. Escalate to GPT-6 Astra at high effort before changing document family, schema semantics, OCR engine, deployment/privacy boundaries, dataset rights policy, evaluation gates, total budget or first-public-result target. Provide the failing case, evidence, alternatives, recommended smallest change, affected FT IDs, hours/reserve impact and revised acceptance criteria. Continue unaffected ready work within scope. The parent/user coordinates escalation; this document does not authorize creating chats or sending messages. An estimate overrun alone can use documented reserve without redesign; a scope change needs a recorded decision. Never invent missing results or soften a failed gate.

## 3. Architecture and responsibility boundaries

Choose a small Python service with FastAPI, a static TypeScript/Vite browser UI and SQLite. Use one container for the service and built frontend; no queue service, object store, GPU dependency or distributed architecture for MVP. Pin versions and audit all dependencies at implementation time. These are implementation recommendations, not claims that packages are already installed.

| Module / future path | Responsibility | Boundary |
| --- | --- | --- |
| contracts/ | JSON Schema, generated client types, example payloads | Versioned types; no OCR logic |
| generator/ | Fictional values, layouts, pixels and exact labels | Ground truth never enters runtime |
| src/formtrace/ingest/ | Validate, rasterize, cap resources, establish coordinate transforms | Untrusted parser runs with time/memory limits |
| src/formtrace/ocr/ | Tesseract adapter yielding tokens and rectangular polygons | Fixed local default behind a testable interface |
| src/formtrace/extraction/ | Caption matching, spatial candidates, normalization | Does not invent missing values |
| src/formtrace/policy/ | Required fields, arithmetic, ambiguity and abstention | Versioned thresholds fitted only on validation split |
| src/formtrace/service/ | API, bounded jobs, authorization, deletion | Public sample-only / local upload profiles |
| src/formtrace/storage/ | Documents, revisions, review events, expirations | SQLite transactions; no raw text in diagnostic logs |
| web/ | Image, evidence overlays, accessible review and export | Never decides acceptance only in client |
| evaluation/ | Held-out tests, confidence bounds, timing and failure selection | Ground truth accessible here only |
| deploy/, scripts/ | Local container, public sample deployment, private profile and health checks | No secrets or private documents in releases |

Flow: input -> constrained rasterizer -> canonical raster -> OCR tokens -> field candidates -> normalization and validation -> suggested or needs_review -> reviewed/accepted record -> export. Reject invalid inputs before OCR. Generator metadata is sidecar-only and excluded from runtime images; public service can read sample images and a sample-ID allowlist but never ground-truth sidecars.

### Default OCR and limited fallback

Use Tesseract 5 CLI with the Portuguese `por.traineddata` file from official `tessdata_fast`. Pin the engine package version and traineddata commit/SHA-256 independently. Both official repositories publish Apache-2.0 licenses; record each license separately, retain notices and audit bundled dependencies [S4, S5]. Do not infer resource rights from the engine license alone.

Invoke the executable with an argument array, never a shell string: `tesseract <canonical.png> stdout -l por --oem 1 --psm 3 tsv`. Parse word-level TSV rows into token text, bounding rectangles, normalized coordinates and raw confidence. Nonword rows and negative confidence are not confidence evidence. Fingerprint the executable version, traineddata checksum and configuration. No network call or training is required.

If the first pass leaves required captions/fields unresolved or ambiguous, retry once on the same raster with `--psm 11`; preserve both pass results. Apply the same extraction/validation pipeline to each. Retain only uniquely supported candidates; disagreement between passes on a required normalized value forces review. Never choose a pass using ground truth. Total processing still has a 60-second deadline, including both passes; unresolved or timed-out records abstain. Tune only the documented thresholds on validation. A different engine, additional weights, cloud fallback or custom training requires Astra scope review and an updated budget. TSV and page-segmentation options are documented by Tesseract [S6].

Local processing is the baseline. Public deployment uses the same container in sample-only mode on Cloud Run, subject to current terms, available credentials and cost configuration. A static page with precomputed results alone is not the functional public result. The local release and its interactive UI must remain downloadable and runnable if hosting is temporarily unavailable; document hosted status honestly. Public launch normally requires a live sample demo plus the local package.

Private deployment later is a single-organization, authenticated instance, not multi-tenancy. Credentials and billing activation belong to the owner; prepare deployable artifacts and runbooks before requesting any missing external action. No customer data is necessary for any milestone.

## 4. Concrete contracts

Define JSON Schema and an OpenAPI contract before implementation. Breaking changes increment schema_version; all records also store generator_version when applicable, pipeline_version, policy_version and OCR adapter/version fingerprint.

### Document and field types

- DocumentInput: document_id (random UUID), sha256, media_type, byte_length, mode (public_sample/local/private), source_kind (synthetic/user_file), created_at, expires_at.
- Page: page_index=0, raster_width, raster_height, raster_sha256, source_to_raster 3x3 transform. Display the exact OCR raster; an original-file preview is optional.
- OcrAdapter.recognize(page, language_hint, timeout_ms) -> OcrPage(tokens, backend_fingerprint, elapsed_ms). Token: token_id, text, polygon with four or more points, raw_confidence nullable. Engine errors/timeouts are typed failures.
- Coordinates: top-left origin, x rightwards/y downwards, normalized [0,1] against the canonical displayed raster. Preserve polygons through rotation/perspective; evidence boxes are polygon envelopes. Inverse transforms are used only for an original-file view. Never mix PDF points, pixels and normalized values.
- Field: name, raw_text nullable, normalized_value nullable, evidence[] (page_index, token_ids, polygon), raw_ocr_confidence nullable, extraction_score nullable, rule_results[], status (suggested/needs_review/confirmed/missing), origin (extracted/operator). An extraction score is a heuristic, not a probability.
- Normalization: trim/collapse whitespace and Unicode NFC for names; preserve invoice-number and tax-ID strings including leading zeros; parse only specified unambiguous date forms into YYYY-MM-DD; parse Portuguese decimal commas/group separators into integer EUR cents. Store raw text as well. Currency normalizes to EUR only from visible evidence.
- Record: schema_version, document_id, revision integer, fields map containing exactly the eight names, validations[], decision (needs_review/auto_accepted/reviewed_accepted/rejected), acceptance_mode (assisted/automatic), timestamps and provenance.
- ReviewEvent: event_id, document_id, base_revision, resulting_revision, field_name, before/after, reason, actor_id and timestamp. These events are operational data and never a training corpus.

State machine: queued -> processing -> needs_review | auto_accepted | failed; needs_review -> reviewed_accepted only after required fields resolve and the operator confirms; edits to any accepted record return it to needs_review and increment revision. Deletion/expiration invalidates access. Missing evidence, failed arithmetic, ambiguous candidates, unsupported currency or any timeout prevents automatic acceptance. A manual value without extracted evidence remains origin=operator, requires a reason and explicit confirmation, and is never counted as an automatically extracted field.

### HTTP behavior

| Endpoint | Input | Output / errors |
| --- | --- | --- |
| GET /api/v1/samples | none | Allowed fictional sample IDs and descriptions |
| POST /api/v1/documents | Public: sample_id JSON; local/private: multipart file | 202 with document_id, state and session capability; 413 too large, 415 unsupported, 422 invalid |
| GET /api/v1/documents/{id} | Session capability or private identity | Job/result and revision; 404 for absent or inaccessible documents |
| GET /api/v1/documents/{id}/page | Same access control | Canonical raster only, protected by the same ownership rule |
| PATCH /api/v1/documents/{id}/fields | base_revision, edits, reasons | New revision and validation state; 409 stale revision |
| POST /api/v1/documents/{id}/confirm | base_revision | Reviewed acceptance, or 422 with unresolved rules |
| GET /api/v1/documents/{id}/export?format=json\|csv | Expected revision, same access control | Accepted record plus audit metadata; 409 for stale/unaccepted state |
| DELETE /api/v1/documents/{id} | Same access control | 204; remove raw/derived files and operational records |
| GET /health/live and /health/ready | No document input | Non-sensitive liveness/readiness |

For public/local sessions return an unguessable bearer capability over HTTPS/loopback; keep it in memory, never URLs/logs or persistent browser storage. Local service binds loopback by default, checks origin and permits no wildcard CORS. Private mode requires authenticated identity and document ownership in addition to session behavior. Use random IDs and safe server-owned file names; never accept storage paths from callers.

Default one running OCR job per process, queue bound four, 429 when full, 60-second processing deadline and typed timeout. A managed worker subprocess allows killing a stuck parser/OCR process. Local durable job metadata marks interrupted jobs failed on restart; do not silently resubmit. Public jobs are disposable and instance-local: deploy one instance with session loss on restart documented, use CPU configuration that permits the worker to run, and do not imply cross-instance durability.

Public samples expire in 30 minutes. Local/private default retention 24 hours, configurable lower. Sweep on startup and every five minutes; deletion removes originals, rasters, records and review payloads from live storage. Private backup retention is documented separately (target at most seven days), so live deletion is not falsely represented as instantaneous removal from backups. Public mode uses ephemeral storage and no backups. Logs contain only random IDs, durations, sizes, states and error codes.

## 5. Data acquisition without new manual dataset work

Correct interpretation: public labels made by humans in the past are allowed. The restriction is on new manual data creation by the user or implementation agents. Reject a dataset for incompatible rights, unavailable provenance or task mismatch, not because people originally annotated it. No hand labeling, rescanning documents, collecting photos, transcribing public samples or manually fixing ground truth is part of the plan.

### Default synthetic acquisition

Generate fictional invoice records from seeded code, using invented supplier names/IDs and no scraped brands, logos or actual invoices. Use self-authored layouts and licensed redistributable fonts. Each source record produces PDF/PNG, eight canonical values, rendered value polygons, visibility/presence flags and expected policy disposition automatically. The renderer emits geometry from its own text-placement operations; do not infer labels from OCR.

Create 12 layout families: six development, three validation, three sealed test. Generate 600 development, 300 validation and 600 test base documents; additionally generate 120 unsupported/ambiguous negative documents. Each base gets a clean image and two deterministic corrupted variants. Base document, supplier identity, layout family and all variants stay in the same split. Use disjoint seed namespaces and hashes. A benchmark selection freezes one variant per base for primary independent-record metrics; the other variants form paired stress tests and must not inflate independent sample counts.

Corruptions: rotation, perspective, blur, compression, shadows, low contrast, cropped fields and text occlusion. Transform polygons with the same geometry pipeline and clip to image bounds. Keep exact original values but label cropped/occluded evidence as not fully visible; the correct action can be abstention. Never call an invisible field safely extractable merely because its synthetic value is known. Record corruption parameters and visible area ratio. Rendered annotations must align within two pixels on unwarped fixtures and within a declared interpolation tolerance on transformed ones.

Author validation/test layouts before finalizing extraction rules. Automated property checks catch geometry and decimal errors. If a generator bug is found, repair generator code and regenerate/version every affected split; do not hand-edit examples. The sealed test is used only at release checkpoints; repeated tuning on test results requires a new frozen version.

### Optional existing public data

DocLayNet is a candidate for a layout-only diagnostic, not a prerequisite or an invoice-field gold set. Its official repository describes layout labels, and its published license is CDLA-Permissive-1.0 [S1, S2]. Existing human annotations are not a disqualifier. Do not invent invoice totals or use OCR predictions as external ground truth. If a compatible public corpus with existing invoice-field labels cannot be verified in the bounded acquisition task, publish the gap and continue on synthetic evidence.

For every acquired item or auditable homogeneous collection, record source URL, version/commit, retrieval date, checksum, source rights, annotation rights, license text/hash, attribution, modifications, permitted commercial development/evaluation/training/redistribution uses, privacy restrictions, scope of approval and status. Separate software, fonts/assets, dataset images, annotations and any OCR resources in the bill of materials. A permissive runtime license does not establish rights to its resources.

Gate states: pending -> approved for named use | excluded. Do not download into build inputs, use for evaluation/tuning or distribute until approved. Unknown or noncommercial terms block use in this commercially intended project unless separate written permission covers that use. Labeling something research or evaluation does not bypass the restriction. A license permitting computation does not automatically mean examples can be republished; public benchmark galleries must use synthetic images unless redistribution is explicitly cleared. No licensed external corpus is promised at planning time.

## 6. Evaluation and product gates

Every report includes commit, data manifest hash, seeds, split, policy version, adapter fingerprint, hardware, sample count, exclusions and raw machine-readable results. Write not measured or insufficient evidence until executed. Synthetic results describe only the generated distribution.

| Measure | Definition | Initial decision target |
| --- | --- | --- |
| Field exact match | Correct normalized values / all required fields; null, wrong and abstained predictions count incorrect for this metric | >=95% overall on 600 primary held-out supported records; publish per-field and corruption slices |
| Evidence localization | Mean polygon IoU against visible synthetic value polygons; union multiple token polygons per field; missing evidence scores zero | >=0.80 mean on fully visible fields; separately report partial/occluded slices |
| Automatic coverage | Auto-accepted supported records / all 600 supported primary records | >=60% if automatic mode is enabled |
| Accepted-record error | Auto-accepted records with any wrong required value or wrong field evidence / auto-accepted records | One-sided 95% exact binomial upper bound <=1%, with at least 300 independent accepted records |
| Negative-set safety | Unsafe automatic accepts / 120 unsupported or ambiguous cases | Zero observed; publish n and bound, never claim zero real-world risk |
| Assisted completion | Supported records reaching valid reviewed export / attempted sessions | >=95% over 40 generated tasks; failures and cancellations retained |
| Workflow time | Median elapsed entry+check+export time, paired by difficulty vs manual-entry baseline | >=30% reduction and no increase in final-record error on 40 generated tasks; report spread and participant count |
| Performance | End-to-end acceptance-to-result including queue time, after warm-up; cold start separately | p95 <=15s locally, <=30s hosted, over 100 samples on declared hardware/resources |
| Isolation/deletion | Inaccessible cross-session records and expired/deleted data across all endpoints | Zero successful cross-session reads; live deletion verified within five minutes |
| Cost | Actual measured runtime and configured provider rates, per 100 documents and idle month | Prelaunch estimate <=EUR 20/month for the capped sample demo; recheck actual billed usage |

For accepted-record error, correct evidence means page 0, associated correct value tokens and IoU >=0.5 on fully visible target polygons; cropped mandatory evidence cannot pass automatic acceptance. Give exact binomial intervals for independent primary records; paired variants need clustered analysis or descriptive results. A zero-error sample of 300 is not a universal guarantee. Report numerator/denominator and bounds, not a rounded 100% claim.

Calibrate heuristic thresholds on validation only. Automatic acceptance is off by default until its coverage, error-bound, evidence and negative-set gates pass together. Otherwise release assisted-only mode with explicit review required and measured correction cost. Assisted mode still requires functional extraction, overlays, review/export, security gates and honest benchmark publication; a static mockup cannot satisfy M5. Accuracy targets that fail must be reported, with a scope reduction or explicit experimental label.

The manual-entry time baseline is product-use evaluation on generated documents whose labels already exist. It does not create a dataset: timed entries are compared automatically and never retained as annotation/training assets. The builder can do the 40-task crossover experiment with distinct matched records and counterbalanced order; disclose n=1 and learning effects. Optional owner-arranged participants may improve confidence, but no outreach is authorized. A solo simulation cannot validate market demand.

At M6, continue only if the constrained workflow completes reliably and time/error results justify further work. If time savings fail, narrow supported layouts or park. If commercial demand/external data are unavailable, mark both unvalidated and do not claim a paid pilot. The final decision can be a maintained experimental release or a parked project; paying customers are not a required or fabricated milestone.

## 7. Milestones and exact hours

Month 4 is the target for the first functional public result, not day-one repository publication. Week 16 lies within relative month 4; week 26 is approximately six calendar months. The work budget includes all 32 tasks and an explicit contingency pool.

| Milestone | Weeks | Issues | Task h | Reserve h | Total h | Exit |
| --- | --- | --- | ---: | ---: | ---: | --- |
| M1 Scope/contracts | 1-2 | FT-001..003 | 24 | 8 | 32 | Planning publication prepared, rights rules and contracts frozen |
| M2 Data/intake | 3-6 | FT-004..007 | 46 | 18 | 64 | Exact synthetic labels, disjoint splits, safe raster path |
| M3 Extraction/evaluation | 7-10 | FT-008..012 | 58 | 6 | 64 | OCR contract, rules, abstention, reproducible measurement |
| M4 Review/export | 11-13 | FT-013..016 | 44 | 4 | 48 | Usable review, audit, exports and isolated API |
| M5 Public result | 14-16 | FT-017..020 | 48 | 0 | 48 | Working public sample demo and local release, measured limitations |
| M6 Validation | 17-20 | FT-021..024 | 44 | 20 | 64 | Time study, external-evidence decision, local deployment rehearsal |
| M7 Private readiness | 21-23 | FT-025..028 | 36 | 12 | 48 | Authentication, isolation, migrations, operating limits |
| M8 Maintenance/handoff | 24-26 | FT-029..032 | 36 | 12 | 48 | License/regression/update drills and final product decision |
| Total | 26 weeks | 32 tasks | 336 | 80 | 416 | One active project at 16 h/week |

M1-M5: 220 task hours + 36 reserve = 256 hours / 16 = 16 weeks. M6-M8: 116 + 44 = 160 hours / 16 = 10 weeks. Reserve is 80/416 = 19.23%, approximately 20%. Backlog identity: sum(issue.estimate_hours) + reserve_hours = total_hours. All issues are 6-12 hours and type:task; no epics, uncounted mandatory child work or hidden implementation budget exists.

Estimates include coding, tests, review and issue-specific documentation. Reserve is exclusively unplanned rework/contingency, not omitted features. Pool reserves across gates, including M5, even though they are shown where capacity is held. Track actuals and reserve draw in a future planning/actuals.csv with issue, hours, reason and remaining budget. Record scheduled-but-unused reserve as slack, not hours worked. When a slice exceeds its estimate, narrow it or split the remaining work without adding hours silently.

Suggested 80-hour risk envelope: OCR packaging/license/integration 24h; geometry/domain-gap failures 20h; parser/deployment/security defects 20h; accessibility/release/operational rework 16h. These are allocations of the same reserve, not additional tasks or epics.

If the month-4 release is threatened, drop optional public-corpus ingestion, automatic acceptance and advanced private operation before dropping evidence, review, safety, export or reproducibility. Do not count a README, recording or precomputed screenshot as a functional result. Record a revised forecast if a release gate fails; first functional release must remain between months 3 and 12. If the 416-hour cap cannot cover the minimum useful release, issue a documented stop/re-scope decision rather than add unbudgeted months or activate other projects concurrently.

## 8. Execution, testing and release

Backlog order encodes dependencies. Each issue body specifies implementation steps, output paths and acceptance tests. Begin with the contracts and fixtures; keep one cohesive extraction pipeline. Do not rewrite architecture or introduce another document family when a bounded task hits uncertainty.

Required verification layers:

1. Unit/property checks for money/date normalization, missing fields, conflicting candidates, clipping/transforms and policy state transitions.
2. Contract tests for OCR adapters and HTTP payloads; no dependency on paid APIs.
3. Integration tests with synthetic documents and a real licensed OCR adapter, plus bounded error/timeout fixtures.
4. Browser end-to-end tests for sample/upload -> evidence -> correction -> confirmation -> JSON/CSV, keyboard navigation and stale revisions.
5. Security tests for payload sizes, malformed PDFs, parser deadlines, path traversal, unauthorized reads, formula injection, public upload bypass and deletion.
6. Frozen evaluation and deployment smoke checks at M5; focused regression at later releases.

CSV cells from user/OCR strings must neutralize spreadsheet formulas beginning with =, +, -, @, tab or carriage return while JSON preserves canonical values. Serialize structured audit events separately as JSON, with a revision reference from CSV. Never silently overwrite a stale review.

Public release assets: source tag, pinned container digest/build instructions, sample corpus with rights manifest, generator invocation, held-out evaluation command and machine-readable report, synthetic failure gallery, limitation/data card, working demo URL if available, local run command, operational/deletion instructions and release notes. Do not commit large corpora or private data; distribute only licensed samples and reproducible generation instructions.

Public deployment: non-root container, read-only filesystem except bounded temp/storage, controlled worker limits, no secrets in images, sample-only route enforcement, TLS, request quotas, one maximum instance and a manual/automated service-disable switch. Provider budget alerts alone are not hard spending caps. Verify actual billing/CPU settings and fresh quotas before enabling the deployment; benchmark resource choices instead of asserting free hosting. Rehearse reverting to a previous digest and make sample-demo downtime explicit if rollback occurs.

No sensitive uploads on public infrastructure. Local upload runs with outbound network blocked by default. Any future managed OCR adapter needs separately approved resource rights, privacy terms, explicit operator opt-in, region/retention documentation and a re-estimated budget; it is outside this baseline.

## 9. Validation, operations and maintainability

M6 evaluates simulated workflow utility and optional existing-label public evidence. M7 prepares a single-organization private deployment with authenticated access, owner-scoped records, retention, migration backups and error/cost limits. If no private environment is available, a local authenticated rehearsal satisfies technical readiness; it does not prove customer adoption or real-world accuracy.

Store structured operational metrics without document contents. Track processing failures, queue rejection, p95 latency, expiring files, version and daily resource totals. Keep dependency locks, license manifests and configuration examples versioned. Use SQLite migrations with schema version checks; back up before migration, restore in a rehearsal, and explicitly account for data contained in backups.

M8 rehearses a dependency update and rollback on generated fixtures, repeats frozen gates, rechecks attribution/redistribution, and writes the final handoff/decision. After week 26, propose a separate maintenance allocation of two hours/week within the same global 16-hour budget if the owner continues. That future recurring allocation is not included in the 416 hours and is not automatically authorized. If the project is parked, provide shutdown instructions and retain a runnable tagged local artifact; no unattended paid service is presumed.

Risks and responses: synthetic-to-real gap -> restrict claims, evaluate existing public labels only where compatible; OCR licensing/packaging -> bounded adapter integration and reserve, never unlicensed fallback; layout leakage -> family-separated data and sealed tests; field ambiguity -> abstention and visible evidence; privacy -> public synthetic-only mode and local processing; poor economics -> measured runtime/cost and bounded scale; absent demand -> mark unvalidated and park rather than invent interviews or traction.

## 10. Sources and verification limits

Only primary sources needed for this plan were checked on 2026-10-19. Current product pricing, package versions, resource licenses and service quotas must be checked again in the corresponding implementation issue. No commercial-use permission is presumed from a link alone.

- [S1: Official DocLayNet repository](https://github.com/DS4SD/DocLayNet): public layout dataset and annotation scope; candidate diagnostic only, not invoice-field ground truth.
- [S2: Official DocLayNet license](https://github.com/DS4SD/DocLayNet/blob/main/LICENSE): CDLA-Permissive-1.0, including retention of attribution, license access and modification notices. Admission still checks the precise version, asset coverage and intended use.
- [S3: Cloud Run container deployment documentation](https://docs.cloud.google.com/run/docs/deploying): supports the proposed container deployment route. No hosting price, free-tier promise, latency or availability is asserted here.
- [S4: Official Tesseract engine](https://github.com/tesseract-ocr/tesseract): version-5 engine, TSV output and Apache-2.0 license; checked 2026-10-20.
- [S5: Official tessdata_fast resources](https://github.com/tesseract-ocr/tessdata_fast): Portuguese traineddata, LSTM-engine compatibility and separate Apache-2.0 license; checked 2026-10-20.
- [S6: Official Tesseract CLI documentation](https://tesseract-ocr.github.io/tessdoc/Command-Line-Usage.html): language, output and page-segmentation configuration; checked 2026-10-20.
- [Public FormTrace repository](https://github.com/123ssk321/cv-formtrace): creation and public visibility confirmed by the parent.

The source report's annotation-origin exclusions are superseded here. Existing public labels can support commercial development when the actual license and covered rights permit it; noncommercial data cannot be repurposed by renaming its use evaluation.

