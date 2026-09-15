# BaRS NRL removal: technical inventory

Reviewed 14 September 2026. Use alongside the impact report. Paths below identify the downloaded published OAS, not a verified source-repository layout.

## Literal NRL references in published OAS

### Catalogue v1.0.7

[Published OAS](https://digital.nhs.uk/restapi/oas/413106) — internal version `1.0.0`.

No literal NRL match found; no DocumentReference paths.


### Catalogue v1.4.1

[Published OAS](https://digital.nhs.uk/restapi/oas/613162) — internal version `1.4.1-alpha`.

- JSON Pointer: `/paths/~1Appointment~1{id}/delete/responses/200/content/application~1fhir+json;version=1.0.0/example/issue/0/details/coding/0/system`
  Value: `https://fhir.nhs.uk/CodeSystem/NRLF-SuccessCode`
- JSON Pointer: `/paths/~1ServiceRequest~1{id}/delete/responses/200/content/application~1fhir+json;version=1.0.0/example/issue/0/details/coding/0/system`
  Value: `https://fhir.nhs.uk/CodeSystem/NRLF-SuccessCode`
- JSON Pointer: `/paths/~1DocumentReference/post/description`
  Finding: NRL-specific document format code system.
- JSON Pointer: `/paths/~1DocumentReference~1{id}/put/description`
  Finding: Historical NRL STU3 status/deletion comparison.
- JSON Pointer: `/paths/~1DocumentReference~1{id}/delete/responses/200/content/application~1fhir+json;version=1.4.0/example/issue/0/details/coding/0/system`
  Value: `https://fhir.nhs.uk/ValueSet/NRL-ResponseCode`

### Catalogue v1.5.0

[Published OAS](https://digital.nhs.uk/restapi/oas/621537) — internal version `1.5.0-alpha`.

- JSON Pointer: `/paths/~1Appointment~1{id}/delete/responses/200/content/application~1fhir+json;version=1.0.0/example/issue/0/details/coding/0/system`
  Value: `https://fhir.nhs.uk/CodeSystem/NRLF-SuccessCode`
- JSON Pointer: `/paths/~1ServiceRequest~1{id}/delete/responses/200/content/application~1fhir+json;version=1.0.0/example/issue/0/details/coding/0/system`
  Value: `https://fhir.nhs.uk/CodeSystem/NRLF-SuccessCode`
- JSON Pointer: `/paths/~1DocumentReference/post/description`
  Finding: NRL-specific document format code system.
- JSON Pointer: `/paths/~1DocumentReference~1{id}/put/description`
  Finding: Historical NRL STU3 status/deletion comparison.
- JSON Pointer: `/paths/~1DocumentReference~1{id}/delete/responses/200/content/application~1fhir+json;version=1.5.0/example/issue/0/details/coding/0/system`
  Value: `https://fhir.nhs.uk/ValueSet/NRL-ResponseCode`

## Additional contract locations to update

Apply to both newer specifications, checking each version before editing. These are not all literal NRL matches.

- `/info/description`: pointer capability summary.
- `/tags`: DocumentReference tag.
- `/paths/~1metadata/get/responses/200/content/application~1fhir+json/example/rest`: DocumentReference capability in server and client resource lists.
- `/paths/~1DocumentReference` and `/paths/~1DocumentReference~1{id}`: all five operations, embedded examples, responses and links.
- `/components/requestBodies/DocumentReference`.
- `/components/schemas/DocumentReference` and `/components/schemas/DocumentReferenceBundle`.
- `/components/parameters/RegistryId_Param`, `subject_QParam`, `custodian_QParam`, `type_QParam`, `nextPageToken_QParam`.

Retain shared `use-context_HParam`, access/transaction headers, `OperationalOutcome`, and `4XX-BARS` / `5XX-BARS` unless independently superseded.

## Guide URLs with literal NRL text

Historical paths and combined views can duplicate requirements. Update future active content; preserve or annotate published history.

- Release notes: [Analysis/Releases](https://simplifier.net/guide/nhsbookingandreferralstandard/Home/Analysis/Releases?version=1.11.1)
- Current Core: [Core/1-4-1/DocumentReference-StandardPattern](https://simplifier.net/guide/nhsbookingandreferralstandard/Home/Core/1-4-1/DocumentReference-StandardPattern?version=1.11.1)
- Current Core: [Core/1-4-1/DocumentReference-StandardPattern/Sender-DocumentReference](https://simplifier.net/guide/nhsbookingandreferralstandard/Home/Core/1-4-1/DocumentReference-StandardPattern/Sender-DocumentReference?version=1.11.1)
- Current Core: [Core/1-4-1/DocumentReference-StandardPattern/Receiver-DocumentReference](https://simplifier.net/guide/nhsbookingandreferralstandard/Home/Core/1-4-1/DocumentReference-StandardPattern/Receiver-DocumentReference?version=1.11.1)
- Current Core: [Core/1-4-1/DocumentReference-StandardPattern/DocumentReference-Interface](https://simplifier.net/guide/nhsbookingandreferralstandard/Home/Core/1-4-1/DocumentReference-StandardPattern/DocumentReference-Interface?version=1.11.1)
- Historical Core path: [Core/1-4-0/DocumentReference-StandardPattern/Sender-DocumentReference](https://simplifier.net/guide/nhsbookingandreferralstandard/Home/Core/1-4-0/DocumentReference-StandardPattern/Sender-DocumentReference?version=1.11.1)
- Historical Core path: [Core/1-4-0/DocumentReference-StandardPattern/Receiver-DocumentReference](https://simplifier.net/guide/nhsbookingandreferralstandard/Home/Core/1-4-0/DocumentReference-StandardPattern/Receiver-DocumentReference?version=1.11.1)
- Historical Core path: [Core/1-4-0/DocumentReference-StandardPattern/DocumentReference-Interface](https://simplifier.net/guide/nhsbookingandreferralstandard/Home/Core/1-4-0/DocumentReference-StandardPattern/DocumentReference-Interface?version=1.11.1)
- Historical Core path: [Core/1.3.1/DocumentReference-StandardPattern/Sender-DocumentReference](https://simplifier.net/guide/nhsbookingandreferralstandard/Home/Core/1.3.1/DocumentReference-StandardPattern/Sender-DocumentReference?version=1.11.1)
- Historical Core path: [Core/1.3.1/DocumentReference-StandardPattern/Receiver-DocumentReference](https://simplifier.net/guide/nhsbookingandreferralstandard/Home/Core/1.3.1/DocumentReference-StandardPattern/Receiver-DocumentReference?version=1.11.1)
- Historical Core path: [Core/1.3.1/DocumentReference-StandardPattern/DocumentReference-Interface](https://simplifier.net/guide/nhsbookingandreferralstandard/Home/Core/1.3.1/DocumentReference-StandardPattern/DocumentReference-Interface?version=1.11.1)
- Historical Core path: [Core/1.1.6/DocumentReference-StandardPattern/Receiver-DocumentReference](https://simplifier.net/guide/nhsbookingandreferralstandard/Home/Core/1.1.6/DocumentReference-StandardPattern/Receiver-DocumentReference?version=1.11.1)
- Historical Core path: [Core/1.1.6/DocumentReference-StandardPattern/DocumentReference-Interface](https://simplifier.net/guide/nhsbookingandreferralstandard/Home/Core/1.1.6/DocumentReference-StandardPattern/DocumentReference-Interface?version=1.11.1)
- Historical Core path: [Core/1.1.5/DocumentReference-StandardPattern/Receiver-DocumentReference](https://simplifier.net/guide/nhsbookingandreferralstandard/Home/Core/1.1.5/DocumentReference-StandardPattern/Receiver-DocumentReference?version=1.11.1)
- Historical Core path: [Core/1.1.5/DocumentReference-StandardPattern/DocumentReference-Interface](https://simplifier.net/guide/nhsbookingandreferralstandard/Home/Core/1.1.5/DocumentReference-StandardPattern/DocumentReference-Interface?version=1.11.1)
- Historical Core path: [Core/1.1.4/DocumentReference-StandardPattern/Receiver-DocumentReference](https://simplifier.net/guide/nhsbookingandreferralstandard/Home/Core/1.1.4/DocumentReference-StandardPattern/Receiver-DocumentReference?version=1.11.1)
- Historical Core path: [Core/1.1.4/DocumentReference-StandardPattern/DocumentReference-Interface](https://simplifier.net/guide/nhsbookingandreferralstandard/Home/Core/1.1.4/DocumentReference-StandardPattern/DocumentReference-Interface?version=1.11.1)
- Historical Core path: [Core/1.1.3/DocumentReference-StandardPattern/Receiver-DocumentReference](https://simplifier.net/guide/nhsbookingandreferralstandard/Home/Core/1.1.3/DocumentReference-StandardPattern/Receiver-DocumentReference?version=1.11.1)
- Historical Core path: [Core/1.1.3/DocumentReference-StandardPattern/DocumentReference-Interface](https://simplifier.net/guide/nhsbookingandreferralstandard/Home/Core/1.1.3/DocumentReference-StandardPattern/DocumentReference-Interface?version=1.11.1)
- Current Core: [Core/1-4-1/DocumentReference-StandardPattern/Introduction](https://simplifier.net/guide/nhsbookingandreferralstandard/Home/Core/1-4-1/DocumentReference-StandardPattern/Introduction?version=1.11.1)
- Historical Core path: [Core/1.3.0/DocumentReference-StandardPattern/Receiver-DocumentReference](https://simplifier.net/guide/nhsbookingandreferralstandard/Home/Core/1.3.0/DocumentReference-StandardPattern/Receiver-DocumentReference?version=1.11.1)
- Historical Core path: [Core/1.3.0/DocumentReference-StandardPattern/DocumentReference-Interface](https://simplifier.net/guide/nhsbookingandreferralstandard/Home/Core/1.3.0/DocumentReference-StandardPattern/DocumentReference-Interface?version=1.11.1)

## Registry dependencies without the NRL name

The impact report separately lists all appointment-pattern locations, including initial booking, update, cancel, reschedule and rebook. A literal NRL search alone will miss these.
