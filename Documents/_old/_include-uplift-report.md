# BaRS Implementation Guide Uplift Report: `_include` Parameter Additions

## Summary

The new OAS file (`bars_api_OAS-with_include.json`) introduces `_include` support on the **GET /Appointment** endpoint and formalises the existing `_include` on **GET /Slot**. This report identifies where the [BaRS Implementation Guide (v1.12.0)](https://simplifier.net/guide/nhsbookingandreferralstandard/Home?version=1.12.0) would require uplift to reflect these changes.

---

## What Was Added in the OAS

### 1. GET /Appointment — NEW `_include` parameter (`AppointmentInclude_QParam`)

Allows senders to request that referenced resources are returned inline (with `search.mode: "include"`) rather than needing separate follow-up reads.

| `_include` Expression | Included Resource |
|---|---|
| `Appointment:slot` | Slot |
| `Appointment:based-on` | ServiceRequest |
| `Slot:schedule` | Schedule |
| `Schedule:actor:Practitioner` | Practitioner |
| `Schedule:actor:PractitionerRole` | PractitionerRole |
| `Schedule:actor:HealthcareService` | HealthcareService |
| `HealthcareService:providedBy` | Organization |
| `HealthcareService:location` | Location |

### 2. GET /Slot — Existing `_include` (formalised/unchanged)

Already documented in the guide but now explicitly enumerated in the OAS with an additional wildcard option (`Slot:*`).

| `_include` Expression | Included Resource |
|---|---|
| `Slot:schedule` | Schedule |
| `Schedule:actor:Practitioner` | Practitioner |
| `Schedule:actor:PractitionerRole` | PractitionerRole |
| `Schedule:actor:HealthcareService` | HealthcareService |
| `HealthcareService:providedBy` | Organization |
| `HealthcareService:location` | Location |
| `Slot:*` | All referenced from Slot |

### 3. CapabilityStatement Example

The CapabilityStatement example in the OAS now includes a `searchInclude` array on the **Appointment** resource type (server mode), advertising the supported inclusions. Receivers must reflect this in their own CapabilityStatements.

---

## Pages Requiring Uplift

### A. BaRS Core — End-to-end Workflow

**Page:** `Home/Core/{version}/End-to-end-workflow`

| Section | Change Required |
|---|---|
| **Search for Slots** | Review whether the `Slot:*` wildcard addition needs documenting alongside the existing named `_include` values. Currently the guide documents the six named expressions — confirm alignment with OAS enum. |
| **GET Appointment (retrieve booking)** | **New section or significant update required.** The guide currently describes GET /Appointment as returning a single Appointment or searchset of Appointments. It must now document the optional `_include` parameter, explain how senders can request related resources in a single call, and describe the response shape (entries with `search.mode: "match"` vs `search.mode: "include"`). |
| **Cancel/Update booking flow** | Update guidance to note that a sender performing a read-before-update can now optionally use `_include` to simultaneously resolve the Slot, Schedule, and HealthcareService context without separate calls. |

---

### B. BaRS Core — Standard Pattern (Booking)

**Page:** `Home/Core/{version}/Standard-Pattern/Booking`

| Section | Change Required |
|---|---|
| Sender workflow diagrams | Add a step/note showing that GET /Appointment now supports `_include` to resolve linked resources. |
| Receiver implementation guidance | Document that receivers **MUST** advertise supported `_include` values in their CapabilityStatement `searchInclude` array for the Appointment resource. If an `_include` cannot be honoured, the receiver omits it from `Bundle.link.url` in the response. |
| Response handling | Document how senders should process a searchset Bundle containing both `match` and `include` entries. |

---

### C. BaRS Core — CapabilityStatement

**Page:** `Home/Core/{version}/End-to-end-workflow` (CapabilityStatement section) or `Home/FHIR-Assets/CapabilityStatement`

| Section | Change Required |
|---|---|
| Server CapabilityStatement example | Add `searchInclude` to the Appointment resource type listing the supported expressions (mirroring what already exists for Slot). |
| Client CapabilityStatement | If receivers acting as clients also perform GET /Appointment with `_include`, document this in the client mode section. |
| Guidance text | Explain that `searchInclude` is the mechanism by which a receiver advertises which `_include` values it supports, and that unsupported values are silently ignored. |

---

### D. BaRS Applications — All Applications That Use Booking

Each Application that involves booking (retrieving/cancelling Appointments) will need at minimum a note pointing implementers to the Core `_include` guidance.

| Application | Reason for Uplift |
|---|---|
| **Application 1** (111 to ED) | Uses booking; senders may benefit from `_include` when retrieving bookings. |
| **Application 2** (111 to UTC) | Uses booking. |
| **Application 3** (Referral into UEC) | Uses booking. |
| **Application 4** (Referral into UEC for Validation) | Uses booking. |
| **Application 5** (Referrals into Pharmacy) | Uses booking. |
| **Application 6** (Referral into an Emergency Treatment Centre) | Uses booking. |
| **Application 9** (999 to CAS Validation) | Uses booking. |
| Any future applications using GET /Appointment | Will inherit this capability. |

For each, the Application-specific workflow page should either:
- Reference the Core `_include` guidance directly, or
- Include a short note in the "Booking" section stating that `_include` is available and linking to Core.

---

### E. FHIR Assets / Examples

**Page:** `Home/FHIR-Assets/`

| Section | Change Required |
|---|---|
| Searchset Bundle examples for Appointment | Add an example showing a response to `GET /Appointment?patient:identifier=...&_include=Appointment:slot&_include=Slot:schedule&_include=Schedule:actor:HealthcareService&_include=HealthcareService:location` — the OAS already contains this example which can be promoted to the guide. |
| Searchset Bundle examples for Slot | Confirm existing examples still match the OAS (particularly if `Slot:*` is new). |

---

### F. Error Handling

**Page:** `Home/Core/{version}/Error-Handling`

| Section | Change Required |
|---|---|
| Unsupported `_include` behaviour | Document expected behaviour: if a receiver cannot honour an `_include`, it **ignores** it and reflects the omission in `Bundle.link.url`. No error is returned. This aligns with the OAS description. |

---

## Summary of Effort

| Priority | Area | Type of Change |
|---|---|---|
| **High** | Core End-to-end Workflow — GET Appointment | New content (new capability) |
| **High** | Core Standard Pattern — Booking | New content + diagram update |
| **High** | Core CapabilityStatement | Example update + guidance text |
| **Medium** | FHIR Assets — Appointment searchset example | New example |
| **Medium** | Core Error Handling | Minor addition |
| **Low** | Core End-to-end Workflow — Search Slot | Review for `Slot:*` wildcard |
| **Low** | Application pages (all booking apps) | Cross-reference note |

---

## Recommendation

1. Draft the Core `_include` guidance as a single reusable section (covering both Slot and Appointment) that Applications can reference.
2. Update the CapabilityStatement example and receiver guidance first — this unblocks suppliers from advertising the new capability.
3. Add the response example from the OAS into the FHIR Assets section as a canonical reference.
4. Review each Application page and add a short callout pointing to the new Core section.
