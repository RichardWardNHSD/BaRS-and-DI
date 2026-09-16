# BaRS: impact of adding _include to GET /Appointment

Review date: 16 September 2026. Status: impact assessment; no repository or deployed API changes made.

## Assessment

The change lets a sender retrieve appointments and their linked scheduling or referral resources in one search response. It affects the API contract, Core appointment guidance, capability negotiation, receiver retrieval, sender response handling, examples and assurance.

The supplied [uplift report](https://github.com/RichardWardNHSD/BaRS-and-Wayfinder/blob/main/Documents/_include-uplift-report.md) establishes the scope: Appointment → Slot → Schedule → service/clinician → organisation/location, plus Appointment → ServiceRequest. This supersedes the earlier provisional suggestion of patient and direct participant includes.

The proposal is a useful starting point, but needs corrections before publication. Comparing the repository’s [baseline OAS](https://github.com/RichardWardNHSD/BaRS-and-Wayfinder/blob/main/BaRS%20-%20OAS/bars%20api%20OAS.json) with its [proposed OAS](https://github.com/RichardWardNHSD/BaRS-and-Wayfinder/blob/main/BaRS%20-%20OAS/bars_api_OAS-with_include.json) shows only the Appointment query parameter and JSON response examples have changed. Appointment capability declarations and the response schema still need work. Multi-step inclusion syntax and the organisation search-parameter name also need correction.

Optional request parameters and unchanged behaviour when absent can make this an additive capability. Mandatory supplier adoption or returning extra resource types by default would have wider compatibility consequences.

## Baseline

| Source | Reviewed version | Implication |
|---|---|---|
| Repository OAS pair | Both declare 1.4.0-alpha | Rebase onto the chosen supported API version |
| Implementation guide | 1.12.0; Core 1.5.0; package 1.41.0 | Target for the guide impact assessment |
| Published API catalogue | 1.0.7, 1.4.1 and 1.5.0 | Appointment _include absent from reviewed public contracts |
| FHIR | R4 4.0.1 | Governs inclusion and Bundle semantics |

Sources: [guide home](https://simplifier.net/guide/nhsbookingandreferralstandard/Home?version=1.12.0), [API catalogue](https://digital.nhs.uk/developer/api-catalogue/booking-and-referral-fhir), [published v1.5.0 OAS](https://digital.nhs.uk/restapi/oas/621537), repository files above.

The catalogue’s production labels and the alpha/development wording inside newer OAS files remain inconsistent. Confirm the release target and deployed capabilities with the BaRS owner; do not overwrite a newer specification with the repository’s older baseline.

## Intended scope and corrected request syntax

Inclusion uses reference search-parameter names, not resource-element paths. Direct includes start from matched Appointments; traversal from included resources needs the iterate modifier. Repeat parameters for multiple values. [FHIR R4 search](https://hl7.org/fhir/R4/search.html#inclusion).

| Information | Recommended request parameter | Relationship |
|---|---|---|
| Slot | _include=Appointment:slot | Direct from Appointment.slot |
| Referral | _include=Appointment:based-on | Direct from Appointment.basedOn |
| Schedule | _include:iterate=Slot:schedule | From included Slot |
| Practitioner | _include:iterate=Schedule:actor:Practitioner | From included Schedule |
| PractitionerRole | _include:iterate=Schedule:actor:PractitionerRole | From included Schedule |
| HealthcareService | _include:iterate=Schedule:actor:HealthcareService | From included Schedule |
| Organisation | _include:iterate=HealthcareService:organization | Maps to HealthcareService.providedBy |
| Location | _include:iterate=HealthcareService:location | From included HealthcareService |

The standard R4 HealthcareService search parameter is organization; providedBy is the element name. Retaining providedBy as a custom search parameter would require a published definition and explicit support. [HealthcareService search parameters](https://hl7.org/fhir/R4/healthcareservice.html#search). The two direct Appointment expressions are standard R4 reference search parameters. [Appointment search parameters](https://hl7.org/fhir/R4/appointment.html#search).

The downstream branches require their preceding inclusions. Including PractitionerRole does not automatically include its Practitioner. Patient, direct Appointment participants, reverse includes and Appointment wildcards are outside the supplied proposal.

Recommended corrected example, split over lines for readability:

~~~http
GET [base]/Appointment?patient:identifier=https%3A%2F%2Ffhir.nhs.uk%2FId%2Fnhs-number%7C4857773456
  &_include=Appointment:slot
  &_include:iterate=Slot:schedule
  &_include:iterate=Schedule:actor:HealthcareService
  &_include:iterate=HealthcareService:organization
  &_include:iterate=HealthcareService:location
~~~

This is a proposed contract, not evidence that deployed receivers support it.

## Corrections to the supplied proposal

| ID | Priority | Verified issue | Required action |
|---|---|---|---|
| F01 | High | Ordinary repeated _include values are described as recursively traversing the resource graph. | Define _include:iterate; correct parameter enums, prose, examples and receiver/proxy behaviour together. |
| F02 | High | HealthcareService:providedBy is used instead of standard R4 organization. | Correct it or formally define a custom alternative. Plan compatibility with existing Slot implementations separately. |
| F03 | High | The report says Appointment searchInclude was added. Neither repository OAS contains it for Appointment in server or client metadata examples. | Add server declarations and receiver guidance. Describe iterative support and limits explicitly. Include client-mode content where relevant. |
| F04 | High | SearchBundleAppt is unchanged and describes an Appointment-shaped resource object. | Model the mixed-resource response accurately. The permissive schema may accept additional types, but does not provide accurate documentation or generated typed models. |
| F05 | Medium | The new example retains a participant urn:uuid reference with no corresponding Bundle entry. | Use a resolvable reference or another valid representation. A non-requested ServiceRequest need not be included; that is a different issue. |
| F06 | Medium | GET /Slot, including Slot:*, is identical in the repository baseline and proposal. | Treat the wildcard as existing in this comparison, not newly introduced. Review shared guidance without silently changing the Slot contract. |
| F07 | Medium | Both OAS files declare 1.4.0-alpha; Appointment JSON media type remains version=1.0.0. | Rebase and align API, Core and negotiated payload versions deliberately. |

F03–F07 are findings from the structural comparison of the repository files linked above. F01–F02 also rely on the cited R4 definitions. Runtime behaviour was not tested.

## Standard and documentation impact

All links below target guide 1.12.0 / Core 1.5.0. These are proposed edits to existing pages, not claims that those pages already specify the new feature.

| ID | Location | Required uplift |
|---|---|---|
| S01 | [Appointment pattern](https://simplifier.net/guide/nhsbookingandreferralstandard/Home/Core/1-5-0/Appointment-StandardPattern?version=1.12.0) | Extend View (Search): optional inclusions, permitted graph, prerequisites and mixed Bundle handling. Separate collection search from GET /Appointment/{id}. |
| S02 | [End-to-end workflow](https://simplifier.net/guide/nhsbookingandreferralstandard/Home/Core/1-5-0/End-to-end-workflow?version=1.12.0) and [Content negotiation](https://simplifier.net/guide/nhsbookingandreferralstandard/Home/Analysis/Content-Negotiation?version=1.12.0) | Explain support discovery, request selection and omissions. Add or amend a sender → proxy → receiver sequence showing receiver-side resolution. |
| S03 | [Booking sender](https://simplifier.net/guide/nhsbookingandreferralstandard/Home/Core/1-5-0/Core-Functionality-Requirements/Booking-Sender?version=1.12.0) and [Booking receiver](https://simplifier.net/guide/nhsbookingandreferralstandard/Home/Core/1-5-0/Core-Functionality-Requirements/Booking-Receiver?version=1.12.0) | Define optional capability versus any application-specific mandatory minimum. Optional use is different from mandatory implementation. |
| S04 | [Bundle guidance](https://simplifier.net/guide/nhsbookingandreferralstandard/Home/Core/1-5-0/BaRS-FHIR-Usage/Bundle?version=1.12.0) | Add Appointment searchsets, match/include/outcome entries, totals, identity and duplicate handling. Current narrative mainly covers messaging and Slot searchsets. |
| S05 | [Update](https://simplifier.net/guide/nhsbookingandreferralstandard/Home/Core/1-5-0/Appointment-StandardPattern/Update-Existing-Booking?version=1.12.0), [Cancel](https://simplifier.net/guide/nhsbookingandreferralstandard/Home/Core/1-5-0/Appointment-StandardPattern/Cancel-Booking?version=1.12.0), [Reschedule](https://simplifier.net/guide/nhsbookingandreferralstandard/Home/Core/1-5-0/Appointment-StandardPattern/Reschedule-Existing-Booking?version=1.12.0), [Rebook](https://simplifier.net/guide/nhsbookingandreferralstandard/Home/Core/1-5-0/Appointment-StandardPattern/Rebook-Methods?version=1.12.0) | Explain optional context retrieval. Preserve read-before-write/freshness rules; do not imply includes work on the single-resource read. Never submit the whole searchset as an Appointment update. |
| S06 | [Failure scenarios](https://simplifier.net/guide/nhsbookingandreferralstandard/Home/Core/1-5-0/Error-Handling/Failure-Scenarios-1-1-x?version=1.12.0) | Separate unsupported includes, malformed queries, absent references and authorisation/server failures. Specify self-link behaviour. |
| S07 | [FHIR assets](https://simplifier.net/guide/nhsbookingandreferralstandard/Home/FHIR-Assets?version=1.12.0) | Publish validated default, direct, iterative and partial-response examples and updated CapabilityStatements. Select applicable profiles for each included type. |
| S08 | [Security](https://simplifier.net/guide/nhsbookingandreferralstandard/Home/Core/1-5-0/Security-and-Authorisation?version=1.12.0) and [Processing times](https://simplifier.net/guide/nhsbookingandreferralstandard/Home/Core/1-5-0/Non-Functional-Requirements/Processing-Times?version=1.12.0) | Clarify per-resource access and permitted reference resolution; assess fan-out and response size. |
| S09 | [Testing](https://simplifier.net/guide/nhsbookingandreferralstandard/Home/Build/Testing-and-Environments?version=1.12.0) and [Assurance](https://simplifier.net/guide/nhsbookingandreferralstandard/Home/Assure/Assure?version=1.12.0) | Update relevant mocks, TKW scenarios and conformance evidence. Internal inventories still need checking. |

Add a release entry and minimum capability version. The verified appointment guidance path is Appointment-StandardPattern; the supplied report’s generic Standard-Pattern/Booking path should not be used as an edit location without verification.

## Application impact: refine the supplied list

Impact follows actual appointment-search use. Applications 1 and 2 are clear booking-workflow review candidates. Application 2 covers 111 Online/streaming and redirection, not simply 111 to UTC. Applications 3–5 should not be labelled affected booking applications without checking their workflows. Core explicitly identifies 999–CAS validation as having no booking step. [Application catalogue](https://simplifier.net/guide/nhsbookingandreferralstandard/Home/Applications/BaRS-Applications?version=1.12.0), [Core workflow scope](https://simplifier.net/guide/nhsbookingandreferralstandard/Home/Core/1-5-0?version=1.12.0).

The pre-release catalogue identifies Application 6 as referrals into an Ambulance Service Trust, Application 7 as bookings into GP Practice, and Application 8 as referrals into a broker. Application 7 is a review candidate missing from the supplied list. Application 9 was not listed in the reviewed catalogues; its identity and relevance remain unverified. [Pre-release catalogue](https://simplifier.net/guide/nhsbookingandreferralstandard/Home/Applications/BaRS-Pre-releases?version=1.12.0).

For confirmed consumers, add a Core cross-reference and any required minimum includes. An optional search enhancement does not automatically require new message payloads or version changes to every application.

## Response and compatibility contract

Recommended BaRS rules:

- Preserve patient-selection and target-service rules. Includes enrich matching appointments; an include-only request must not enable an unrestricted search.
- Keep no-include behaviour unchanged. Distinguish matched Appointments from included resources and optional outcome entries. Bundle.total counts matching Appointments, not all entries. Resolve references by fullUrl/resource identity, not entry order. [FHIR Bundle definitions](https://hl7.org/fhir/R4/bundle-definitions.html).
- Deduplicate shared resources within each page. Define paging, reference/version handling and resource limits; each page should contain its relevant context.
- Retain the proposal’s lenient handling for well-formed unsupported values, reflecting accepted parameters in the self link. Distinguish unsupported parameters from supported parameters whose referenced resource is absent. Decide whether Prefer: handling=strict is supported. Malformed queries and access failures need separate error rules. [FHIR search rules](https://hl7.org/fhir/R4/search.html#errors).
- Do not equate a successful response with complete context or interpret omitted resources as clinical absence.

The newer OAS lists demographic alternatives, while failure guidance still describes NHS number as mandatory. Reconcile this when specifying validation. Metadata also advertises _id/date searches not listed in the operation parameters. Do not silently introduce _id search to make this enhancement work for a single appointment.

## Implementation and operational impact

| Component | Work to assess |
|---|---|
| Proxy | Preserve repeated keys and :iterate; update parameter validation and routing tests. Keep service boundaries and accurate response links. |
| Receiver | Resolve the allowed graph, batch reads, apply access controls, assemble mixed Bundles, deduplicate and bound traversal. |
| Sender | Discover support, serialize repeated parameters, parse multiple types and handle partial context. |
| Data model | Ensure requested links are resolvable. Define absent, identifier-only and external-reference handling; an enum cannot create missing data. |
| Security | Appointment access must not automatically grant access to its ServiceRequest. Avoid unrestricted remote retrieval and cross-organisation leakage. |
| Performance | Test response growth, database fan-out, timeouts and caching against existing Core limits. |
| Wayfinder display | Agree minimum context and fallback behaviour. A service may have several locations; an included Location is not automatically proof of the booked venue. |

These are inferred impacts, not confirmed implementation defects. Core’s reviewed processing guidance specifies all requests below 5000 ms and a 90% target below 2100 ms, excluding transport. [Processing times](https://simplifier.net/guide/nhsbookingandreferralstandard/Home/Core/1-5-0/Non-Functional-Requirements/Processing-Times?version=1.12.0). Location multiplicity is defined in [FHIR HealthcareService](https://hl7.org/fhir/R4/healthcareservice.html).

## Relationship to NRL removal

The changes can be delivered independently. Includes enrich results from a known service; they do not discover unknown appointments nationally, register pointers or replace NRL discovery. Coordinate appointment-pattern wording so both changes describe a coherent retrieval flow.

## Delivery and acceptance

Correct syntax, mixed Bundle modelling and metadata first. Then update Core/examples together, confirm actual application consumers, and implement against an agreed release. Acceptance should cover default behaviour, each direct/iterative branch, missing and inaccessible references, duplicates, errors, self links, totals, paging where offered, JSON/XML parity where advertised, proxy forwarding and unchanged appointment lifecycle safeguards. The companion inventory provides exact edit locations and test cases.

## Method and limits

Downloaded and structurally compared both repository OAS files and read the supplied report on 16 September 2026. Reviewed 18 targeted guide 1.12.0 pages plus the Core index and FHIR R4 primary references. Published API checks used earlier downloaded snapshots and web representations checked during this review. No deployed calls, full package validation, supplier-code audit or internal assurance review was performed. Repository main and public pages can change; source hashes are recorded in the companion inventory.
