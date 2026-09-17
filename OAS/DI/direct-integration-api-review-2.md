# Direct Integration API — Second Review (what's left)

**Files reviewed (current state)**
- `direct-integration-api-prototype.json` — the working merged spec
- `direct-integration-api.yaml` — the Wayfinder standard (fold-in source)

**Progress so far (done):** title/servers/version/overview merged; dual-auth *narrative* in place; `NHSD-End-User-Organisation` optional with conditional prose; `GET /Appointment` now has optional `_id`, `_include` (BaRS chain **and** Wayfinder actor-based aliases) and `_include:iterate`; `_id` vs `/Appointment/{id}` narrative added.

This review lists what is **still outstanding** to fully reconcile the two specs, grouped by priority.

---

## Status tracker (updated)

| Item | Description | Status |
|---|---|---|
| **A1** | Replace `OAuth_Token` with dual `NHSLoginPatientAccess` + `NHSApplicationAccess` schemes + top-level security | ✅ **DONE** |
| **A2** | Add `401` / `403` / `400` auth responses to all operations | ✅ **DONE** (+ TODO note added to `4XX-BARS`) |
| **A3** | Scope `/metadata` CapabilityStatement to this API (Appointment-only, `searchInclude` aligned) | ✅ **DONE** |
| **A4** | Create status code | ✅ **DONE** — decided to **keep `200` (no `201`)**, documented (no `Location` header, in line with BaRS) |
| **B** | Fold Wayfinder Appointment richness into schema (modes, `_patientInstruction`+Media, specialty, NOPAT, status enum) + add Location/PractitionerRole/Practitioner | ✅ **DONE** |
| **B1** | Extend `SearchBundleAppt` to allow actor `include` entries (`anyOf` + `search.mode`) | ✅ **DONE** |
| **A2+** | `4XX-BARS` table restructured (`Source`/`Notes` cols) with DI 400/401/403 rows + duplicate-code notes | ✅ **DONE** |
| **C** | Domain exclusion rules (under-16, PDS, Trust, England-only), empty `total:0` bundle, `429`, `504` | ✅ **APPLIED** (pending decisions) — exclusion-rules narrative added to `GET /Appointment`; under-16 `403` + empty-`200` + `429` in 4XX table; `504` added to 5XX table. **Open:** decision #7 (under-16 `403` vs silent empty-`200`) and the `504`-vs-`408` timeout overlap — both flagged with REVIEW notes in the spec |
| **D** | Strict `patient:identifier` NHS-number `pattern`; confirm required-for-search | ⏳ **OUTSTANDING** |
| **E** | Polish: example URLs → DI base `int.api.service.nhs.uk/FHIR/R4`; `openapi` bumped to `3.0.3`; "booking" → "appointment" wording | ✅ **DONE** |

**Done:** A1, A2 (+ 4XX table restructure), A3, A4, B, B1, C (applied), E. **Left:** D. **Open decisions only (flagged in the spec):** under-16 `403` vs silent empty-`200`; `504`-vs-`408` timeout.

The detail for each item is below; completed items are marked ✅ and retained for the record.

---

## A. High priority — contract correctness gaps

