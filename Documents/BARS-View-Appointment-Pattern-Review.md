# Review: BARS — View Appointment Pattern

**Document reviewed:** `BARS_+View+Appointment+Pattern.doc` (Confluence export)
**Use case:** Patient-facing retrieval of an existing appointment via the NHS App / Patient Care Aggregator, using the BaRS Appointment resource (`GET /Appointment/{id}`).

---

## 1. Overall Assessment

This is a clear, well-scoped page for a single interaction (read one appointment). It correctly anchors on the BaRS **Appointment Management Foundation** model — provider is authoritative, sender reads the latest version, read-before-write feeds the downstream update/cancel/reschedule flows. Scope inclusions/exclusions are crisp, and the sender/provider responsibility split is good.

The page aligns well with the repository's own Standard Pattern (`03-view-appointment.md`). The gaps are mostly about **precision and consistency with the wider BaRS contract** rather than direction: the `_include` semantics, the `/metadata` capability check, patient-facing authorisation, and error/response detail need tightening so the page is implementable without ambiguity.

---

## 2. Alignment With the BaRS Standard Pattern

The page matches the repo's `03-view-appointment.md` on the essentials:

| Aspect | View Appointment page | Standard Pattern (`03-view-appointment.md`) | Aligned? |
|---|---|---|---|
| Primary operation | `GET /Appointment/{id}` | `GET /Appointment/{id}` | ✅ |
| Provider is authoritative / latest version | Yes | Yes | ✅ |
| Read-before-write for downstream actions | Yes (§1.5, §1.6) | Yes (explicit rule) | ✅ |
| Service discovery before the read | Yes (§1.4.1) | Yes (target service resolution) | ✅ |
| FHIR R4 UK Core Appointment | Yes (§1.6) | Yes (`UKCore-Appointment`) | ✅ |

**The main thing missing versus the Standard Pattern:** the mandatory **`GET /metadata` (CapabilityStatement) check** before first use of a target service. The Standard Pattern makes this a prerequisite for *every* interaction — the sender must confirm the receiver supports the view operation (and, here, whether it supports `_include`) before relying on it. This page does not mention it. **Recommend adding it to §1.4.1 / §1.6.**

---

## 3. Key Observations

### 3.1 `_include` — semantics need to be precise (§1.6)
The page says: *"Provider accepts `_include` variable and returns a single response with multiple co-located resources."* Two issues:

1. **`_include` is a search parameter, not a read parameter.** In FHIR, `_include` applies to a **search** (`GET /Appointment?...`), which returns a `searchset` Bundle. A read-by-id (`GET /Appointment/{id}`) returns a single resource, not a Bundle, and does not carry `_include`. As written, the page implies `_include` on the read-by-id path, which is not standard FHIR.

   **Recommend:** state that where linked resources are needed in one response, the interaction is a **search** — e.g. `GET /Appointment?_id={id}&_include=...` or `GET /Appointment?patient:identifier=...&_include=...` — returning a searchset Bundle with `match` + `include` entries. Keep `GET /Appointment/{id}` for the plain single-resource read.

2. **Enumerate the supported `_include` values.** The page lists Location, Practitioner, Participant informally. Tie these to the actual OAS `_include` expressions so providers and senders share one contract. Based on the Appointment `_include` already defined in the BaRS OAS, the relevant chain is:
   - `Appointment:slot` → Slot
   - `Slot:schedule` → Schedule
   - `Schedule:actor:HealthcareService` → HealthcareService
   - `HealthcareService:location` → Location
   - `Schedule:actor:Practitioner` / `Schedule:actor:PractitionerRole`
   - `Appointment:based-on` → ServiceRequest (referral)

   Note: "Participant" is generally **inline** in the Appointment resource (`participant.actor`), not something resolved via `_include` — worth clarifying so implementers don't expect an included Patient/Practitioner where the reference is already embedded.

3. **Unsupported `_include` behaviour.** State the expected behaviour when a receiver cannot honour an `_include`: per BaRS/FHIR it should ignore the unsupported value and reflect the omission in `Bundle.link.url`, not error. This matters for patient-facing robustness.

### 3.2 CapabilityStatement / `/metadata` check missing (§1.4.1)
As above — add the one-time `GET /metadata` step so the sender confirms the receiver supports view (and `_include`) before use. This is a hard prerequisite in the Standard Pattern and underpins the "provider must expose the appointment through the BaRS API endpoint" requirement in §1.5.

### 3.3 Patient-facing authorisation is under-specified (§1.5, §1.9)
The page states the appointment "MUST be patient facing" and that "all patient facing filters MUST be applied at the receiver/supplier side" — good — and names the Patient Care Aggregator as the Sender. But it doesn't reference the **authentication/authorisation model** that makes this safe. For a patient-facing read via the NHS App this is the crux:

- The trust model shifts from organisation-to-organisation (B2B) to citizen-to-service (B2C).
- The patient must only be able to view **their own** appointment; the authorised identity (NHS login, typically P9/AAL3) must be bound to the requested appointment/patient before the receiver returns data.
- `_include` must not let a caller pull linked resources **outside** the authorisation scope of the original request.

