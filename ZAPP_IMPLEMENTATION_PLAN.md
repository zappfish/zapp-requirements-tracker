# ZAPP Implementation Plan

> 🤖 Generated with support from Claude AI. This is not set in stone and meant to be a support for planning. Connect with Sarah with any concerns. 
> 
> Suggested task ordering, architecture recommendations, and priority assignments for the Zebrafish Phenotype Atlas Project (ZAPP).
>
> **Source issues:** [#16](https://github.com/zappfish/zapp-requirements-tracker/issues/16) (Data Model), [#15](https://github.com/zappfish/zapp-requirements-tracker/issues/15) (Ontologies), [#32](https://github.com/zappfish/zapp-requirements-tracker/issues/32) (Data Storage), [#44](https://github.com/zappfish/zapp-requirements-tracker/issues/44) (Publish Data), [#49](https://github.com/zappfish/zapp-requirements-tracker/issues/49) (Data Input), [#72](https://github.com/zappfish/zapp-requirements-tracker/issues/72) (Atlas UI)
>
> Generated: 2026-02-10

---

<details>
<summary><h2>Current Architecture Inventory</h2></summary>

| Repo | Stack | Role |
|------|-------|------|
| `zebrafish-toxicology-atlas-schema` | **LinkML** (YAML schema), Python, justfile | Biological data model — already defines Study, Experiment, ExposureEvent, Phenotype, Fish, Chemical, Image, etc. |
| `zapp-atlas` | **Flask** (Python server), **React + Vite + TypeScript** (client), **Zod** validation | Main application: submission form + atlas viewer |
| `frogpot` | **React + TypeScript**, published to npm | Phenotype picker component (ontology tree browser) |
| `phenotype-picker` | Deployment mirror of zapp-atlas | Demo deployment of the frogpot component |
| `zappfish.github.io` | Static site | Project website |

</details>

---

<details>
<summary><h2>Priority Definitions</h2></summary>

| Priority | Meaning | MVP? |
|----------|---------|------|
| **P1** | Foundational — blocks other work. Must be done first. | Pre-MVP |
| **P2** | Core MVP features — the minimum for a usable form + atlas. | Pre-MVP |
| **P3** | MVP completion — polish, secondary workflows, and remaining MVP-labeled items. | Pre-MVP |
| **P4** | Post-MVP — items without an MVP label. Future enhancements. | Post-MVP |

> **Rule:** Every issue labeled `MVP Form` or `MVP Atlas` is P1, P2, or P3.
> Issues with neither label are P4.

</details>

---

<details>
<summary><h2>P1 — Foundations</h2></summary>

Everything else depends on these. Work the three tracks in parallel where possible.

<details>
<summary><h3>P1-A: Finalize the Biological Schema (LinkML)</h3></summary>

> Parent: [#16](https://github.com/zappfish/zapp-requirements-tracker/issues/16) Data Model / [#17](https://github.com/zappfish/zapp-requirements-tracker/issues/17) Biological Schema
> Labels: MVP Form, MVP Atlas

The LinkML schema in `zebrafish-toxicology-atlas-schema` already models Study, Experiment, ExposureEvent, Phenotype, Fish, Chemical, and Image. This track validates completeness and fills gaps.

| Order | Task | Issue | What to do |
|-------|------|-------|------------|
| 1 | Validate user/submitter identity slots | [#18](https://github.com/zappfish/zapp-requirements-tracker/issues/18) | Confirm `annotator` (ORCID) + `Study.experiment` grouping covers ORCID IDs, study ID, experiment-within-study nesting. |
| 2 | Validate control vs. experimental modeling | [#19](https://github.com/zappfish/zapp-requirements-tracker/issues/19) | Confirm `Control` and `ExposureEvent` are clearly distinguished in the schema; ensure the relationship between them within an `Experiment` is unambiguous. |
| 3 | Validate fish information capture | [#20](https://github.com/zappfish/zapp-requirements-tracker/issues/20) | `Fish` has ZFIN pattern. Add slots if needed: genetics/alleles/transgene references, rearing conditions, fallback ID for fish not yet in ZFIN. |
| 4 | Validate image capture | [#21](https://github.com/zappfish/zapp-requirements-tracker/issues/21) | Confirm `Image` vs `ControlImage` distinction is clear. One fish per image constraint — add documentation or validation. |
| 5 | Validate phenotype capture | [#22](https://github.com/zappfish/zapp-requirements-tracker/issues/22) | `PhenotypeObservationSet` → `Phenotype` (multivalued). Confirm frogpot output maps cleanly. |
| 6 | Validate exposure chemical info | [#23](https://github.com/zappfish/zapp-requirements-tracker/issues/23) | `ExposureEvent.stressor` → `StressorChemical` chain + `Regimen`. Check completeness. |
| 7 | Validate control chemical info | [#24](https://github.com/zappfish/zapp-requirements-tracker/issues/24) | `Control.vehicle_if_treated` as `VehicleEnumeration`. Extend enum values as needed. |
| 8 | Validate chemical entity info | [#25](https://github.com/zappfish/zapp-requirements-tracker/issues/25) | `ChemicalEntity` has chebi_id, cas_id, chemical_name. Add fallback UUID strategy for chemicals lacking standard IDs. Manufacturer slot exists. |

**Architecture:**
- **LinkML** remains the single source of truth for the biological schema.
- Use **linkml-runtime** for Python dataclass generation + instance validation.
- Use **gen-json-schema** to produce JSON Schema → feed into Zod schemas for client-side validation.
- Use **gen-sqltables** (linkml-sqlutils) to generate SQL DDL, keeping DB and schema in sync (Monarch pattern).

</details>

<details>
<summary><h3>P1-B: Core Ontology Preparation</h3></summary>

> Parent: [#15](https://github.com/zappfish/zapp-requirements-tracker/issues/15) Ontologies / [#1](https://github.com/zappfish/zapp-requirements-tracker/issues/1) ZP / [#6](https://github.com/zappfish/zapp-requirements-tracker/issues/6) Other Ontology Needs
> Labels: MVP Form, MVP Atlas

These ontology data sets are needed by both the submission form (autocompletes) and the atlas (search/filtering).

| Order | Task | Issue | What to do |
|-------|------|-------|------------|
| 1 | ZP-ZAPP Quality Control | [#2](https://github.com/zappfish/zapp-requirements-tracker/issues/2) | Use **OAK** to validate: all terms have labels, correct subset is extracted. Automated QC pipeline. |
| 2 | Prepare Chemical Ontologies (CAS → PubChem → CHEBI) | [#7](https://github.com/zappfish/zapp-requirements-tracker/issues/7) | Use **SRI Node Normalizer** or **Monarch Name Resolution** for ID mapping. Build cached lookup table. Define fallback for missing IDs. |
| 3 | Prepare ZFA for anatomy filtering/picking | [#9](https://github.com/zappfish/zapp-requirements-tracker/issues/9) | Already used by frogpot. Ensure hierarchy is loaded, up to date, and searchable. |
| 4 | Prepare ZFS for developmental stages | [#10](https://github.com/zappfish/zapp-requirements-tracker/issues/10) | Confirm ZFS covers all stages referenced in exposures/phenotypes. Load as OAK SQLite. |
| 5 | Prepare ECTO for exposure route/type/regimen | [#8](https://github.com/zappfish/zapp-requirements-tracker/issues/8) | Schema already uses `ExposureTypeEnum.reachable_from` on ECTO. Extract relevant subset via OAK. |
| 6 | Prepare ZFIN IDs — types needed for MVP | [#12](https://github.com/zappfish/zapp-requirements-tracker/issues/12), [#13](https://github.com/zappfish/zapp-requirements-tracker/issues/13) | **MVP needs:** Labs (provenance), WT genotype, Fish IDs. Alleles/Tg/Gene are secondary per the issue table. |
| 7 | PMID/DOI for provenance | [#11](https://github.com/zappfish/zapp-requirements-tracker/issues/11) | Validate `Study.publication` — add pattern constraint for PMID, DOI, or "not published". |

**Architecture:**
- **OAK (Ontology Access Kit)** for all ontology operations: QC, subsetting, term lookup, hierarchy traversal. Monarch standard tool.
- **Semantic SQL** (OAK's SQL backend) for fast local ontology querying in server endpoints.
- Cache subsets as **SQLite databases** (OAK's default format) — serve via lightweight autocomplete API endpoints.
- For chemical ID resolution: call **SRI Node Normalizer** (`https://nodenormalization-sri.renci.org`) and cache results.

</details>

<details>
<summary><h3>P1-C: Authentication & Deployment Infrastructure</h3></summary>

> Parent: [#32](https://github.com/zappfish/zapp-requirements-tracker/issues/32) Data Storage / [#33](https://github.com/zappfish/zapp-requirements-tracker/issues/33) Deployment / [#42](https://github.com/zappfish/zapp-requirements-tracker/issues/42) Authentication
> Labels: MVP Form (some also MVP Atlas)

| Order | Task | Issue | Labels | What to do |
|-------|------|-------|--------|------------|
| 1 | ORCID OAuth 2.0 authentication | [#43](https://github.com/zappfish/zapp-requirements-tracker/issues/43) | MVP Form | Replace credential-file auth in Flask with ORCID OAuth authorization code flow. |
| 2 | Docker image | [#34](https://github.com/zappfish/zapp-requirements-tracker/issues/34) | MVP Form, MVP Atlas | Containerize Flask server + built React client. Multi-stage build. |
| 3 | Web host selection & setup | [#35](https://github.com/zappfish/zapp-requirements-tracker/issues/35) | MVP Form, MVP Atlas | Evaluate: cloud VM, managed containers, or institutional hosting. Deploy Docker image. |

**Architecture:**
- **ORCID Public API** with OAuth 2.0 authorization code flow. Use Python `authlib` library with Flask.
- Store sessions via **Flask-Session** with a database backend, or use **JWT** tokens.
- **Docker:** Multi-stage Dockerfile — Stage 1: `npm run build` (Vite client), Stage 2: Python image with Flask serving static + API.
- Consider **Traefik** or **Caddy** as reverse proxy with automatic TLS.

</details>

</details>

---

<details>
<summary><h2>P2 — Core MVP Features</h2></summary>

These deliver the usable submission form and atlas. Depend on P1 foundations being stable.

<details>
<summary><h3>P2-A: Persistence Layer</h3></summary>

> Parent: [#36](https://github.com/zappfish/zapp-requirements-tracker/issues/36) Persistence Layer
> Labels: MVP Form, MVP Atlas

| Order | Task | Issue | Labels | What to do |
|-------|------|-------|--------|------------|
| 1 | Store observations — DB selection + migrations | [#37](https://github.com/zappfish/zapp-requirements-tracker/issues/37) | MVP Form, MVP Atlas | Migrate from current filesystem JSON storage to relational DB. Set up migration framework. |
| 2 | Image storage & normalization | [#40](https://github.com/zappfish/zapp-requirements-tracker/issues/40) | MVP Form, MVP Atlas | TIFF → PNG/JPEG conversion. Store originals + web copies. Define size limits. |
| 3 | User/Person/Submitter table | [#30](https://github.com/zappfish/zapp-requirements-tracker/issues/30) | MVP Form, MVP Atlas | ORCID profile data for ZAPP users (name, lab affiliation). Links to submissions. |
| 4 | Store partial / draft observations | [#38](https://github.com/zappfish/zapp-requirements-tracker/issues/38) | MVP Form | Draft state with `status` flag (draft → submitted → published). Enables save-and-resume. |
| 5 | Cache ontology data for autocompletes | [#41](https://github.com/zappfish/zapp-requirements-tracker/issues/41) | MVP Form, MVP Atlas | Preload OAK-generated SQLite ontology DBs. Serve via search endpoints. |
| 6 | Non-frogpot autocomplete storage | [#28](https://github.com/zappfish/zapp-requirements-tracker/issues/28) | MVP Form, MVP Atlas | Storage for chemical, ZFIN, and other autocomplete data not handled by frogpot. |
| 7 | Audit logging table(s) | [#27](https://github.com/zappfish/zapp-requirements-tracker/issues/27) | MVP Form, MVP Atlas | Track who changed what and when. `change_log` table. |
| 8 | Warehouse / denormalized search tables | [#29](https://github.com/zappfish/zapp-requirements-tracker/issues/29) | MVP Form, MVP Atlas | Evaluate need. If PostgreSQL full-text + closure tables are fast enough, defer. |

**Architecture:**
- **PostgreSQL** as primary database.
- **linkml-sqlutils** to generate SQL schema from LinkML (Monarch pattern: schema → DDL, automatically).
- **Alembic** for migrations (works with SQLAlchemy, which linkml-sqlutils generates).
- **Object storage** for images: S3-compatible (MinIO self-hosted or AWS S3). Metadata in PostgreSQL. **Pillow** for TIFF→PNG/JPEG conversion.
- Audit: PostgreSQL trigger-based audit table or application-level logging.
- Autocomplete: OAK SQLite databases served via Flask endpoints with prefix-search.

</details>

<details>
<summary><h3>P2-B: Technical Schema Additions</h3></summary>

> Parent: [#26](https://github.com/zappfish/zapp-requirements-tracker/issues/26) Technical/Application Schema
> Labels: MVP Form, MVP Atlas

| Task | Issue | What to do |
|------|-------|------------|
| "My Stuff" schema (chemical cabinet, fish tank) | [#31](https://github.com/zappfish/zapp-requirements-tracker/issues/31) | Add LinkML classes for user-scoped saved items. Links to [#66](https://github.com/zappfish/zapp-requirements-tracker/issues/66). |

> Note: Issues #27, #28, #29, #30 are covered in P2-A (Persistence Layer) above since they're implementation tasks, not schema-only.

**Architecture:**
- Extend the LinkML schema with a `UserProfile` class and `SavedItem` (chemical cabinet / fish tank) classes.
- Keep these in the `zebrafish-toxicology-atlas-schema` repo alongside biological classes, but in a separate schema module if desired.

</details>

<details>
<summary><h3>P2-C: Submission Form — Core Workflow</h3></summary>

> Parent: [#49](https://github.com/zappfish/zapp-requirements-tracker/issues/49) Data Input / [#51](https://github.com/zappfish/zapp-requirements-tracker/issues/51) Submission Forms
> Labels: MVP Form

Build the multi-step submission wizard in the React client.

| Order | Task | Issue | What to do |
|-------|------|-------|------------|
| 1 | Entry point & how-to documentation | [#50](https://github.com/zappfish/zapp-requirements-tracker/issues/50) | Landing page explaining the submission process and linking to the form. |
| 2 | Login with ORCID to submit | [#52](https://github.com/zappfish/zapp-requirements-tracker/issues/52) | Integrate ORCID auth (from P1-C) into the submission flow. Gate form behind login. |
| 3 | Enter control info (fish, vehicle, image, type) | [#60](https://github.com/zappfish/zapp-requirements-tracker/issues/60) | Form section with ZFIN fish autocomplete, vehicle enum dropdown, image upload, control type. |
| 4 | Enter chemical info (ID, name, concentration, vehicle) | [#61](https://github.com/zappfish/zapp-requirements-tracker/issues/61) | Chemical autocomplete (CHEBI/CAS/PubChem via P1-B mapping). Concentration + unit input. |
| 5 | Enter exposure details (route, regimen, pattern) | [#62](https://github.com/zappfish/zapp-requirements-tracker/issues/62) | ECTO-backed autocompletes for route/type. Structured regimen input (start/end stage, duration, pattern). |
| 6 | Enter phenotype info (term, prevalence, severity, stage) | [#63](https://github.com/zappfish/zapp-requirements-tracker/issues/63) | Use **frogpot** for ZP term selection. ZFS for stage. Prevalence + severity inputs. |
| 7 | Upload images (crop, multi-upload) | [#59](https://github.com/zappfish/zapp-requirements-tracker/issues/59) | Client-side crop tool, multi-file upload for control + phenotype images. |
| 8 | Save & resume later | [#57](https://github.com/zappfish/zapp-requirements-tracker/issues/57) | Auto-save drafts to server. Depends on P2-A partial storage (#38). |

**Architecture:**
- Build as a **multi-step wizard** in React: Study metadata → Fish & Control → Exposure & Chemical → Phenotype & Images → Review & Submit.
- **Zod** (already a dependency) for client-side validation. Generate Zod schemas from LinkML's JSON Schema output (`gen-json-schema`) to keep validation in sync.
- **frogpot** (already an npm dependency) for the phenotype picker step.
- Autocomplete endpoints in Flask querying OAK SQLite caches.
- Image crop: use `react-easy-crop` or similar lightweight React library.
- Draft auto-save: debounced POST to a `/api/drafts` endpoint every 30s or on step change.

</details>

<details>
<summary><h3>P2-D: Atlas UI — Core Search & View</h3></summary>

> Parent: [#72](https://github.com/zappfish/zapp-requirements-tracker/issues/72) Atlas / [#75](https://github.com/zappfish/zapp-requirements-tracker/issues/75) Navigation / [#76](https://github.com/zappfish/zapp-requirements-tracker/issues/76) Search / [#77](https://github.com/zappfish/zapp-requirements-tracker/issues/77) Viewing
> Labels: MVP Atlas

| Order | Task | Issue | What to do |
|-------|------|-------|------------|
| 1 | Atlas entry point | [#73](https://github.com/zappfish/zapp-requirements-tracker/issues/73) | Unified entry point connecting atlas + submission form. |
| 2 | Determine what data is shown | [#74](https://github.com/zappfish/zapp-requirements-tracker/issues/74) | Only published observations. Define the "card" format for results. |
| 3 | Homepage (showcase images/submissions) | [#78](https://github.com/zappfish/zapp-requirements-tracker/issues/78) | Featured/recent submissions with images. |
| 4 | Search criteria definition | [#85](https://github.com/zappfish/zapp-requirements-tracker/issues/85) | By submitter (ORCID), lab (ZFIN), phenotype (ZP), chemical, fish type, study ID, anatomy. |
| 5 | Search method (text + controlled vocab + browse) | [#86](https://github.com/zappfish/zapp-requirements-tracker/issues/86) | Full-text search + faceted filtering via ontology hierarchies. |
| 6 | Search results display (image-centric) | [#87](https://github.com/zappfish/zapp-requirements-tracker/issues/87) | Image-anchored result cards showing fish, chemical/exposure, phenotype metadata. |
| 7 | SPA architecture (live AJAX updates) | [#89](https://github.com/zappfish/zapp-requirements-tracker/issues/89) | Already a React SPA. Use client-side routing with URL state for search params. |
| 8 | Study pages (all images from one study) | [#90](https://github.com/zappfish/zapp-requirements-tracker/issues/90) | Aggregate view. Ability to download the set. Route: `/study/:uuid`. |
| 9 | Individual image pages | [#91](https://github.com/zappfish/zapp-requirements-tracker/issues/91) | Detail page per image with all associated metadata. Route: `/image/:uuid`. |
| 10 | Persistent URLs / CURIEs | [#92](https://github.com/zappfish/zapp-requirements-tracker/issues/92) | e.g., `https://zappfish.org/submission/1234`. Use UUIDs from schema. |

**Architecture:**
- **React Router** for client-side routing. Encode search state in URL query params for shareability ([#88](https://github.com/zappfish/zapp-requirements-tracker/issues/88)).
- For search: **PostgreSQL full-text search** + ontology-aware queries using closure tables (generated from OAK). If dataset grows large, consider **Solr** (used by Monarch) or Elasticsearch.
- Flask search API: accepts faceted query params, returns paginated results as JSON.
- Image routes: `/image/:uuid`, study routes: `/study/:uuid`.

</details>

</details>

---

<details>
<summary><h2>P3 — MVP Completion</h2></summary>

Remaining MVP-labeled items. These round out the user experience before launch.

<details>
<summary><h3>P3-A: Submission Form — Advanced Workflows</h3></summary>

> Labels: MVP Form

| Task | Issue | What to do |
|------|-------|------------|
| Review submitted data ("My Submissions" dashboard) | [#53](https://github.com/zappfish/zapp-requirements-tracker/issues/53) | List all submissions by the logged-in user with status indicators. |
| Edit submitted data | [#54](https://github.com/zappfish/zapp-requirements-tracker/issues/54) | Re-open a submission for editing. Track changes via audit log. |
| Reuse previous submission as template | [#56](https://github.com/zappfish/zapp-requirements-tracker/issues/56) | "Clone submission" button that pre-fills the form with data from a previous entry. |
| Delay publishing / embargo | [#58](https://github.com/zappfish/zapp-requirements-tracker/issues/58) | Add `publish_date` field. Submissions hidden from atlas until date passes or user explicitly publishes. |
| Request new phenotype term or name | [#64](https://github.com/zappfish/zapp-requirements-tracker/issues/64) | In-app form that opens a GitHub issue or emails curators with the term request. |
| Submitter data download | [#65](https://github.com/zappfish/zapp-requirements-tracker/issues/65) | Export own submitted data as TSV or JSON. |
| "My Stuff" UI (chemical cabinet, fish tank) | [#66](https://github.com/zappfish/zapp-requirements-tracker/issues/66) | User-scoped saved items with nicknames for quick reuse in the form. Depends on P2-B schema. |
| Data load procedures for non-frogpot autocompletes | [#70](https://github.com/zappfish/zapp-requirements-tracker/issues/70) | ETL pipeline (using OAK) to refresh ontology caches on a schedule. |

</details>

<details>
<summary><h3>P3-B: Atlas UI — Navigation, Info & Polish</h3></summary>

> Labels: MVP Atlas

| Task | Issue | What to do |
|------|-------|------------|
| How to use the Atlas (tutorial) | [#79](https://github.com/zappfish/zapp-requirements-tracker/issues/79) | Guided tour or help page. |
| Information / About page | [#80](https://github.com/zappfish/zapp-requirements-tracker/issues/80) | Project purpose, citation guide, re-use policy, help links. Link to zappfish.org. |
| Data browsing by ontology slims | [#81](https://github.com/zappfish/zapp-requirements-tracker/issues/81) | Browse by upper-level categories (anatomy, chemical). Use OAK slims. |
| Atlas data download | [#82](https://github.com/zappfish/zapp-requirements-tracker/issues/82) | Download from search results or study pages. |
| Connection to submission form | [#83](https://github.com/zappfish/zapp-requirements-tracker/issues/83) | "Submit your data" CTA prominently on the atlas. |
| Connection to zappfish.org | [#84](https://github.com/zappfish/zapp-requirements-tracker/issues/84) | Navigation link to the project site. |
| Search persistence & shareability | [#88](https://github.com/zappfish/zapp-requirements-tracker/issues/88) | Search params properly encoded in URL. Copy-link button. |
| Observation diff view | [#93](https://github.com/zappfish/zapp-requirements-tracker/issues/93) | Side-by-side comparison of two+ observations. Download the comparison set. |

</details>

<details>
<summary><h3>P3-C: Ontology Amendment & Publishing</h3></summary>

> Labels: MVP Form (for #3), MVP Form + MVP Atlas (for #5)

| Task | Issue | What to do |
|------|-------|------------|
| ZP synonym/label amendment process | [#3](https://github.com/zappfish/zapp-requirements-tracker/issues/3) | Define curation workflow: add synonyms, promote to preferred labels, automate new submissions to curators. Use OAK + GitHub Issues. |
| Publish ZAPP subset of ZP | [#5](https://github.com/zappfish/zapp-requirements-tracker/issues/5) | Determine cadence (quarterly). Publish to Monarch OLS. Automated release via OAK. |

</details>

<details>
<summary><h3>P3-D: Data Publishing & Download</h3></summary>

> Labels: MVP Form + MVP Atlas (for #45, #46, #47), MVP Atlas (for #48)

| Task | Issue | What to do |
|-------|------|------------|
| Determine if we need an API | [#45](https://github.com/zappfish/zapp-requirements-tracker/issues/45) | **Recommendation: Yes.** A REST API is needed for the SPA client anyway; expose it for external consumers. Consider using **linkml-store** API capabilities or a FastAPI layer. |
| Data download for individual users | [#47](https://github.com/zappfish/zapp-requirements-tracker/issues/47) | Per-study or per-search-result export (TSV, JSON-LD). For publications. |
| Data download for bulk consumers | [#48](https://github.com/zappfish/zapp-requirements-tracker/issues/48) | Full dataset dump. Publish as **KGX** or **JSON-LD** for Monarch/Biolink interoperability. |

</details>

</details>

---

<details>
<summary><h2>P4 — Post-MVP</h2></summary>

Issues without `MVP Form` or `MVP Atlas` labels. Plan these after launch.

<details>
<summary><h3>P4-A: Advanced Permissions & Collaboration</h3></summary>

| Task | Issue | What to do |
|------|-------|------------|
| Determine permissions model for updating | [#39](https://github.com/zappfish/zapp-requirements-tracker/issues/39) | Role-based access: submitter, lab editor, admin. Allow lab members to edit each other's submissions. |
| Set permissions for others to edit | [#55](https://github.com/zappfish/zapp-requirements-tracker/issues/55) | UI for granting edit access at the lab or individual level. |

</details>

<details>
<summary><h3>P4-B: Bulk Upload Pipeline</h3></summary>

| Task | Issue | What to do |
|------|-------|------------|
| Bulk uploads (super) | [#67](https://github.com/zappfish/zapp-requirements-tracker/issues/67) | — |
| Templates for researchers | [#68](https://github.com/zappfish/zapp-requirements-tracker/issues/68) | Provide TSV/Excel templates matching the LinkML schema. Use **linkml-csv** or **linkml-transformer** to validate. |
| Determine bulk upload format | [#69](https://github.com/zappfish/zapp-requirements-tracker/issues/69) | Recommend **TSV with LinkML validation** as primary. API for programmatic access. |

</details>

<details>
<summary><h3>P4-C: Ontology Governance — External Processes</h3></summary>

| Task | Issue | What to do |
|------|-------|------------|
| Process for adding new ZP ontology terms | [#4](https://github.com/zappfish/zapp-requirements-tracker/issues/4) | Issue template + OAK validation pipeline for proposed new terms. |
| Communication with ZFIN for new terms | [#14](https://github.com/zappfish/zapp-requirements-tracker/issues/14) | Establish relationship. Define shepherding process for new terms not yet in ZFIN. |

</details>

</details>

---

<details>
<summary><h2>Suggested Execution Order</h2></summary>

```
P1 — Foundations (parallel tracks)
│
├─ P1-A: Schema validation & gap-filling (LinkML)
│   Issues: #17, #18, #19, #20, #21, #22, #23, #24, #25
│
├─ P1-B: Ontology preparation (OAK + Node Normalizer)    ← parallel with P1-A
│   Issues: #1, #2, #6, #7, #8, #9, #10, #11, #12, #13
│
└─ P1-C: Auth (ORCID) + Docker + Hosting                 ← parallel with P1-A, P1-B
    Issues: #42, #43, #33, #34, #35

P2 — Core MVP (start when respective P1 tracks are stable)
│
├─ P2-A: Persistence layer (PostgreSQL + linkml-sqlutils)
│   Issues: #36, #37, #38, #40, #41, #28, #27, #29, #30
│   Depends on: P1-A (schema finalized)
│
├─ P2-B: Technical schema additions
│   Issues: #26, #31
│   Depends on: P1-A
│
├─ P2-C: Submission form — core workflow                  ← parallel with P2-D
│   Issues: #50, #51, #52, #57, #59, #60, #61, #62, #63
│   Depends on: P2-A (persistence), P1-B (autocompletes), P1-C (auth)
│
└─ P2-D: Atlas UI — core search & view                   ← parallel with P2-C
    Issues: #73, #74, #75, #76, #77, #78, #85, #86, #87, #89, #90, #91, #92
    Depends on: P2-A (persistence), P1-B (ontology data)

P3 — MVP Completion (after P2 core is functional)
│
├─ P3-A: Submission form — advanced workflows
│   Issues: #53, #54, #56, #58, #64, #65, #66, #70
│
├─ P3-B: Atlas UI — navigation, info, polish
│   Issues: #79, #80, #81, #82, #83, #84, #88, #93
│
├─ P3-C: Ontology amendment & publishing
│   Issues: #3, #5
│
└─ P3-D: Data publishing & download
    Issues: #45, #46, #47, #48

P4 — Post-MVP
│
├─ P4-A: Permissions & collaboration (#39, #55)
├─ P4-B: Bulk upload pipeline (#67, #68, #69)
└─ P4-C: Ontology governance — external (#4, #14)
```

</details>

---

<details>
<summary><h2>Architecture Summary: Monarch-Aligned Stack</h2></summary>

```
┌─────────────────────────────────────────────────────────────┐
│                        Client (React)                       │
│  zapp-atlas/client  •  Vite  •  TypeScript  •  Zod          │
│  frogpot (phenotype picker)  •  React Router                │
│  Zod schemas ← generated from LinkML JSON Schema            │
└────────────────────────┬────────────────────────────────────┘
                         │ REST API
┌────────────────────────┴────────────────────────────────────┐
│                     Server (Flask / Python)                  │
│  ORCID OAuth (authlib)  •  Submission API  •  Search API    │
│  Image upload + TIFF conversion (Pillow)                    │
│  linkml-runtime (validation)                                │
│  OAK (ontology queries, autocomplete)                       │
└──────┬──────────────┬──────────────────┬────────────────────┘
       │              │                  │
┌──────┴──────┐ ┌─────┴──────┐ ┌────────┴────────┐
│ PostgreSQL  │ │ Object     │ │ Ontology Cache  │
│ (via        │ │ Storage    │ │ (SQLite via     │
│ linkml-     │ │ (S3/MinIO) │ │  OAK/Semantic   │
│ sqlutils)   │ │ Images     │ │  SQL)           │
└─────────────┘ └────────────┘ └─────────────────┘

Schema source of truth:
  zebrafish-toxicology-atlas-schema (LinkML YAML)
    ├── gen-python     → Python dataclasses (server validation)
    ├── gen-json-schema → JSON Schema → Zod (client validation)
    ├── gen-sqltables  → PostgreSQL DDL (database)
    └── gen-doc        → Documentation site
```

<details>
<summary><h3>Key Monarch Initiative Tools</h3></summary>

| Tool | Use Case | Status in ZAPP |
|------|----------|----------------|
| **LinkML** | Schema definition, validation, code generation | Already in use |
| **linkml-runtime** | Python dataclass generation, instance validation | Recommended for server |
| **linkml-sqlutils** | Generate SQL schema from LinkML, ORM integration | Recommended for DB |
| **linkml-store** | Data storage abstraction over multiple backends | Evaluate as alternative to raw sqlutils |
| **OAK (Ontology Access Kit)** | Ontology QC, subsetting, lookup, hierarchy traversal | Recommended for all ontology ops |
| **Semantic SQL** | SQL-queryable ontology databases | Recommended for autocomplete serving |
| **SRI Node Normalizer** | Chemical ID mapping (CAS → PubChem → CHEBI) | Recommended for chemical autocomplete |
| **Biolink Model** | Interoperability with Monarch knowledge graph | Reference for data export |
| **KGX** | Knowledge graph exchange format | Recommended for bulk data export (P3-D) |
| **gen-json-schema** | JSON Schema from LinkML → Zod client validation | Recommended for schema sync |
| **linkml-csv** / **linkml-transformer** | Validate and ingest bulk data uploads | Recommended for P4-B bulk uploads |

</details>

</details>

---

<details>
<summary><h2>Issue Cross-Reference Index</h2></summary>

| Issue | Title | Labels | Priority | Phase |
|-------|-------|--------|----------|-------|
| [#16](https://github.com/zappfish/zapp-requirements-tracker/issues/16) | Data Model (super) | Form, Atlas | P1 | 1-A |
| [#17](https://github.com/zappfish/zapp-requirements-tracker/issues/17) | Biological Schema - LinkML | Form, Atlas | P1 | 1-A |
| [#18](https://github.com/zappfish/zapp-requirements-tracker/issues/18) | User identifier / study / experiment | Form, Atlas | P1 | 1-A |
| [#19](https://github.com/zappfish/zapp-requirements-tracker/issues/19) | Control vs experimental modeling | Form, Atlas | P1 | 1-A |
| [#20](https://github.com/zappfish/zapp-requirements-tracker/issues/20) | Fish information | Form, Atlas | P1 | 1-A |
| [#21](https://github.com/zappfish/zapp-requirements-tracker/issues/21) | Image information | Form, Atlas | P1 | 1-A |
| [#22](https://github.com/zappfish/zapp-requirements-tracker/issues/22) | Phenotype information | Form, Atlas | P1 | 1-A |
| [#23](https://github.com/zappfish/zapp-requirements-tracker/issues/23) | Exposure chemical info | Form, Atlas | P1 | 1-A |
| [#24](https://github.com/zappfish/zapp-requirements-tracker/issues/24) | Control chemical info | Form, Atlas | P1 | 1-A |
| [#25](https://github.com/zappfish/zapp-requirements-tracker/issues/25) | Chemical entity info | Form, Atlas | P1 | 1-A |
| [#26](https://github.com/zappfish/zapp-requirements-tracker/issues/26) | Technical schema additions (super) | Form, Atlas | P2 | 2-B |
| [#27](https://github.com/zappfish/zapp-requirements-tracker/issues/27) | Audit logging | Form, Atlas | P2 | 2-A |
| [#28](https://github.com/zappfish/zapp-requirements-tracker/issues/28) | Non-frogpot autocomplete storage | Form, Atlas | P2 | 2-A |
| [#29](https://github.com/zappfish/zapp-requirements-tracker/issues/29) | Warehouse/denormalized tables | Form, Atlas | P2 | 2-A |
| [#30](https://github.com/zappfish/zapp-requirements-tracker/issues/30) | User/Person/Submitter table | Form, Atlas | P2 | 2-A |
| [#31](https://github.com/zappfish/zapp-requirements-tracker/issues/31) | "My Stuff" schema | Form, Atlas | P2 | 2-B |
| [#15](https://github.com/zappfish/zapp-requirements-tracker/issues/15) | Ontologies / External IDs (super) | Form, Atlas | P1 | 1-B |
| [#1](https://github.com/zappfish/zapp-requirements-tracker/issues/1) | ZP (super) | Form, Atlas | P1 | 1-B |
| [#2](https://github.com/zappfish/zapp-requirements-tracker/issues/2) | ZP-ZAPP QC | Form, Atlas | P1 | 1-B |
| [#3](https://github.com/zappfish/zapp-requirements-tracker/issues/3) | ZP amendment — synonyms | Form | P3 | 3-C |
| [#5](https://github.com/zappfish/zapp-requirements-tracker/issues/5) | Publish ZAPP subset | Form, Atlas | P3 | 3-C |
| [#6](https://github.com/zappfish/zapp-requirements-tracker/issues/6) | Other ontology needs (super) | Form, Atlas | P1 | 1-B |
| [#7](https://github.com/zappfish/zapp-requirements-tracker/issues/7) | Chemical ontologies | Form, Atlas | P1 | 1-B |
| [#8](https://github.com/zappfish/zapp-requirements-tracker/issues/8) | ECTO for exposure | Form, Atlas | P1 | 1-B |
| [#9](https://github.com/zappfish/zapp-requirements-tracker/issues/9) | ZFA for filtering | Form, Atlas | P1 | 1-B |
| [#10](https://github.com/zappfish/zapp-requirements-tracker/issues/10) | ZFS for stages | Form, Atlas | P1 | 1-B |
| [#11](https://github.com/zappfish/zapp-requirements-tracker/issues/11) | PMID for provenance | Form, Atlas | P1 | 1-B |
| [#12](https://github.com/zappfish/zapp-requirements-tracker/issues/12) | ZFIN IDs | Form, Atlas | P1 | 1-B |
| [#13](https://github.com/zappfish/zapp-requirements-tracker/issues/13) | ZFIN ID types | Form, Atlas | P1 | 1-B |
| [#32](https://github.com/zappfish/zapp-requirements-tracker/issues/32) | Data Storage / Retrieval (super) | Form, Atlas | P1-P2 | 1-C / 2-A |
| [#33](https://github.com/zappfish/zapp-requirements-tracker/issues/33) | Deployment (super) | Form, Atlas | P1 | 1-C |
| [#34](https://github.com/zappfish/zapp-requirements-tracker/issues/34) | Docker image | Form, Atlas | P1 | 1-C |
| [#35](https://github.com/zappfish/zapp-requirements-tracker/issues/35) | Web host | Form, Atlas | P1 | 1-C |
| [#36](https://github.com/zappfish/zapp-requirements-tracker/issues/36) | Persistence layer (super) | Form, Atlas | P2 | 2-A |
| [#37](https://github.com/zappfish/zapp-requirements-tracker/issues/37) | Store observations | Form, Atlas | P2 | 2-A |
| [#38](https://github.com/zappfish/zapp-requirements-tracker/issues/38) | Store partial observations | Form | P2 | 2-A |
| [#40](https://github.com/zappfish/zapp-requirements-tracker/issues/40) | Image storage | Form, Atlas | P2 | 2-A |
| [#41](https://github.com/zappfish/zapp-requirements-tracker/issues/41) | Autocomplete cache | Form, Atlas | P2 | 2-A |
| [#42](https://github.com/zappfish/zapp-requirements-tracker/issues/42) | Authentication (super) | Form | P1 | 1-C |
| [#43](https://github.com/zappfish/zapp-requirements-tracker/issues/43) | Register with ORCID | Form | P1 | 1-C |
| [#44](https://github.com/zappfish/zapp-requirements-tracker/issues/44) | Publish Data (super) | Form, Atlas | P3 | 3-D |
| [#45](https://github.com/zappfish/zapp-requirements-tracker/issues/45) | Determine if API needed | Form, Atlas | P3 | 3-D |
| [#46](https://github.com/zappfish/zapp-requirements-tracker/issues/46) | Data download (super) | Form, Atlas | P3 | 3-D |
| [#47](https://github.com/zappfish/zapp-requirements-tracker/issues/47) | Download for individual users | Form, Atlas | P3 | 3-D |
| [#48](https://github.com/zappfish/zapp-requirements-tracker/issues/48) | Download for bulk consumers | Atlas | P3 | 3-D |
| [#49](https://github.com/zappfish/zapp-requirements-tracker/issues/49) | Data Input / Submission (super) | Form | P2 | 2-C |
| [#50](https://github.com/zappfish/zapp-requirements-tracker/issues/50) | Submission form entry point | Form | P2 | 2-C |
| [#51](https://github.com/zappfish/zapp-requirements-tracker/issues/51) | Submission forms (super) | Form | P2 | 2-C |
| [#52](https://github.com/zappfish/zapp-requirements-tracker/issues/52) | Login with ORCID to submit | Form | P2 | 2-C |
| [#53](https://github.com/zappfish/zapp-requirements-tracker/issues/53) | Review submitted data | Form | P3 | 3-A |
| [#54](https://github.com/zappfish/zapp-requirements-tracker/issues/54) | Edit submitted data | Form | P3 | 3-A |
| [#56](https://github.com/zappfish/zapp-requirements-tracker/issues/56) | Reuse previous submissions | Form | P3 | 3-A |
| [#57](https://github.com/zappfish/zapp-requirements-tracker/issues/57) | Save & resume | Form | P2 | 2-C |
| [#58](https://github.com/zappfish/zapp-requirements-tracker/issues/58) | Delay publishing / embargo | Form | P3 | 3-A |
| [#59](https://github.com/zappfish/zapp-requirements-tracker/issues/59) | Upload images | Form | P2 | 2-C |
| [#60](https://github.com/zappfish/zapp-requirements-tracker/issues/60) | Enter control info | Form | P2 | 2-C |
| [#61](https://github.com/zappfish/zapp-requirements-tracker/issues/61) | Enter chemical info | Form | P2 | 2-C |
| [#62](https://github.com/zappfish/zapp-requirements-tracker/issues/62) | Enter exposure details | Form | P2 | 2-C |
| [#63](https://github.com/zappfish/zapp-requirements-tracker/issues/63) | Enter phenotype info | Form | P2 | 2-C |
| [#64](https://github.com/zappfish/zapp-requirements-tracker/issues/64) | Request new phenotype term | Form | P3 | 3-A |
| [#65](https://github.com/zappfish/zapp-requirements-tracker/issues/65) | Submitter download | Form | P3 | 3-A |
| [#66](https://github.com/zappfish/zapp-requirements-tracker/issues/66) | "My Stuff" UI | Form | P3 | 3-A |
| [#70](https://github.com/zappfish/zapp-requirements-tracker/issues/70) | Data load for non-frogpot autocompletes | Form | P3 | 3-A |
| [#72](https://github.com/zappfish/zapp-requirements-tracker/issues/72) | Atlas / Atlas UI (super) | Atlas | P2 | 2-D |
| [#73](https://github.com/zappfish/zapp-requirements-tracker/issues/73) | Atlas entry point | Atlas | P2 | 2-D |
| [#74](https://github.com/zappfish/zapp-requirements-tracker/issues/74) | What data is shown | Atlas | P2 | 2-D |
| [#75](https://github.com/zappfish/zapp-requirements-tracker/issues/75) | Atlas navigation (super) | Atlas | P2 | 2-D |
| [#76](https://github.com/zappfish/zapp-requirements-tracker/issues/76) | Data search (super) | Atlas | P2 | 2-D |
| [#77](https://github.com/zappfish/zapp-requirements-tracker/issues/77) | Data viewing (super) | Atlas | P2 | 2-D |
| [#78](https://github.com/zappfish/zapp-requirements-tracker/issues/78) | Homepage | Atlas | P2 | 2-D |
| [#79](https://github.com/zappfish/zapp-requirements-tracker/issues/79) | How to use Atlas | Atlas | P3 | 3-B |
| [#80](https://github.com/zappfish/zapp-requirements-tracker/issues/80) | Information page | Atlas | P3 | 3-B |
| [#81](https://github.com/zappfish/zapp-requirements-tracker/issues/81) | Data browsing | Atlas | P3 | 3-B |
| [#82](https://github.com/zappfish/zapp-requirements-tracker/issues/82) | Atlas data download | Atlas | P3 | 3-B |
| [#83](https://github.com/zappfish/zapp-requirements-tracker/issues/83) | Connection to submission form | Atlas | P3 | 3-B |
| [#84](https://github.com/zappfish/zapp-requirements-tracker/issues/84) | Connection to zappfish.org | Atlas | P3 | 3-B |
| [#85](https://github.com/zappfish/zapp-requirements-tracker/issues/85) | Search criteria | Atlas | P2 | 2-D |
| [#86](https://github.com/zappfish/zapp-requirements-tracker/issues/86) | Search method | Atlas | P2 | 2-D |
| [#87](https://github.com/zappfish/zapp-requirements-tracker/issues/87) | Search results | Atlas | P2 | 2-D |
| [#88](https://github.com/zappfish/zapp-requirements-tracker/issues/88) | Search persistence & shareability | Atlas | P3 | 3-B |
| [#89](https://github.com/zappfish/zapp-requirements-tracker/issues/89) | Search architecture (SPA) | Atlas | P2 | 2-D |
| [#90](https://github.com/zappfish/zapp-requirements-tracker/issues/90) | Study pages | Atlas | P2 | 2-D |
| [#91](https://github.com/zappfish/zapp-requirements-tracker/issues/91) | Image pages | Atlas | P2 | 2-D |
| [#92](https://github.com/zappfish/zapp-requirements-tracker/issues/92) | Persistent URLs | Atlas | P2 | 2-D |
| [#93](https://github.com/zappfish/zapp-requirements-tracker/issues/93) | Observation diff view | Atlas | P3 | 3-B |
| [#4](https://github.com/zappfish/zapp-requirements-tracker/issues/4) | New ZP ontology terms | — | P4 | 4-C |
| [#14](https://github.com/zappfish/zapp-requirements-tracker/issues/14) | New ZFIN terms | — | P4 | 4-C |
| [#39](https://github.com/zappfish/zapp-requirements-tracker/issues/39) | Update permissions | — | P4 | 4-A |
| [#55](https://github.com/zappfish/zapp-requirements-tracker/issues/55) | Others edit permissions | — | P4 | 4-A |
| [#67](https://github.com/zappfish/zapp-requirements-tracker/issues/67) | Bulk uploads (super) | — | P4 | 4-B |
| [#68](https://github.com/zappfish/zapp-requirements-tracker/issues/68) | Bulk upload templates | — | P4 | 4-B |
| [#69](https://github.com/zappfish/zapp-requirements-tracker/issues/69) | Bulk upload format | — | P4 | 4-B |

</details>
