# BaRS: impact of removing National Record Locator references

Review date: 14 September 2026  
Problem statement added: 21 September 2026  
Scope: public BaRS implementation guidance and published API specifications. No standards, systems or live records have been changed.

## Problem statement: why NRL should be removed from BaRS

BaRS currently describes an NRL integration that was never completed. Its NRL-related API operations are dead ends: they present suppliers with a documented route that does not deliver the intended end-to-end capability. Keeping these operations and their associated requirements in the standard creates a misleading expectation of what suppliers can implement and what BaRS supports.

This creates four problems:

1. **Incomplete functionality is presented as part of the standard.** The documented NRL operations and registry-dependent workflows imply an available capability, although the integration was never completed. Their continued inclusion leaves a gap between the published contract and the functionality that can be delivered.
2. **Suppliers are confused about what they need to build.** Suppliers cannot reliably distinguish supported requirements from unfinished proposals. They may spend time designing against dead-end operations, seeking clarification or planning registry integration that cannot be completed. This introduces avoidable implementation effort and uncertainty about conformance.
3. **The underlying use case has never had full product-owner support.** Without that support, there is no sufficiently agreed basis for requiring suppliers to implement this approach. Retaining it in the standard allows an unresolved product proposal to appear to be an established commitment.
4. **The original approach needs to be reassessed.** There may now be better or alternative ways to meet the underlying discovery and access needs. Continuing to prescribe NRL within BaRS constrains that assessment before the use case, ownership and options have been agreed. No replacement is assumed or selected by this report.

NRL-specific operations, references and dependent requirements should therefore be removed from the active BaRS standard and API through a coordinated change. This would give suppliers a clearer, implementable contract and allow any future record-discovery requirement to be considered on its own merits, with explicit product ownership and an assessment of the available approaches. Direct booking and referral functionality should be preserved.

*Basis: the implementation status, supplier confusion and product-owner support described above are project context supplied for this report on 21 September 2026. They were not independently established by the public-document review. The possibility of alternative approaches is a reason for reassessment, not a finding that a particular replacement is available or superior.*

## Assessment

