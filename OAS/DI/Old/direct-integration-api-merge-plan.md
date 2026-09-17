# Direct Integration API — Merge Plan

**Purpose:** A working document to merge the two Direct Integration specs into a single target OpenAPI specification.

**Inputs**
- **Prototype** — `direct-integration-api-prototype.json` (BaRS-aligned, full-CRUD, dual patient/application auth, JSON)
- **Wayfinder standard** — `direct-integration-api.yaml` (provider-facing, single `GET /Appointment`, NHS-login ID-token auth, rich patient-facing payload, concrete SLAs, YAML)

**Target / working file:** the merge is being applied **in place** to **`direct-integration-api-prototype.json`** (the prototype JSON is the base being grown into the merged spec). The Wayfinder standard `direct-integration-api.yaml` is the source we fold in from and is left unchanged.

> How to use this doc: work top to bottom. Each section states the **decision**, the **rationale**, and the **action**. Open decisions are flagged **⚠️ DECISION NEEDED** — resolve these before applying. As each section is applied to `direct-integration-api-prototype.json` it is marked **✅ APPLIED**.

## Progress log

| Section | Status | Notes |
|---|---|---|
| §2 Metadata & document identity | ✅ **APPLIED** to `direct-integration-api-prototype.json` | Title, servers, version, status prose and merged Overview written; JSON validated |
| §3 Operations | Pending | |
| §4 Authentication & authorisation | ✅ **DECIDED** — prototype correct; no JSON change needed | Token-exchange model kept; `NHSD-ID-Token` dropped |
| §5 Parameters | Pending — has ⚠️ decisions | |
| §6 `_include` | Pending — has ⚠️ decision | |
| §7 Appointment model | Pending — has ⚠️ decision | |
| §8 Domain rules & exclusions | Pending — has ⚠️ decision | |
| §9 Headers | Pending | |
| §10 Errors | Pending | |
| §11 NFRs | Applied via Overview (§2); schema/operation-level TBC | |

---

## 1. Guiding principles for the merge

1. **BaRS is the reference.** Where the two disagree on shape/semantics, follow BaRS (the prototype already declares this). Wayfinder-specific additions are layered on top, not in place of, BaRS.
2. **One resource model, two audiences.** The merged Appointment must serve both the aggregator/consumer flow (prototype) and the provider-facing patient display flow (Wayfinder). That means the *union* of fields, with clear optionality.
3. **Keep the full operation surface, gate by capability.** Retain CRUD from the prototype; receivers advertise what they actually support via `/metadata`. A read-only provider is a conformant subset.
4. **Prefer explicit contracts.** Enumerated values, required/optional stated per auth mode, documented error semantics.
5. **No silent data loss.** Every field, extension, exclusion rule and NFR from both specs must be either carried forward or explicitly dropped with a reason (see §12 traceability).

---

## 2. Metadata & document identity

**✅ DECIDED & APPLIED to `direct-integration-api-prototype.json` (JSON validated). Actions recorded below.**

| Item | Prototype | Wayfinder | **Merged decision (DECIDED)** |
|---|---|---|---|
| Title | Direct Integration API (Prototype) | Wayfinder Direct Integration API Specification | ✅ **Use the Wayfinder title: "Wayfinder Direct Integration API Specification"** |
| Format | JSON | YAML | ✅ **JSON for now** (canonical). A YAML rendering can be generated later if needed |
| Version | 1.0.0 | 1.0.0 | ✅ **1.0.0** |
| Status | draft / experimental | beta | ✅ **As per the JSON (prototype): `status: draft`, `experimental: true`** on the CapabilityStatement |
| Servers | placeholder | INT + Prod | ✅ **Use the Wayfinder servers** — `https://int.api.service.nhs.uk` (INT) and `https://api.service.nhs.uk` (Prod) |
| Overview prose | BaRS-alignment + security | Wayfinder usage + PCA + NFRs | ✅ **Merge both** — keep the prototype's Overview + full Security/authentication prose, and fold in Wayfinder's "Who can use", PCA Record Service link, Status, Technology, Network access and NFR content |

