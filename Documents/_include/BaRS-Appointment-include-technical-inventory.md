# GET /Appointment _include: technical edit inventory

Reviewed 16 September 2026. Companion to the impact report. This is an edit plan, not a patched specification.

Existing locations refer to the [proposed repository OAS](https://github.com/RichardWardNHSD/BaRS-and-Wayfinder/blob/main/BaRS%20-%20OAS/bars_api_OAS-with_include.json), compared with the [repository baseline](https://github.com/RichardWardNHSD/BaRS-and-Wayfinder/blob/main/BaRS%20-%20OAS/bars%20api%20OAS.json). JSON Pointers escape slashes as ~1.

## Verified repository delta

1. Added /components/parameters/AppointmentInclude_QParam.
2. Appended its reference at /paths/~1Appointment/get/parameters/4.
3. Replaced the JSON response example with examples/BookingsForPatient and examples/BookingsForPatientWithIncludes under /paths/~1Appointment/get/responses/200/content/application~1fhir+json;version=1.0.0.

No other JSON value changes were found. In particular, metadata, SearchBundleAppt, GET /Slot and info.version are unchanged.

## Edit register

| ID | JSON Pointer / asset | Action |
|---|---|---|
| A01 | /components/parameters/AppointmentInclude_QParam | Keep optional/repeatable. Restrict direct values to Appointment:slot and Appointment:based-on for this scope. Correct description; specify style=form and explode=true. |
| A02 | NEW /components/parameters/AppointmentIncludeIterate_QParam | Add optional query parameter named _include:iterate for the six downstream traversals. Use HealthcareService:organization. |
| A03 | /paths/~1Appointment/get/parameters | Reference both components at operation level; shared path-level parameters would also affect POST. Preserve patient criteria. |
| A04 | /paths/~1Appointment/get/description | Explain optional related resources and distinguish collection search from single-resource read. |
| A05 | /components/schemas/SearchBundleAppt/properties/entry/items/properties/resource | Accurately model Appointment plus Slot, ServiceRequest, Schedule, Practitioner, PractitionerRole, HealthcareService, Organization and Location. Include OperationOutcome if warning entries are supported. |
| A06 | /components/schemas/SearchBundleAppt/properties/entry/items/properties/search | Describe match/include/outcome. Existing mode is a free string with match as its example, not a restrictive enum. |
| A07 | /components/schemas/SearchBundleAppt/properties/total and /properties/link | Explain match totals, self links and paging. Check fullUrl and reference identity too. |
| A08 | /paths/~1Appointment/get/responses/200/content | Apply the response model to both advertised JSON and XML. Validate examples against the selected profiles. |
| A09 | JSON examples/BookingsForPatientWithIncludes | Correct iteration syntax in summary, description and self link. Fix the unresolved participant UUID. Preserve total=1 for one matched Appointment. Add referral and organisation examples. |
| A10 | /paths/~1metadata/get/responses/200/content/application~1fhir+json/example/rest/0/resource/2 | Verified server Appointment entry: add searchInclude values and explain iterative support/limits in guidance. |
| A11 | /paths/~1metadata/get/responses/200/content/application~1fhir+json/example/rest/1/resource/0 | Verified client Appointment entry: align with the client role actually exercised. Proxy routing alone does not establish receiver capability. |
| A12 | /components/schemas/Capability | Verify the schema and generated examples represent the metadata changes; searchInclude already exists elsewhere in newer baselines. |
| A13 | /paths/~1Slot/get/parameters and Slot metadata | Related consistency review only. Slot:* already exists. The metadata spelling HealthcareService.providedBy and enum spelling HealthcareService:providedBy both differ from standard R4 organization. Plan legacy compatibility separately. |
| A14 | /paths/~1Appointment/get/responses/4XX and Core failure scenarios | Define unsupported, malformed, access and strict/lenient cases. Avoid accidental changes to shared errors. |
| A15 | /info/version, media types, guide matrices and release notes | Agree the target release, reconcile version declarations and publish capability availability. |

The resource schema is permissive: it has no restrictive resourceType enum and does not prohibit unspecified properties. The finding is inaccurate modelling/documentation and potential generated-client impact, not a demonstrated rejection of the new example. If using oneOf, constrain resourceType in each branch to avoid ambiguous matches between permissive schemas.

SearchBundleAppt is referenced by the JSON and XML 200 responses for GET /Appointment in the reviewed proposal. The existing default JSON example also needs an accurate self link containing the actual patient search and a valid participant reference.

## Recommended parameter fragments

These are proposed definitions, not applied edits. Add descriptions specifying prerequisites, error policy and traversal limits before publication.

~~~yaml
AppointmentInclude_QParam:
  name: _include
  in: query
  required: false
  style: form
  explode: true
  schema:
    type: array
    items:
      type: string
      enum:
        - Appointment:slot
        - Appointment:based-on
AppointmentIncludeIterate_QParam:
  name: _include:iterate
  in: query
  required: false
  style: form
  explode: true
  schema:
    type: array
    items:
      type: string
      enum:
        - Slot:schedule
        - Schedule:actor:Practitioner
        - Schedule:actor:PractitionerRole
        - Schedule:actor:HealthcareService
        - HealthcareService:organization
        - HealthcareService:location
~~~

Repeat keys rather than comma-joining values. Advertise supported expression values in Appointment.searchInclude; separately document which require iterate and the traversal bounds. The metadata value list alone does not specify a complete traversal policy. [FHIR search](https://hl7.org/fhir/R4/search.html#inclusion), [CapabilityStatement](https://hl7.org/fhir/R4/capabilitystatement-definitions.html#CapabilityStatement.rest.resource.searchInclude).

An enum can cause a gateway to reject unsupported values before the receiver sees them. Align validation with the chosen lenient/strict policy; do not promise ignored values while enforcing a contradictory gateway rule.

## Acceptance-test matrix

| Test | Expected evidence |
|---|---|
| No include | Existing appointment matching and default response semantics preserved. |
| Direct Slot | Matched Appointment plus referenced Slot(s); booked Slots may be busy, not only free. |
| Direct ServiceRequest | Only linked and authorised referrals included. |
| Full graph | Slot → Schedule → HealthcareService → Location/Organization uses iterate; no implicit recursion without it. |
| Practitioner branches | Test both typed Schedule actor branches; PractitionerRole does not imply further Practitioner traversal. |
| Shared context | Two appointments sharing a resource do not duplicate that entry within a page. |
| Empty results | Successful empty searchset; no unrelated resources included. |
| Absent reference/target | Matching appointment retained; absent context omitted according to policy. |
| Unauthorised referral | No disclosure through inclusion; documented omission/error policy applied. |
| External/identifier-only references | Defined resolution policy; no unrestricted remote fetching. |
| Unsupported valid expression | Agreed lenient handling and accurate self link; strict behaviour if supported. |
| Malformed expression/modifier | Agreed OperationOutcome/error; distinct from a supported include with no data. |
| Missing patient criteria | Includes alone cannot enumerate appointments. |
| Repeated keys through proxy | Every include/iterate value retained without comma-joining, dropping or double encoding. |
| Totals and paging | Total counts matches; page context is sufficient; paging preserves effective search where offered. |
| Mixed types and identity | Correct search.mode; fullUrl resolves intended links; no entry-order dependency. |
| Metadata | Advertised values match receiver implementation. |
| JSON/XML | Equivalent behaviour for advertised representations. |
| Performance | Largest supported graph stays within agreed limits and Core processing requirements. |
| Lifecycle | Searchset never submitted as Appointment update; read-before-write safeguards remain. |

Tests above are recommended acceptance criteria. They have not been executed against a live BaRS deployment. Full FHIR/profile validation of final examples and specification validation remain necessary.

## Source snapshots

Sources were downloaded from repository main on 16 September 2026. SHA-256 hashes follow to make the reviewed content identifiable even if main changes.

- uplift-report.md: a6bc7bca4a8985812a6c33d4c92c4b3e3e734815655d2f1cc3e33dc406f4bea9

- baseline.json: b27ee12eb40f052943a421df349c596a80c9804a7a25452e7aa5d176ff7f5f6f

- proposed.json: 9fae899b5006ebac7b75e9ad678c81a0e9a9e675d07df2640acf590624886cb3
