---
schema_version: 1
jira_key: DEAL-2097
title: OD Mortgage Sourcing Phase 5
status: approved
spec_status_required: approved
---

# Plan: `DEAL-2097`

HOW only. Must implement the approved specification. Do not change WHAT/WHY here.

## Gate

- Specification path: `specs/DEAL-2097/spec.md`
- Specification is approved in `task-state.yaml` (`approved_by: oksanaodocuk`, `method: chat`).
- Plan Human Approval recorded from chat (`I approve this plan`) on 2026-09-05.

## Approach

Keep the existing Hub sourcing surface (`/api/hub/sourcing`). Do not add a CRM-only backend. CRM iframe and other consumers already call the same Hub APIs (BR-001).

Work in three layers, all on the current search/prefill/EoR path:

1. **Fact Find → SOAP Client fields.** Developer applies the mapping workbook and Twenty7tec SOAP types. Integration-service fetches the mapped Dataverse attributes. Hub maps them onto SOAP `Applicant` (and nested types) and returns the same fields on prefill. Do not add email, phone, or correspondence address to the frontend contract (BR-008).
2. **Applicant cardinality.** Replace index-only `Applicant1`/`Applicant2` assignment with BR-003 selection, then map extras to `AdditionalApplicant` (title, first name, last name only). `ApplicantBuilder` already covers the SOAP `Applicant` setters; `MortgageSourcingInputMapper.buildApplicant` currently uses a thin subset.
3. **Opt-in affordability on search.** Default off. When unset: `runSource` only; rows have no affordability outcomes. When set: call unused CXF `ISourcing.affordabilitySource(SourceInput)` (returns `AffordabilityId`), then `affordabilitySourceResults` with that id; join results onto product rows by `LenderCode`. Apply the four status filters Hub-side. EoR still uses `generateEoRDocument` with the same `SourceInput`; do not fail if the PDF omits affordability (BR-009). Do not call `generateEoADocument`.