### A1. Security scheme doesn't match the overview (top of list) — ✅ DONE
- **Now:** top-level `security` is `[{ "OAuth_Token": [] }]` and the only `securitySchemes` entry is `OAuth_Token` (inherited from the BaRS refactor).
- **But:** the merged `info.description` describes the **dual model** — `NHSLoginPatientAccess` (patient/token-exchange) and `NHSApplicationAccess` (application/client-credentials) — per the §4 decision.
- **Gap:** the machine-readable security contradicts the prose. Callers/tools will see one generic bearer scheme, not the two intended.
- **Action:** replace `OAuth_Token` with the two schemes from `Old/direct-integration-api-prototype.json` (`NHSLoginPatientAccess`, `NHSApplicationAccess`) and set top-level `security` to list both as alternatives. (This is the §4 decision; it just hasn't been applied to the schemes/`security` block yet.)

### A2. No explicit auth error responses (401 / 403 / 400) — ✅ DONE
- **Now:** every operation lists only `200 / 4XX / 5XX` (BaRS `4XX-BARS` / `5XX-BARS`).
- **Overview promises:** `401` (+`WWW-Authenticate: Bearer`), `403` (insufficient permissions / patient assurance), and `400` (missing/malformed `NHSD-End-User-Organisation` for application auth).
- **Gap:** those responses exist in `Old/` (`Unauthenticated`, `Forbidden`, `InvalidOrganisationHeader`) but are **not** wired into the current operations.
- **Action:** add `401`, `403`, and (where relevant) `400` responses to each operation, referencing the three response components brought over from `Old/`.

### A3. `/metadata` CapabilityStatement still advertises the full BaRS server — ✅ DONE
- **Now:** the `/metadata` 200 example still describes `$process-message`, `MessageDefinition`, `ServiceRequest`, `DocumentReference`, `Slot` — i.e. the whole BaRS server, not this API.
- **Gap:** this API only exposes `/metadata` + `Appointment` (read/search/create/update/patch/delete). The CapabilityStatement should advertise **only** those interactions, and its `searchInclude` should list the Appointment `_include` values we now support.
- **Action:** trim the `/metadata` example to an Appointment-only CapabilityStatement (id/name/url/status=draft/experimental=true for this API), with `Appointment` resource interactions `read, search-type, create, update, patch, delete` and `searchInclude` = the agreed `_include` set. (The `Old/` prototype already had an Appointment-only CapabilityStatement to reuse.)

### A4. Success status codes are all `200` — no `201` for create — ✅ DONE (kept `200` by decision)
- **Decision taken:** keep `POST /Appointment` at **`200`** — **`201` is deliberately not used** because no `Location` header is returned. This is in line with BaRS: the created resource (with its server-assigned `id`) is returned in the body and the client reads the `id` from there, not from `Location`.
- **Applied:** the create operation description and its `200` response now explain this explicitly (and the stray "Update a single Appointment resource" description was corrected to a proper create description).
- **Note:** `PUT`/`PATCH` remain `200`; `DELETE` returns `200` + OperationOutcome (unchanged, consistent with BaRS).

---

## B. High/medium — data model — ✅ DONE (Wayfinder richness folded in)

The current JSON `Appointment` schema is the **lean BaRS** one. None of the Wayfinder patient-facing features are present (confirmed absent in the schema *and* anywhere in the file):

| Feature (from YAML) | In JSON now? | Action |
|---|---|---|
| `status` enum `booked \| fulfilled \| cancelled \| noshow` | ❌ (free string, example only) | Add enum; reconcile with UKCore/BaRS status |
| Delivery-channel extension (UKCore) | ❌ | Add |
| **Reschedule-mode** extension | ❌ | Add |
| **Cancellation-mode** extension | ❌ | Add |
| `_patientInstruction` (R5 cross-version) + contained `Media` | ❌ | Add |
| `specialty` (UKCore PracticeSettingCode) | ❌ | Add |
| `NOPAT` security label on `meta.security` | ❌ | Add |
| Rich `Location` / `PractitionerRole` / `Practitioner` schemas | ❌ (minimal) | Add for `_include`/actor resolution |

- **Action:** merge the Wayfinder Appointment schema fields/extensions into the JSON `Appointment` (mark them optional so the aggregator flow is unaffected when absent), and add the richer Location/PractitionerRole/Practitioner schemas so the actor-based `_include` values actually have target schemas.

### B1. Response Bundle schema vs actor-based includes — ✅ DONE
- The JSON search response uses `SearchBundleAppt`. Now that we accept `Appointment:actor` / `Appointment:location` and `_include:iterate`, the Bundle `entry` needs to allow **Location / PractitionerRole / Practitioner** include entries (the YAML `Bundle` used an `anyOf` for this).
- **Action:** extend `SearchBundleAppt` (or add a merged Bundle schema) to permit those `include`-mode entries.

---

## C. Medium — domain rules & error catalogue — ✅ APPLIED (2 decisions still open: under-16 403-vs-200, and 504-vs-408 timeout)

**Progress:** the `4XX-BARS` error table has been **restructured** (added `Source` + `Notes` columns) and now documents the DI-specific `400`/`401`/`403` responses with **duplicate-code notes** stating which definition is authoritative. The **under-16 `403 UNDER_16_DENIED`** row and the **`200` `total: 0` empty-searchset** row have been folded into that table, each carrying a **REVIEW REQUIRED** note tied to decision #7. Still outstanding: the exclusion rules as narrative prose, and `504`.

| Rule / case (YAML) | In JSON now? | Action / status |
|---|---|---|
| Under-16 exclusion (`403 UNDER_16_DENIED`) | ✅ In `4XX-BARS` table with REVIEW note | **Decision #7 still open** — keep distinct 403 vs silent empty `200` |
| Empty happy path → `200` Bundle `total: 0` | ✅ In `4XX-BARS` table with note | Documented as the response for genuine no-match **and** withheld/excluded records (no disclosure) |
| `429` Too Many Requests | ✅ Present in `4XX-BARS` table | Represented (BaRS SEND_/REC_ rows) |
| Excluded patients — PDS sensitive/restricted | ⏳ Noted (empty-200 behaviour) | **Still to add** as an explicit inclusion/exclusion prose rule |
| Excluded patients — Trust flag | ⏳ Noted (empty-200 behaviour) | **Still to add** as an explicit prose rule |
| NHS-in-England-only | ⏳ partial (word in prose) | **Still to confirm** as an explicit prose rule |
| `504` Gateway Timeout | ❌ | **Still to add** to `5XX` handling |

- **Remaining action:** add an "Appointment inclusion/exclusion rules" section to the description (England-only, Trust flag, PDS sensitive/restricted → all surface as empty `200 total:0`), add `504` to the 5XX catalogue, and resolve decision #7 (under-16: distinct 403 vs silent empty result).

---

## D. Medium — parameter reconciliation — ⏳ OUTSTANDING (partly done)

| Item | State | Action |
|---|---|---|
| `patient:identifier` strictness | JSON: optional, loose example | Adopt YAML's strict NHS-number `pattern` (`^https://fhir.nhs.uk/Id/nhs-number\|[1-9][0-9]{9}$`); confirm whether required for search |
| Demographic filters (`name`, `birthdate`, `postalcode`) | JSON only | Keep (optional) — no YAML equivalent |
| `_id` optionality | ✅ optional, narrative added | Done |
| `_include` vocab (chain + actor aliases) | ✅ both present | Done — but see A3 (advertise in `/metadata`) and B1 (Bundle entries) |
| `Accept` header | JSON: `application/fhir+json; version=1.0.0` | Confirm the version token is intended for this API |

---

## E. Low — consistency & polish — ✅ DONE

- **`operationId` / summaries** still say "Get bookings for a patient" / "booking" (BaRS wording). Fine, but consider aligning with "appointment" terminology used elsewhere.
- **Examples** in the JSON still use `sandbox.api.service.nhs.uk/booking-and-referral/...` URLs in `link.self` / `fullUrl`; update to the DI servers for consistency.
- **`openapi` version:** JSON is `3.0.0`, YAML is `3.0.3` — align (recommend `3.0.3`).
- **Tags:** JSON uses `Booking` / `Metadata`; fine.
- **NFRs** are in the description (from §2) but not attached to operations — acceptable; optionally add response-time/throughput notes per operation.

---

## F. Outstanding decisions (from the merge plan §13)

Resolved:
1. ~~Canonical format~~ ✅ JSON
2. ~~Auth token model~~ ✅ prototype token-exchange
6. ~~`status` value set~~ ✅ applied as `booked | fulfilled | cancelled | noshow` in B (still worth confirming against UKCore)

Still open (each is flagged with a REVIEW note in the spec where relevant):
3. **`/metadata` required?** — CapabilityStatement is now Appointment-only (A3); confirm whether implementing `/metadata` is mandatory for Receivers.
4. **Keep `_id` long-term** or deprecate after migration? — narrative added; lifecycle still to decide.
5. **`_include` vocab** — BaRS chain + Wayfinder actor-aliases both supported; decide whether the aliases are permanent or transitional.
7. **Under-16** — `403 UNDER_16_DENIED` vs silent empty `200 total:0`. Applied both the 403 row and the empty-200 behaviour with REVIEW notes; **the choice between them is still open** and is the one inconsistency in the exclusion model.
8. **NFRs apply to write ops?** — figures (400ms p95 / 60 TPS) were written for a read endpoint.
9. **`504` vs `408` timeout** *(new)* — `504` (gateway) added to 5XX; BaRS also has `408 REC_TIMEOUT`. Pick one for a downstream timeout rather than defining both.

---

## Suggested order of work — status

1. **A1 + A2** — security schemes + 401/403/400 responses. ✅ **DONE**
2. **A3** — Appointment-only `/metadata` CapabilityStatement incl. `searchInclude`. ✅ **DONE**
3. **A4** — create status code. ✅ **DONE** (kept `200` by decision; delete confirmed as `200`+OperationOutcome).
4. **B / B1** — Wayfinder Appointment richness + Bundle include entries. ✅ **DONE**
5. **C** — exclusion rules + error catalogue. ✅ **DONE** (2 decisions open: #7 under-16, #9 504-vs-408).
6. **E** — example URLs, `openapi` version, wording. ✅ **DONE**
7. **D** — strict `patient:identifier` pattern. ⏳ **REMAINING** (no open decision; can be applied any time).

**Net:** everything in the review is applied except **D**. What's left is otherwise a set of **decisions** (F.3, F.4, F.5, F.7, F.8, F.9), each already surfaced in the spec with a REVIEW note.

---

## Quick status snapshot

| Area | State |
|---|---|
| info / servers / version / overview | ✅ Done |
| Operation surface (metadata + Appointment CRUD) | ✅ Present |
| `_id` / `_include` / `_include:iterate` params | ✅ Present (optional) |
| Security **schemes** match dual-auth prose | ✅ Done — dual schemes (A1) |
| Auth error responses 401/403/400 | ✅ Done (A2) — `4XX-BARS` carries a review TODO |
| `/metadata` scoped to this API | ✅ Done — Appointment-only (A3) |
| Create status code | ✅ Done — `200` by decision, no `201`/`Location`, in line with BaRS (A4) |
| Wayfinder Appointment richness (modes/instruction/specialty/NOPAT/status enum) | ✅ Done (B) |
| Bundle allows actor include entries | ✅ Done (B1) |
| Auth error table (4XX) documents 400/401/403 + duplicates | ✅ Done (A2+) |
| Under-16 `403` + empty-`200` in 4XX table (with REVIEW notes) | ✅ Done (C) |
| Exclusion-rules narrative (England/Trust/PDS → empty-200) on `GET /Appointment` | ✅ Done (C) |
| `504` in 5XX table (with 408-overlap review note) | ✅ Done (C) |
| Open decisions in C: under-16 (403 vs 200); 504-vs-408 timeout | ⏳ Decisions only (flagged in spec) |
| Strict `patient:identifier` pattern | ❌ Outstanding (D) |
| Polish (example URLs, `openapi` version, wording) | ✅ Done (E) |