**Actions taken (this section):**
- `info.title` → **"Wayfinder Direct Integration API Specification"**.
- Canonical artefact is **JSON** (`direct-integration-api.json`).
- `info.version` → **"1.0.0"**.
- CapabilityStatement carries **`status: "draft"`, `experimental: true`** (from the prototype JSON); document `## Status` prose to match (i.e. present as draft/experimental, not beta).
- `servers` block → **Wayfinder INT + Prod URLs** (replacing the prototype placeholder).
- `info.description` → **merged Overview**: prototype's overview + Security/authentication sections retained verbatim, with Wayfinder's "Who can use this API standard", PCA Record Service API link, Status, Technology, Network access and Non-functional (see §11) content added.

> Note: two internal tensions to keep consistent elsewhere in the merged spec —
> (1) **Status wording:** we're taking `draft/experimental` from the JSON, so the Overview `## Status` text should say draft/experimental rather than Wayfinder's "beta". Update §2 target status accordingly (superseded the earlier "beta" recommendation).
> (2) **Title vs server host:** the title is now "Wayfinder…" while servers are the shared `*.api.service.nhs.uk` hosts — expected, no action.

---

## 3. Operations (paths)

The merged spec keeps the **prototype's full surface**; the Wayfinder read is a subset of it.

| Operation | Prototype | Wayfinder | **Merged** |
|---|---|---|---|
| `GET /metadata` | ✅ | ❌ (explicitly unsupported) | ✅ **Include.** ⚠️ **DECISION NEEDED** — Wayfinder providers currently don't implement `/metadata`. Either (a) require it in the merged standard, or (b) mark it optional and let capability be assumed for read-only providers. Recommend **(a) require it** so senders can discover supported interactions/`_include`. |
| `GET /Appointment` (search) | ✅ | ✅ | ✅ Merge — see §4, §5 |
| `POST /Appointment` (create) | ✅ | ❌ | ✅ Keep; optional per receiver capability |
| `GET /Appointment/{id}` | ✅ | ❌ (uses `_id` query instead) | ✅ Keep path form; **also accept `_id` search** for Wayfinder compatibility — see §5 |
| `PUT /Appointment/{id}` | ✅ | ❌ | ✅ Keep; optional per capability |
| `PATCH /Appointment/{id}` | ✅ | ❌ | ✅ Keep; optional per capability |
| `DELETE /Appointment/{id}` | ✅ | ❌ | ✅ Keep; optional per capability |

**Action:** base the paths block on the prototype. Add read-only conformance note: a provider MAY implement only `GET /Appointment` (+`/metadata`) and remain conformant.

---

## 4. Authentication & authorisation

**✅ DECIDED — the prototype is correct. Already present in `direct-integration-api-prototype.json`; no JSON change required.**

Decision: adopt the **prototype's authentication & authorisation model in full**. The prototype's token-exchange approach is the target; the Wayfinder `NHSD-ID-Token` raw-ID-token model is **not** carried forward.

| Aspect | Prototype | Wayfinder | **Merged decision (DECIDED)** |
|---|---|---|---|
| Patient auth | NHS login → **token exchange** → API Platform access token as bearer | NHS login **ID token in `NHSD-ID-Token`** header, verified by provider | ✅ **Prototype's token-exchange model** — bearer = the exchanged API Platform access token. `NHSD-ID-Token` is **dropped** |
| Application auth | client-credentials + signed JWT assertion | (client-credentials scheme present, thin) | ✅ **Keep prototype's application-based mode** in full |
| P9 verification | Required (patient mode) | Required (via ID token) | ✅ **Retain P9 requirement** (patient mode) |
| Security schemes | `NHSLoginPatientAccess`, `NHSApplicationAccess` | `oAuth2ClientCredentials` | ✅ **Prototype's two schemes**; the thin Wayfinder scheme is **not** carried forward |
| Org header | conditional (app: MUST, patient: SHOULD NOT) | not used | ✅ **Keep prototype's conditional rule** |
| Failure semantics | 400/401/403 + `WWW-Authenticate` | 401/403 (403=under-16) | ✅ **Keep prototype's**; Wayfinder's domain 403 (under-16) handled as an exclusion/authorisation rule — see §8 |