**Recommend:** add an Authorisation subsection (or reference the Standard Pattern's `08-patient-facing-nhs-identity.md`) covering: how the patient identity is asserted, that the receiver must enforce patient-level access, and that `_include` stays within the original authorisation scope. This also connects to §1.9 (a new solutions assurance model) — the assurance model largely exists *because* of this B2C authorisation shift, so it's worth making the linkage explicit.

### 3.4 Read-by-id vs search-by-patient — which is the primary path? (§1.4.2, §1.6)
The page assumes a known Appointment `id` ("Retrieval using a known Appointment identifier"). For an NHS App / aggregator flow, the app often will **not** know the appointment id up front — it will search by patient (`GET /Appointment?patient:identifier=...`) to discover appointments, then optionally read one.

**Recommend:** clarify both entry points:
- **Discovery:** `GET /Appointment?patient:identifier=...` (searchset) — how the app finds the patient's appointments.
- **Read:** `GET /Appointment/{id}` — retrieve a specific one.

The Standard Pattern documents both; the page currently only foregrounds the read-by-id.

### 3.5 Response and error detail (§1.6, §1.8)
The page describes the happy path but not the response shape or errors. For an implementable pattern, add:

- **Success:** `200` returning a `UKCore-Appointment` (single) for read-by-id, or a `searchset` Bundle for the search form.
- **Errors:** at least `401` (unauthorised / token invalid), `403` (authorised user not permitted for this patient — important in the patient-facing model), `404` (appointment not found), `501` (receiver does not support the operation). The Standard Pattern lists 401/404/501; **403 should be added for the patient-facing case.**
- **Required headers:** `Authorization`, `X-Request-Id`, `X-Correlation-Id`, `Accept: application/fhir+json`, and — via the proxy — `NHSD-End-User-Organisation` / `NHSD-Target-Identifier`. The page's §1.4.1 service-discovery requirement implies these but doesn't list them.

### 3.6 Minimum Data Set reference (§1.5)
"MUST meet the minimum data requirements … as set out in the MVP definition" — good, but link/cite the actual MVP/MDS definition so it's testable. An acceptance criterion in §1.8 could reference the specific mandatory fields (e.g. status, start/end, location, appointment type) rather than "without loss of meaning or structure" alone.

### 3.7 Appointment type coding (§1.6)
The page gives "Outpatient (AMB), Community, etc." Pin these to the actual code system/value set (e.g. the FHIR/UK Core appointment or encounter class coding) so providers return consistent, machine-readable types rather than free text. Patient-facing display depends on this being coded.

---

## 4. Smaller Points

- **§1.6 "Provider accepts `_include` variable"** — "variable" → "parameter" (FHIR search parameter).
- **§1.1 numbering** — the page starts at "1.1 Use Case" then "Overview"; the heading levels are slightly inconsistent (cosmetic, from the export).
- **§1.9 References** — both reference links render as the generic "NHS Booking and Referral Standard"; point them at the specific Appointments Standard Pattern and Application 1 pages/anchors.
- **Information flow (§1.7)** — appears to be a diagram that didn't survive the text export; ensure the published page includes the sequence (discovery → read → response → consumer processing), ideally as a sequence diagram.
- **Clinical safety (§1.4.3)** — good that intermediaries must not alter clinical intent; consider stating that the proxy/aggregator is pass-through for the FHIR payload and performs no clinical transformation.

---

## 5. Suggested Priority Actions

| Priority | Action | Section |
|---|---|---|
| **High** | Correct `_include` semantics: it's a **search** parameter (searchset Bundle), not a read-by-id parameter; enumerate supported `_include` values against the OAS | §1.6 |
| **High** | Add the patient-facing **authorisation** model (patient can only view own appointment; `_include` stays in scope); link to `08-patient-facing-nhs-identity.md` | §1.5, §1.9 |
| **High** | Add the mandatory `GET /metadata` capability check before first use | §1.4.1 |
| **Medium** | Clarify both entry points — search-by-patient (discovery) vs read-by-id | §1.4.2, §1.6 |
| **Medium** | Add response shapes and error codes, including **403** for the patient-facing case | §1.6, §1.8 |
| **Low** | Pin appointment-type coding and the MDS/MVP reference to concrete value sets/definitions | §1.5, §1.6 |
| **Low** | Fix references, restore the §1.7 flow diagram, terminology (`parameter`) | §1.6, §1.7, §1.9 |

---

## 6. Summary

The pattern is directionally correct and matches the BaRS Appointment Management Foundation and the repository's `03-view-appointment.md`. The most important corrections are: (1) treat `_include` as a **search** capability with an enumerated, OAS-aligned value list and defined unsupported-value behaviour; (2) make the **patient-facing authorisation** model explicit, since this is a B2C read via the NHS App and the receiver must enforce patient-level access; and (3) add the `/metadata` capability check. Tightening the response/error detail and coding references will make the page directly implementable and testable.

---

*Prepared from the exported document text and cross-referenced against the BaRS Appointments Standard Pattern in `BaRS-Appointments-StandardPattern/` (`03-view-appointment.md`, `08-patient-facing-nhs-identity.md`, `README.md`) and the Appointment `_include` definition in the BaRS OAS.*