Removing NRL from BaRS is more than an editorial change. The implementation guide explicitly maps five BaRS DocumentReference operations to NRL, and the appointment pattern depends on a central registry for discovery and pointer maintenance. Removing the name alone would leave those requirements in place. [DocumentReference interface](https://simplifier.net/guide/nhsbookingandreferralstandard/Home/Core/1-4-1/DocumentReference-StandardPattern/DocumentReference-Interface?version=1.11.1)

This report assumes the intended outcome is to remove the NRL dependency from the future BaRS standard and API. If the intention is only to remove branding while retaining a registry, the affected locations remain relevant, but the registry contract, ownership, access controls and replacement service must be specified instead of deleting the functionality.

The recommended boundary is to retire NRL pointer discovery and maintenance where they are exposed through BaRS, while preserving direct booking/referral operations and ordinary FHIR document references. Whether any published operation can be withdrawn immediately depends on deployment and supplier usage; the public documentation does not establish that.

## Baseline and confidence

| Publication | Observed baseline | Relevance |
|---|---|---|
| NHS BaRS service page | Edited 20 August 2026 | Entry point to standard and applications |
| Implementation guide linked from API catalogue | Guide 1.11.1; Core 1.4.1; package 1.40.0 | Main standard reviewed |
| API catalogue v1.0.7 | Downloaded OAS identifies itself as 1.0.0 | No DocumentReference paths or literal NRL references found |
| API catalogue v1.4.1 | Downloaded OAS identifies itself as 1.4.1-alpha | Five pointer operations; five NRL-bearing string locations |
| API catalogue v1.5.0 | Downloaded OAS identifies itself as 1.5.0-alpha | Same five pointer operations and five NRL-bearing locations |

Sources: [BaRS service](https://digital.nhs.uk/services/booking-and-referral-standard), [guide](https://simplifier.net/guide/nhsbookingandreferralstandard/home?version=1.11.1), [API catalogue](https://digital.nhs.uk/developer/api-catalogue/booking-and-referral-fhir), [v1.0.7 OAS](https://digital.nhs.uk/restapi/oas/413106), [v1.4.1 OAS](https://digital.nhs.uk/restapi/oas/613162), [v1.5.0 OAS](https://digital.nhs.uk/restapi/oas/621537).

There is a material publication inconsistency: the catalogue labels all three versions “In production”, but the two newer OAS descriptions say development/no production availability. Core 1.4.1 also contains preview language for endpoint management. Treat these as documentation findings, not proof of the deployed release or whether NRL integration is live. Resolve the supported baseline before announcing withdrawal.

## Standard: confirmed affected locations

The following are confirmed content dependencies. “Remove” means remove from the successor standard if pointer functionality is being retired; preserve versioned historical specifications or annotate them as superseded.

| ID | Location | Affected content and proposed action |
|---|---|---|
| S01 | [Core 1.4.1 overview](https://simplifier.net/guide/nhsbookingandreferralstandard/Home/Core/1-4-1?version=1.11.1) | DocumentReference pattern summary and directory entries. Remove the obsolete capability and links, or replace them with an explicitly defined alternative. |
| S02 | [DocumentReference pattern](https://simplifier.net/guide/nhsbookingandreferralstandard/Home/Core/1-4-1/DocumentReference-StandardPattern?version=1.11.1) and [Introduction](https://simplifier.net/guide/nhsbookingandreferralstandard/Home/Core/1-4-1/DocumentReference-StandardPattern/Introduction?version=1.11.1) | BaRS-as-NRL-gateway description, consumer/producer roles and registry architecture. Retire the pattern if NRL-backed discovery is removed. The combined page repeats child content; update the source and verify both views. |
| S03 | [DocumentReferences for Senders](https://simplifier.net/guide/nhsbookingandreferralstandard/Home/Core/1-4-1/DocumentReference-StandardPattern/Sender-DocumentReference?version=1.11.1) | Patient pointer search, interpretation of returned identifiers/types, and subsequent retrieval from the owning service. Remove NRL discovery steps; retain direct retrieval guidance where the service and resource ID are already known. Specify what users do when they are not known. |
| S04 | [DocumentReferences for Receivers](https://simplifier.net/guide/nhsbookingandreferralstandard/Home/Core/1-4-1/DocumentReference-StandardPattern/Receiver-DocumentReference?version=1.11.1) | Creation after accepting bookings/referrals, pointer field population, saving to NRL, verification, read-before-update/delete and ownership restrictions. Remove these registry duties, including the requirement for a distinct pointer per accepted request. |
| S05 | [DocumentReference interface](https://simplifier.net/guide/nhsbookingandreferralstandard/Home/Core/1-4-1/DocumentReference-StandardPattern/DocumentReference-Interface?version=1.11.1) | All five BaRS-to-NRL mappings, forwarding behaviour, producer authorisation, search parameters, payloads and response tables. Retire or replace the complete contract. |
| S06 | [Appointment pattern overview](https://simplifier.net/guide/nhsbookingandreferralstandard/Home/Core/1-4-1/Appointment-StandardPattern?version=1.11.1) | Registry dependency and List capability using GET /DocumentReference. Distinguish national discovery from searching a known service. Remove the national-list promise unless a replacement is defined. |
| S07 | [Initial booking](https://simplifier.net/guide/nhsbookingandreferralstandard/Home/Core/1-4-1/Appointment-StandardPattern/Initial-Booking?version=1.11.1) | Receiver creates a registry pointer after booking. Remove this post-booking requirement; keep booking creation and retention of the returned appointment ID. |
| S08 | [Update existing booking](https://simplifier.net/guide/nhsbookingandreferralstandard/Home/Core/1-4-1/Appointment-StandardPattern/Update-Existing-Booking?version=1.11.1) | Registry lookup when the appointment ID is unknown. Replace this fallback with an agreed service-specific search or manual process. |
| S09 | [Cancel booking](https://simplifier.net/guide/nhsbookingandreferralstandard/Home/Core/1-4-1/Appointment-StandardPattern/Cancel-Booking?version=1.11.1) | Registry lookup plus deletion of the pointer after cancellation. Remove the registry actions while retaining appointment cancellation. Cancelling a booking and deleting its pointer are separate actions. |
| S10 | [Reschedule existing booking](https://simplifier.net/guide/nhsbookingandreferralstandard/Home/Core/1-4-1/Appointment-StandardPattern/Reschedule-Existing-Booking?version=1.11.1) | Registry lookup plus PUT of the pointer to update its context.period. Remove those steps; preserve the appointment/slot change. |
| S11 | [Rebook methods](https://simplifier.net/guide/nhsbookingandreferralstandard/Home/Core/1-4-1/Appointment-StandardPattern/Rebook-Methods?version=1.11.1) | Creation of a new registry pointer for the new booking; inherited cancellation requirements also matter. Remove both registry obligations from the future flow. |
| S12 | [Releases](https://simplifier.net/guide/nhsbookingandreferralstandard/Home/Analysis/Releases?version=1.11.1), [Core change log](https://simplifier.net/guide/nhsbookingandreferralstandard/Home/Analysis/Releases/Technical-Release-Notes/BaRS-Core?version=1.11.1) and [API change log](https://simplifier.net/guide/nhsbookingandreferralstandard/Home/Analysis/Releases/Technical-Release-Notes/API-Spec-Change-Log?version=1.11.1) | Release descriptions still advertise registry and DocumentReference additions. Add a clear removal/deprecation entry and update current summaries. Retain accurate historical entries. |

The pointer model also contains the booking/referral identifier, service identifier, product identifier, patient NHS number, custodian and record type. These fields should cease to be required *for an NRL pointer* if the pointer contract is retired. That does not justify removing the same identifiers or demographic fields from ordinary BaRS messages. See the sender and receiver pages above.

The linked architecture image is also affected: its SVG contains NRLF Consumer API and NRLF Producer API labels, including embedded diagram source. Update or retire the image and its references together. [Diagram asset](https://raw.githubusercontent.com/NHSDigital/NHSDigital-FHIR-BookingAndReferrals/main/BaRS-Images/DocumentReference/NRLF%20Via%20BaRS-1.1.0.svg).

## API: confirmed affected contract

| Operation | Capability lost if removed |
|---|---|
| GET /DocumentReference | Patient-level discovery of booking/referral pointers |
| POST /DocumentReference | Registering pointers |
| GET /DocumentReference/{id} | Reading a pointer, including before maintenance |
| PUT /DocumentReference/{id} | Updating pointers |
| DELETE /DocumentReference/{id} | Removing pointers |

These five operations occur in both newer published specifications. The guide maps search to the NRL consumer API and the remaining four operations to its producer API. [Interface mapping](https://simplifier.net/guide/nhsbookingandreferralstandard/Home/Core/1-4-1/DocumentReference-StandardPattern/DocumentReference-Interface?version=1.11.1).

The API cleanup must also cover:

- The pointer-management overview bullet and DocumentReference tag.
- DocumentReference entries in both server and client CapabilityStatement examples under GET /metadata.
- DocumentReference request body, DocumentReference and DocumentReferenceBundle schemas, examples and associated response content.
- RegistryId_Param, subject_QParam, custodian_QParam, type_QParam and nextPageToken_QParam. Reference tracing in v1.5.0 found these used only by the pointer paths.
- NRL-specific format coding and response coding, plus the comparison with NRL STU3 in the pointer update description.

These are candidates for removal from the successor OAS after its reference graph is rechecked. Keep shared headers, use-context_HParam, generic outcome schemas and common error responses. They are also used by retained operations. [v1.5.0 OAS](https://digital.nhs.uk/restapi/oas/621537).

A less obvious issue is the successful DELETE response example for both Appointment/{id} and ServiceRequest/{id}: each uses `https://fhir.nhs.uk/CodeSystem/NRLF-SuccessCode`. Correct those examples and verify the intended response contract rather than deleting the booking/referral operations. Do not mechanically rename a terminology URI: its meaning and any supplier dependence need checking. [v1.4.1 OAS](https://digital.nhs.uk/restapi/oas/613162).

The companion technical inventory identifies every literal NRL-bearing OAS string and the relevant component locations, so the specification edits can be assigned directly.

## Functional and implementation consequences

These are impact assessments inferred from the confirmed standard dependencies, not observations of live systems.

| Area | Consequence | Required decision or check |
|---|---|---|
| Discovery | A caller may no longer find an unknown booking/referral across services through BaRS. | Decide whether cross-service discovery is withdrawn or replaced. A patient search against a known receiver is not an equivalent national search. |
| Receiver processing | Pointer creation and maintenance no longer follow accepted bookings/referrals. | Remove registry calls, retries and related failure handling only after checking which implementations perform them. |
| Appointment lifecycle | Booking, cancellation, rescheduling and rebooking cease to maintain registry entries. | Confirm user-visible flows still work when IDs are retained or the receiving service can be searched. |
| Existing pointers | Stopping updates could leave stale discovery results in NRL for other consumers. | Establish whether pointers exist, who owns them, and an agreed retirement/migration process. This report does not authorise deletion. |
| Proxy and infrastructure | The documented forwarding and authorisation boundary changes. | Inspect deployed routes, upstream configuration, credentials, permissions, monitoring and operational runbooks. No runtime configuration was available for this review. |
| Suppliers and consumers | Code generated from an older contract may still call pointer operations. | Check usage and supplier commitments, publish migration guidance, and use a versioned withdrawal where necessary. |
| Testing and assurance | Pointer-specific tests may become obsolete; direct flows still need assurance. | Review SCAL, TKW scenarios, mocks, fixtures and deployment checks. Their internal inventories were not inspected. |
| Clinical and information governance | Record discoverability and any registry failure assumptions change. | Review the actual affected safety hazards and data flows with their owners; do not assume the whole BaRS assurance baseline needs replacement. |

## What should remain

Retain the direct booking and referral functions, message processing, metadata negotiation, slot discovery, service routing and shared authentication/transaction headers unless a separate requirement changes them. The registry finds records; the BaRS endpoint catalogue routes requests to a known service. They have different purposes. [Core overview](https://simplifier.net/guide/nhsbookingandreferralstandard/Home/Core/1-4-1?version=1.11.1).

Do not delete every FHIR `DocumentReference` occurrence. Application pages include inherited FHIR references, such as clinical supporting information, observation derivation or consent-source references. These are not evidence of NRL integration. The review found no literal NRL mention in the retrieved Application 1–8 pages; that does not remove their dependency on applicable Core requirements. [Applications catalogue](https://simplifier.net/guide/nhsbookingandreferralstandard/Home/Applications/BaRS-Applications?version=1.11.1).

No literal NRL reference was found in the retrieved infrastructure, testing/environments, assurance or FHIR-assets landing pages. They remain review areas for linked or unpublished material, rather than confirmed text edits.

## Related publication cleanup

The NRL service page still describes BaRS as considering NRL for booking/referral discovery. Coordinate a correction to that relationship if the plan is dropped. Its roadmap also records booking/referral pointer support in 2023; that historical entry should not be erased merely because BaRS changes direction. [NRL service](https://digital.nhs.uk/services/national-record-locator), [NRL roadmap](https://digital.nhs.uk/services/national-record-locator/roadmap).

The published BaRS requirements specification already records a 2021 change removing NRL references. This is historical context, not evidence that the current technical standard is NRL-free. Preserve the audit history and reconcile the current documents. [Requirements specification v3.0, revision history](https://digital.nhs.uk/binaries/content/assets/website-assets/corporate-information/directions-and-data-provision-notices/nhs-england-directions/2025/booking-and-referral-requirements-specification-v3.0.pdf).

## Suggested delivery order and acceptance criteria

1. Agree the target release and whether the change removes only NRL integration or all registry-based discovery. Resolve the catalogue/OAS status mismatch and identify actual users.
2. Revise Core: retire or replace the pointer pattern, amend all five appointment lifecycle pages, update overview links and diagrams, and publish a clear change entry.
3. Revise the OAS and its source assets: update pointer paths, metadata, components and terminology examples together. Preserve shared elements and direct operations.
4. Check implementation and transition work with service owners: routing, access, existing pointers, supplier migration and consumer behaviour.
5. Validate the release: no unexplained active NRL/NRLF references; no residual mandatory registry calls; no broken internal links or OAS references; CapabilityStatements match supported operations; retained booking/referral flows pass appropriate regression checks; historical material is clearly identified.

## Review method and limits

The review downloaded and searched 738 distinct guide URLs within guide version 1.11.1, including 82 Core 1.4.1 URLs and linked historical core content. Twenty-two retrieved pages contained literal NRL/NRLF/National Record text; this includes combined pages and historical duplicates, so it is not a count of independent requirements. Searches also covered DocumentReference, document reference, pointer and registry terms, and relevant matches were inspected in context. All discovered links to Core 1.4.1 pages within the retrieved guide set were covered.

All three OAS files linked by the API catalogue were parsed, including descriptions, examples and component references. The NRL diagram's SVG labels were inspected. This was a public-document review, not an exhaustive crawl of every external dependency, every historical guide version or every embedded image. The FHIR package archive, source repositories, private assurance artefacts, production configuration, usage logs and supplier systems were not audited. Download links can change; findings apply to the versions retrieved on the review date.