**Actions taken (this section):**
- Confirmed `direct-integration-api-prototype.json` already carries the full prototype Security & authentication prose, the two bearer security schemes (`NHSLoginPatientAccess` token-exchange, `NHSApplicationAccess` client-credentials), the conditional `NHSD-End-User-Organisation` rule, and the `Unauthenticated` (401 + `WWW-Authenticate`), `Forbidden` (403) and `InvalidOrganisationHeader` (400) responses.
- Verified the JSON contains **no** `NHSD-ID-Token` header and **no** `oAuth2ClientCredentials` scheme — nothing to remove.
- **No JSON edit required for §4.** The prototype is already the agreed target.

> Note: because the Wayfinder ID-token model is being dropped rather than migrated, there is no "provider migration note" to add. If providers currently send `NHSD-ID-Token`, that is a client-side change tracked outside this spec.

---

## 5. Query & path parameters

| Parameter | Prototype | Wayfinder | **Merged decision** |
|---|---|---|---|
| `patient:identifier` | optional, patient search | **required**, strict NHS-number regex | **Required for search**; adopt Wayfinder's strict NHS-number `pattern` |
| `patient:name` / `:birthdate` / `:address-postalcode` | ✅ demographic filters | ❌ | **Keep** as optional filters (within authorised patient context) |
| Single read | path `/Appointment/{id}` | **`_id` query (required)** | **Support both**: canonical `GET /Appointment/{id}`; also accept `GET /Appointment?_id=` for Wayfinder compatibility. ⚠️ **DECISION NEEDED** — confirm whether to keep `_id` long-term or deprecate after migration |
| `_include` | array, `explode:true`, BaRS chain | single string, `required`, actor-based | See §6 |
| `_include:iterate` | `PractitionerRole:practitioner` | `PractitionerRole:practitioner` (`required`) | Align — see §6; make **optional** |