Do not invent FF logical names in this plan. The workbook is the mapping source. Confluence [Hub Service](https://onedome.atlassian.net/wiki/spaces/Engineerin/pages/581763075/Hub+Service) lists sourcing endpoints; the Mortgage Sourcing flow section is empty. Code is implementation truth.

## Per-repository changes

### `integration-service`

- **Path:** `integration-service`
- **Change:**
  - Extend `CrmSourcingQuery.OPP_SOURCING_DETAILS_URL_TMPL` `$select` / `$expand` with every Personal Fact Find and Application Fact Find attribute the mapping workbook maps onto SOAP Client groups. Do not select applicant email/phone/address for sourcing.
  - Extend `CrmOpportunitySourcingDetailsDto.ApplicantContactDto` (and parent DTO if workbook maps application-level Client fields) with those attributes only. Keep `@GenerateSchema` and regenerate Hub’s `co.movedin.shared.integration` copies with the existing schema pipeline.
  - Map the new fields in `CrmOpportunitySourcingDetailsMapper` (including option-set labels via `CrmOptionSets` where CRM stores codes). Preserve `mainApplicant` (opportunity `customerid_contact` vs PFF contact) and list order of `inc_opportunity_inc_personalfactfind_Opportunity` — Hub uses both for BR-003.
  - Leave `GET /opportunities/{opportunityUuid}/sourcing` as the prefill source. No new endpoint.
- **Code evidence inspected:** `src/main/java/co/movedin/integration/integrations/msdc/client/sourcing/CrmSourcingQuery.java`; `CrmOpportunitySourcingDetailsMapper.java`; `src/main/java/co/movedin/integration/integrations/msdc/model/request/CrmOpportunitySourcingDetailsDto.java`; `src/main/java/co/movedin/integration/integrations/msdc/controller/MsDcController.java`.
- **Branch:** null

### `hub-service`

- **Path:** `hub-service`
- **Change:**
  - Expand `MortgageSourcingApplicantDetails` to the SOAP → Client groups in the spec (nested types for employed/self-employed/income, outgoings, credit history, dependents). No contact email/phone/address. Regenerated `integration-service` `shared/hub` copies if that pipeline includes this DTO.
  - Map CRM prefill in `CrmToTwenty7TecMapper.getApplicantDetails` from the expanded `ApplicantContactDto`.
  - Map Hub JSON → SOAP in `MortgageSourcingInputMapper.buildApplicant` via existing `ApplicantBuilder`. Send employment-dependent nested types only when they apply to `EmploymentStatus`.
  - Change `addApplicants`: `Applicant1` = applicant with `mainApplicant==true`, else first in list; `Applicant2` = next remaining person in list order when two or more people exist; further people → `AdditionalApplicant` name-only. Keep today’s `ExistingMortgage.LenderCode` on `Applicant1` when `currentLender` is present.
  - Before `runSource` / `productDetails` / EoR `SourceInput` build: reject incomplete search when Twenty7tec-mandatory Client fields (from workbook + SOAP docs) are missing. Use the existing `ExceptionWithErrorCode` pattern (`MortgageSourcingService`).
  - Add `includeAffordabilityCheck` (default unset/false) on the sourcing request (`PrefillingOpportunityData` or `MortgageSourcingInput` — one place, Jackson `NON_NULL`). When false/absent: do not call `affordabilitySource` / `affordabilitySourceResults`; do not put affordability outcomes on rows (AC-004).
  - When true: build the same `SourceInput` as search (Client fields + `AffordabilityOptions` / `AffordabilityDetails` if the SOAP type requires them for this call). Call `t7tPort.affordabilitySource(licenseKey, sourceInput, referenceData)`. Read `AffordabilityId` from `AffordabilitySourceOutput`. Call `t7tPort.affordabilitySourceResults` with `AffordabilitySourceInput` (companyId, siteId, that id). Join `AffordabilitySourceResult` onto `MortgageSourcingOutput.Row` by `LenderCode` (SOAP result has lender, not product code). Map `Affordable` / `AffordabilityStatus` / `AffordabilityMessage` / `MaxBorrowing` to the four Nuclino statuses using Twenty7tec docs — do not invent score integers. Products whose lender has no result: **No information available**.
  - Four filters (BR-007): No Filtering / All Accept / No Declines / All Decline ↔ `AffordabilityStatusFilterEnum`. CXF `AffordabilityFilters` is on `EoAInput` only — apply Hub-side on joined row outcomes. Add the filter field on `SourcingFiltersDto`. Ignore it when the check is off.
  - `generateEoRDocument`: keep `buildEorInput` → `generateEoRDocument`. If the search had the check on, pass the same `SourceInput` (including affordability SOAP fields). Do not add mandatory EoR affordability columns. Accept success whether T7T includes affordability in the PDF or not (AC-008). Do not call `generateEoADocument`.
  - Same mapper path for product details, pin, send-to-deal, and stored input JSON so affordability and Client fields round-trip. Dynamo `MortgageSourcingRow.payload` is JSON; extra row fields are backward compatible.
- **Code evidence inspected:** `src/main/java/co/movedin/hub/controller/MortgageSourcingController.java`; `service/sourcing/MortgageSourcingService.java`; `MortgageSourcingInputMapper.java`; `ApplicantBuilder.java`; `CrmToTwenty7TecMapper.java`; `MortgageSourcingOutputMapper.java`; `filters/SourcingFiltersDto.java`; `filters/FiltersMapper.java`; `model/sourcing/MortgageSourcingApplicantDetails.java`; `MortgageSourcingOutput.java`; `PrefillingOpportunityData.java`; `src/main/java/co/movedin/shared/integration/ApplicantContactDto.java`; CXF `ISourcing.affordabilitySource` / `affordabilitySourceResults`, `AffordabilitySourceOutput`, `AffordabilitySourceResult`, `SourceInput`, `Applicant`, `AdditionalApplicant`, `AffordabilityOptions`, `AffordabilityDetails`, `AffordabilityStatusFilterEnum`, `AffordabilityFilters`, `Results`, `EoRInput`, `EoAInput`; tests `MortgageSourcingInputMapperRemortgageDetailsTest.java`, `MortgageSourcingInputMapperEorTest.java`.
- **Branch:** null

No other catalog repositories are required. `api-edge` / `movedin-edge` already proxy `/api/hub/**`. Frontend is a consumer, not an SDD delivery target.

## Sequencing

1. Developer extracts FF attribute → SOAP field → mandatory flag from the mapping workbook and Sourcing API.pdf (including Affordable/AffordabilityStatus → four Nuclino statuses).
2. `integration-service`: query + DTO + mapper; regenerate Hub shared integration classes.
3. `hub-service`: applicant DTO + CRM mapper + SOAP mapper + BR-003 slotting + mandatory check.
4. `hub-service`: `includeAffordabilityCheck`; when on, `affordabilitySource` then `affordabilitySourceResults`; join by `LenderCode`; four Hub-side filters.
5. `hub-service`: EoR uses the same `SourceInput`; no EoA; no failure if PDF omits affordability.
6. Unit tests in Hub (mapper/slotting/mandatory/affordability on/off/filters). Integration-service mapper tests for new FF fields. No frontend work.

Contract order: integration-service publishes a richer `GET .../sourcing` payload; Hub consumes it for prefill; Hub search JSON grows in lockstep so the iframe consumer can POST the same Client fields back.

## Risks

| Risk | Impact | Mitigation |
| --- | --- | --- |
| Mapping workbook / PDF unread in this session | Wrong or missing FF attributes | Developer applies workbook + SOAP; do not guess Dataverse names in code review without the sheet |
| `Affordable` / `AffordabilityStatus` are booleans, Nuclino has four statuses | Wrong Accept / Partial / Decline / No information | Use Sourcing API.pdf for the mapping; lender with no `AffordabilitySourceResult` → No information available |
| Join is by `LenderCode`, not product code | Several products of one lender share one outcome | Join on lender; do not invent a product-level SOAP key |
| Extra SOAP round-trip (`affordabilitySource` + `affordabilitySourceResults`) | Search latency | Call only when the check is on; keep `runSource` for the product set |
| Schema copies (`shared/integration`, `shared/hub`) drift | Prefill/search JSON mismatch | Regenerate generated classes in the same change as DTO edits |

## Test plan

- **AC-001:** Mapper test: populated SOAP → Client group fields on `Applicant1`/`Applicant2`; no email/phone/address properties on Hub applicant JSON; employment nested types present only when status requires them.
- **AC-002:** Prefill path: `CrmToTwenty7TecMapper` fills workbook-mapped fields from `ApplicantContactDto`; unmapped/non-SOAP fields absent.
- **AC-003:** Search with a documented mandatory Client field missing → `ExceptionWithErrorCode` (or existing Hub error) and no `runSource` call (mock `ISourcing`).
- **AC-004:** `includeAffordabilityCheck` unset → `affordabilitySource` not called; rows have no affordability outcome.
- **AC-005:** Check on → Hub calls `affordabilitySource` then `affordabilitySourceResults`; rows have one of the four statuses; each of the four filters reduces/keeps rows per enum meaning; T7T mock returns lender results.
- **AC-006:** No CRM-specific sourcing service; tests use the same `MortgageSourcingInputMapper` / `runSourcing` for any consumer payload.
- **AC-007:** Three applicants (main in the middle of the list) → `Applicant1` = main, `Applicant2` = next in list, third = `AdditionalApplicant` names only.
- **AC-008:** `buildEorInput` succeeds with and without affordability on `SourceInput`; `generateEoADocument` not invoked.

## Knowledge updates

```yaml
- path: knowledge/integrations/twenty7tec-sourcing-client-fields.md
  action: add
  notes: After implementation, record FF attribute → SOAP Applicant field and mandatory flags actually shipped (from workbook + code). Verify against hub-service mapper, not the workbook alone.
```

## Open Questions

None.