**Action:** merged `_include`/`_include:iterate` should be **optional arrays** (prototype's requiredness wins — requiring includes is unusual and forces every caller to fan out).

---

## 6. `_include` reconciliation

The two use different reference vocabularies. Merge to the **superset**, expressed as the BaRS chain, and map Wayfinder's actor-based shorthands onto it.

| Wayfinder value | Meaning | BaRS-chain equivalent (merged) |
|---|---|---|
| `Appointment:location` | Location off the appointment | Covered via `Appointment` participant/actor → `HealthcareService:location` in the chain, **or** keep `Appointment:location` as an accepted alias |
| `Appointment:actor` | Location/Practitioner/PractitionerRole participants | Map to `Schedule:actor:Practitioner` / `:PractitionerRole` / `:HealthcareService` + `HealthcareService:location` |
| `PractitionerRole:practitioner` (iterate) | Practitioner from included role | **Same** — keep `_include:iterate=PractitionerRole:practitioner` |

**Merged supported `_include` set (superset):**
`Appointment:slot`, `Appointment:based-on`, `Slot:schedule`, `Schedule:actor:Practitioner`, `Schedule:actor:PractitionerRole`, `Schedule:actor:HealthcareService`, `HealthcareService:providedBy`, `HealthcareService:location` — plus accepted Wayfinder aliases `Appointment:location`, `Appointment:actor`.

⚠️ **DECISION NEEDED** — do we (a) standardise on the BaRS chain and deprecate the `Appointment:actor`/`Appointment:location` aliases, or (b) support both permanently? Recommend **(a)** with a transition period, so there is one include model long-term.

**Behaviour to standardise (from prototype):** unsupported `_include` values are ignored and the omission is reflected in `Bundle.link.url`; `/metadata` `searchInclude` advertises what the receiver supports.

---

## 7. Appointment resource model (the union)

Base = prototype Appointment; **add the Wayfinder patient-facing fields** (these are the richer, MVP/patient-display elements and must not be lost):

Carry forward from Wayfinder into the merged Appointment schema:
- **UK Core extensions:** delivery channel, **reschedule-mode**, **cancellation-mode**
- **`_patientInstruction`** (R5 cross-version extension) referencing a **contained `Media`** resource (prep guidance)
- **`specialty`** (UK Core PracticeSettingCode)
- **`NOPAT`** security label support on `meta.security`
- **`status` enum:** `booked | fulfilled | cancelled | noshow` (reconcile with any BaRS status values — ⚠️ **DECISION NEEDED**: confirm the canonical status value set; BaRS/UKCore Appointment status vs this patient-facing set)
- Richer **`Location` / `PractitionerRole` / `Practitioner`** schemas (telecom, address, position, managingOrganization ODS, role coding, names)

Keep from prototype: `slot`, `basedOn` (ServiceRequest/referral link), `participant.actor` references, `UKCore-Appointment` profile.

**Action:** produce one `Appointment` schema = prototype core ∪ Wayfinder extensions. Mark patient-display fields optional so the aggregator/application flow is unaffected when absent.

---

## 8. Domain rules & exclusions (Wayfinder-only — must be carried forward)

The prototype has none of these; they are essential provider behaviour and belong in the merged standard:

- **NHS in England only** — exclude out-of-England care settings.
- **Excluded patients (Trust)** — appointments flagged not for NHS App access.
- **Excluded patients (PDS)** — sensitive/restricted filtered out.
- **Under-16 exclusion** — Wayfinder returns **403 `UNDER_16_DENIED`**. ⚠️ **DECISION NEEDED**: is under-16 a 403 (Wayfinder) or should it be an empty result? Align with the chosen patient-authorisation model.
- **Empty happy path** — no/excluded appointments → **200 with `Bundle.total: 0`** (adopt this).

**Action:** add an "Appointment inclusion/exclusion rules" section (from Wayfinder) to the merged spec, reconciled with the prototype's patient-authorisation policy in §4.

---

## 9. Headers

| Header | Prototype | Wayfinder | **Merged** |
|---|---|---|---|
| `X-Request-Id` | required | defined, commented out | **Required** (BaRS standard) |
| `X-Correlation-Id` | required | required | **Required** |
| `NHSD-Target-Identifier` | ✅ | ❌ | **Keep** |
| `NHSD-End-User-Organisation` | conditional | ❌ | **Keep** (conditional per auth mode) |
| `NHSD-Requesting-Software` | ✅ | ❌ | **Keep** |
| `NHSD-Requesting-Practitioner` | optional | ❌ | **Keep** (optional) |
| `NHSD-ID-Token` | ❌ | required | **Drop** if token-exchange chosen in §4; otherwise retain. Tied to the §4 ⚠️ |
| `Accept` | required | — | **Keep** |

---

## 10. Errors

Adopt the **prototype's `OperationOutcome` structure and coded errors** (SEND_/REC_/PROXY_, plus `Unauthenticated` 401, `Forbidden` 403, `InvalidOrganisationHeader` 400), and **fold in Wayfinder's domain-specific cases**:

- 400 missing/invalid `_id` (if `_id` retained)
- 403 under-16 (per §8 decision)
- 404 not found
- **429** rate limit (add to merged)
- **504** gateway timeout (add to merged)

⚠️ Reconcile the two `OperationOutcome` schema names (`OperationalOutcome` in prototype vs `OperationOutcome` in Wayfinder) — **use `OperationOutcome`** (correct FHIR spelling).

---

## 11. Non-functional requirements (Wayfinder-only — carry forward)

The prototype has none; adopt Wayfinder's:
- **Gold service** (24/7/365)
- **≤ 400ms @ p95** response time
- **60 TPS** throughput
- **Diagnostic logging** with `X-Correlation-ID`, PII excluded in production, **90-day** retention
- Add the prototype's note that placeholder/deployment config must be completed before go-live.

⚠️ **DECISION NEEDED** — confirm these NFR figures still apply to the merged, CRUD-capable API (they were written for a single read endpoint; write operations may need their own targets).

---

## 12. Field-level traceability checklist

Use this to confirm nothing is lost during merge. Tick each once represented in the merged spec.

**From Prototype**
- [ ] `/metadata` + CapabilityStatement (searchInclude advertising)
- [ ] `GET/POST /Appointment`, `GET/PUT/PATCH/DELETE /Appointment/{id}`
- [ ] Dual auth (patient token-exchange + application client-credentials), P9
- [ ] Org-header conditional rule; requesting software/practitioner headers
- [ ] BaRS `_include` chain + `_include:iterate`; unsupported-include behaviour
- [ ] Coded OperationOutcome errors; 400/401/403 auth semantics + `WWW-Authenticate`
- [ ] `basedOn` referral link; target-identifier header

**From Wayfinder**
- [ ] Real INT/Prod servers; "who can use"; PCA record-service link
- [ ] Strict `patient:identifier` NHS-number pattern; `_id` search support
- [ ] Delivery-channel / reschedule-mode / cancellation-mode extensions
- [ ] `_patientInstruction` + contained `Media`
- [ ] `specialty` (PracticeSettingCode); `NOPAT` security label
- [ ] `status` enum (booked/fulfilled/cancelled/noshow) — reconciled
- [ ] Richer Location/PractitionerRole/Practitioner schemas
- [ ] Exclusion rules (England-only, Trust/PDS flags, under-16); empty `total:0` happy path
- [ ] 429 + 504 errors
- [ ] NFRs (gold, 400ms p95, 60 TPS, logging/retention)

---

## 13. Open decisions to resolve before building (summary)

1. ~~**Canonical format** — JSON vs YAML~~ ✅ **DECIDED: JSON for now** (see §2).
2. ~~**Auth token model** — token-exchange (prototype) vs `NHSD-ID-Token` (Wayfinder)~~ ✅ **DECIDED: prototype token-exchange model; `NHSD-ID-Token` dropped** (see §4).
3. **`/metadata`** — require it in the merged standard? (recommend yes).
4. **Single-read** — keep `_id` query alongside `/{id}` path, and for how long?
5. **`_include` vocabulary** — standardise on BaRS chain and deprecate `Appointment:actor`/`:location` aliases? (recommend yes, with transition).
6. **`status` value set** — reconcile Wayfinder's patient-facing enum with BaRS/UKCore.
7. **Under-16** — 403 vs empty result.
8. **NFRs** — do the read-only figures apply to write operations?

---

## 14. Suggested build sequence

1. Resolve the eight open decisions in §13.
2. Start from the **prototype JSON** as the skeleton (it already has the operation surface and auth).
3. Merge the **Wayfinder Appointment schema** fields/extensions into the prototype's `Appointment` (§7) and add the Location/PractitionerRole/Practitioner richness.
4. Add the **exclusion rules, empty-bundle behaviour, 429/504 errors and NFRs** (§8, §10, §11).
5. Swap in **real servers** and reconcile the **`_include`** vocabulary (§3, §6).
6. Walk the **§12 checklist** to confirm no loss.
7. Validate the merged spec (lint/parse) and produce examples covering: search (no include), search (with include chain), single read, create, update/patch, cancel/delete, empty bundle, and each error.

---

*This plan does not modify the two source specs. It defines the target merged specification (`direct-integration-api.json`) and the decisions required to produce it.*
