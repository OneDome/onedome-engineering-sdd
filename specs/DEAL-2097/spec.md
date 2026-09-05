---
schema_version: 1
jira_key: DEAL-2097
jira_title: OD Mortgage Sourcing Phase 5
status: approved
created_at: 2026-09-05T16:30:00+03:00
updated_at: 2026-09-05T17:19:00+03:00
approval:
  status: approved
  approved_by: oksanaodocuk
  approved_at: 2026-09-05T17:19:00+03:00
---

# Specification: `DEAL-2097`

Approved **backend** business specification. WHAT and WHY only. Do not invent behaviour. Do not describe HOW. Do not specify frontend/UI.

Human Approval recorded from chat (`I approve this spec`) on 2026-09-05. Job title is irrelevant.

## Distinctions

| Kind | Where it belongs |
| --- | --- |
| Explicitly requested future behaviour | Expected Behaviour, Business Rules, Acceptance Criteria |
| Verified current behaviour | Current Behaviour |
| Assumptions | Assumptions (never copied into requirements) |
| Unknowns | Open Questions |

## Business Context

Mortgage Sourcing must send Twenty7tec the same Client and Affordability information Twenty7tec uses for product search and recommendation documents.

There is one Mortgage Sourcing backend and one sourcing functionality. CRM embeds the existing sourcing UI in an iframe; it is not a second product. Nuclino’s “CRM not ODP” is superseded by Human Review.

Evidence: Nuclino [Phase 5: Client and Affordability tabs](https://app.nuclino.com/OneDome/Reactor/Phase-5-Affordability-b1d4a2b4-3b40-4678-b4b3-63349b8275c8); Human Review. Locator: [DEAL-2097](https://onedome.atlassian.net/browse/DEAL-2097).

## Problem

Sourcing search today sends only a thin applicant slice (identity, employment status, basic salary, lender). It does not carry full Twenty7tec Client data (including answer-dependent extras) or an opt-in affordability check with results on the product set.

## Current Behaviour

`hub-service` exposes Mortgage Sourcing under `/api/hub/sourcing` (prefill by opportunity, search, product details, sessions, EoR and related documents). Search calls Twenty7tec `runSource`. `affordabilitySource`, `affordabilitySourceResults`, and `generateEoADocument` exist on the generated client and are not used.

Applicant payload today (`MortgageSourcingInputMapper.buildApplicant`): title, names, gender, date of birth, employment status, basic annual salary, applicant type. Applicant 1/2 map to Twenty7tec `Applicant1`/`Applicant2`. Further people map to `AdditionalApplicant` (title, first name, last name only). Outgoings, credit history, and applicant affordability extras are not set.

CRM prefill: `integration-service` `GET /opportunities/{opportunityUuid}/sourcing` returns application FF mortgage/property fields and applicants with title, names, DOB, residential status, employment status, basic annual income, current lender, main-applicant flag. Hub `CrmToTwenty7TecMapper` maps that set into prefill. Contact, outgoings, and credit history are not in that payload.

Twenty7tec CXF (`hub-service/build/generated/sources/cxf/t7t/co/movedin/twenty7tec`): `SourceInput` holds `Applicant1`, `Applicant2`, `AdditionalApplicants`, `AffordabilityDetails`, `AffordabilityOptions`. Full `Applicant` has no email, phone, or correspondence address. Product row type `Results` has `AffordabilityScore`. Filter enum `AffordabilityStatusFilterEnum`: ShowAll, AllAccept, NoDeclines, AllDeclines. Period enum includes `FiveYear`. `GenerateEoRDocument` takes `EoRInput` with nested `SourceInput`. `GenerateEoADocument` is a separate document type.

## Expected Behaviour

Backend only. Same sourcing APIs for every consumer (including CRM iframe).

1. Accept and return Client data only for SOAP fields that Twenty7tec `Applicant` (and its nested types) uses on search. Do not send the frontend email, phone, or correspondence address: they are not on `Applicant`.
2. Group those SOAP fields to Nuclino Client areas as in **SOAP → Client groups** below. Nested types that depend on employment status (employed / self-employed / contractor) are the answer-dependent extras.
3. Prefill from Fact Find every such field the mapping workbook allows. Applying that workbook is a developer task, not a spec gap.
4. Fields mandatory on Twenty7tec Client must be supplied before search is treated as complete. Which FF cells fill which SOAP fields is a developer task against the mapping workbook.
5. When **Include an Affordability Check** is not set (default), search results do not include affordability outcomes.
6. When it is set, search results include per-product affordability outcomes with statuses: Accept, Partial accept, Declined, No information available; and support filters: No Filtering, All Accept, No Declines, All Decline (Twenty7tec: ShowAll, AllAccept, NoDeclines, AllDeclines).
7. The sourcing report (EoR) may include affordability results and may omit them. Affordability on the report is not mandatory.

Screens, tabs, and checkbox placement are frontend and out of scope.

### SOAP → Client groups

Source: generated `Applicant` and nested types. Consumer payloads (prefill and search) include only these fields.

**Personal Details**

- Title, FirstName, MiddleName, LastName, Gender, DateOfBirth, AgeNextBirthday
- ApplicantType, ResidentialStatus, Nationality, CountryOfResidence, MaritalStatus
- Smoker, CriminalConvictions, RetirementAge, PremierCustomer
- NumberOfChildDependents, NumberOfAdultDependents, Dependents[].DateOfBirth

**Contact Details**

- No fields on SOAP `Applicant`. Do not invent contact fields for the frontend.

**Employment, Financial & Income Details**

- EmploymentStatus; CurrentEmploymentYears; CurrentEmploymentMonths
- EmployedDetails: BasicAnnualSalaryGross, MonthsInCurrentEmployment, MonthsInContinuousEmployment, NetMonthlyIncome
- SelfEmployedDetails: years of accounts, net profits (3 years), months trading, ltd director salary / retained profits / dividends (3 years), PercentageShareHolding
- AdditionalAnnualIncome (earned/other income, allowances, pension, bonuses, commission, rental, tax credits, etc.)
- AffordabilityAdditionalIncome, AffordabilityContractor, AffordabilityBenefits (used when those employment answers apply)
- FinancialDetails.CurrentAccountProviderCode
- ExistingMortgage: OutstandingBalance, CurrentMonthlyPayment, LenderCode, TimeWithCurrentLender, EarlyRepaymentChargeAmount, MonthsRemainingOnMortgage, PaymentTiers

**Outgoings**

- Outgoings: SecuredLoan, UnSecuredLoan, OverDraft, CreditStoreCards, HirePurchase, ChildMaintenance, ChildEducation, ExistingMortgage (each Outgoing: MonthlyCommitment, OutstandingBalance, NumberOfMonthLeftToPay, ClearedOnCompletion)
- Outgoings.MonthlyExpenses (household, utilities, living costs — including HomePhone/MobilePhone as **expense amounts**, not contact numbers)
- AffordabilityCommitments (type, amounts, months remaining)
- AffordabilityMonthlyOutgoings (ground rent, service charge, insurance, commuting, etc.)

**Credit History**

- CreditHistoryDetails lists: Arrears, Bankruptcies, CCJs, Defaults, IVAs, PaydayLoans, DebtManagementPlans, Repossessions (each item carries amount/dates as in the SOAP type, e.g. CCJ Amount, DateRegistered, DateSatisfied)

**Additional applicants (not a full Client record)**

- AdditionalApplicant: Title, FirstName, LastName only (Twenty7tec `SourceInput.AdditionalApplicants`)

## User / System Flows

### Flow 1 — Prefill Client data

1. **Given** an opportunity with Fact Find data
2. **When** sourcing prefill is requested
3. **Then** the backend returns only SOAP Client fields (SOAP → Client groups) that the FF mapping can fill, for the applicants selected by BR-002 / BR-003. No contact email/phone/address.

### Flow 2 — Search without affordability

1. **Given** Client data on the sourcing request and Include an Affordability Check unset
2. **When** search runs
3. **Then** Twenty7tec search uses that Client data and the product set has no affordability outcomes

### Flow 3 — Search with affordability

1. **Given** Include an Affordability Check is set
2. **When** search runs
3. **Then** the product set includes affordability outcomes and the four status filters can be applied

## Business Rules

- **BR-001:** One Mortgage Sourcing backend. CRM iframe and other consumers use the same behaviour. Not a CRM-only fork.
- **BR-002:** One applicant: one full `Applicant` (`Applicant1`). Two or more: two full records (`Applicant1`, `Applicant2`).
- **BR-003:** Follow Twenty7tec `SourceInput`: at most two full `Applicant` objects. Further people are `AdditionalApplicant` (title, first name, last name only). When more than two people exist on the opportunity, the two full slots are the main applicant and, if possible, the applicant added second after the main; any others go to AdditionalApplicants.
- **BR-004:** Client data, including employment-dependent nested types, must match SOAP `Applicant`. Twenty7tec-mandatory Client questions are mandatory for search completeness. FF cell mapping is a developer task against the mapping workbook.
- **BR-005:** Prefill from Fact Find every SOAP Client field the mapping workbook allows.
- **BR-006:** Include an Affordability Check is off by default. Affordability outcomes and the four filters apply only when it was set for that search.
- **BR-007:** Affordability filters are exactly: No Filtering, All Accept, No Declines, All Decline.
- **BR-008:** The backend exposes to the frontend only fields that will be sent to Twenty7tec (SOAP → Client groups). No extra Client fields.
- **BR-009:** The sourcing report (EoR) may include affordability results or omit them. Neither presence nor absence of affordability on the report is a defect.

## Acceptance Criteria

- **AC-001:** Given sourcing search with Client data for the selected applicants, When Twenty7tec search runs, Then the SOAP → Client group fields present on the request are sent, including employment-dependent nested types that were supplied. Email/phone/address are not on the contract.
- **AC-002:** Given Fact Find values the mapping workbook covers, When prefill is requested, Then those SOAP Client fields are populated and no non-SOAP Client fields are returned.
- **AC-003:** Given a Twenty7tec-mandatory Client field is missing, When search is requested, Then the search is not accepted as complete.
- **AC-004:** Given Include an Affordability Check is unset, When search returns products, Then affordability outcomes are absent.
- **AC-005:** Given Include an Affordability Check is set, When search returns products, Then each product has an affordability outcome using the four statuses, and the four filters can be applied.
- **AC-006:** Given the same sourcing request, When it originates from CRM iframe or from the non-iframe consumer, Then Client and Affordability backend behaviour is the same (BR-001).
- **AC-007:** Given three or more people on the opportunity, When search runs, Then at most two full `Applicant` records are sent and any others are AdditionalApplicant name-only (BR-003).
- **AC-008:** Given an EoR is generated, When the document is returned, Then the request is accepted whether the report includes affordability results or not (BR-009).

## Edge Cases

- One vs two vs three-or-more people: two full SOAP applicants max; extras are name-only (BR-003).
- Employment-dependent nested types: send only those that apply to the chosen EmploymentStatus.
- Nuclino Contact Details: no SOAP fields; not on the frontend contract (BR-008).
- MonthlyExpenses HomePhone/MobilePhone are money amounts, not contact numbers.
- EoR with affordability and EoR without affordability are both valid (BR-009).

## Assumptions

- Nested Twenty7tec types that appear only for some employment statuses are the “extra questions depending on user’s answer”.
- Affordability numbers and statuses come from Twenty7tec when the check is included. Nuclino does not define calculation rules.
- Fact Find → SOAP mapping and Twenty7tec mandatory flags are applied by the developer from the mapping workbook and Twenty7tec docs; they are not open business questions.

## Dependencies

- Mapping workbook (developer task): https://onedome-my.sharepoint.com/:x:/p/svitlana_taran/IQA0tABUk0zYT73-nh-jnqqEATQuQsAMuWPAo3P8ShfB87k?e=l0uWG1
- `Sourcing API.pdf` / generated CXF for Twenty7tec applicant cardinality and field set.

## Potentially Affected Systems

| System / repository id | Why it might be affected | Confidence |
| --- | --- | --- |
| hub-service | Sourcing APIs, Twenty7tec `SourceInput` / search / EoR. Client extras and affordability unused today. | high |
| integration-service | Opportunity sourcing details used for prefill; Client groups need more FF fields than today. | medium |

Frontend sourcing UI is a **consumer** of this backend (CRM hosts it in an iframe). It is not an SDD delivery target.

## Out of Scope

- Frontend / UI (screens, client apps, styling, tabs, checkbox placement). SDD delivers backend only.
- Changing Twenty7tec itself.
- A second CRM-only or ODP-only sourcing backend (BR-001).
- Inventing Client fields that Twenty7tec SOAP `Applicant` does not define (including contact email/phone/address).
- Requiring the sourcing report to always include (or always omit) affordability results.

## Open Questions

None.

## Evidence / References

### Jira

- [DEAL-2097](https://onedome.atlassian.net/browse/DEAL-2097) — title “OD Mortgage Sourcing Phase 5”; empty description; locator only
- Attachment `Sourcing API.pdf` (id `68422`) — not readable in this session

### Confluence

- [Hub Service](https://onedome.atlassian.net/wiki/spaces/Engineerin/pages/581763075/Hub+Service) — page id `581763075`; sourcing APIs and Twenty7Tec listed; Mortgage Sourcing flow section empty

### Nuclino

- [Phase 5: Client and Affordability tabs](https://app.nuclino.com/OneDome/Reactor/Phase-5-Affordability-b1d4a2b4-3b40-4678-b4b3-63349b8275c8) — item `b1d4a2b4-3b40-4678-b4b3-63349b8275c8`
- Mapping workbook (unread): https://onedome-my.sharepoint.com/:x:/p/svitlana_taran/IQA0tABUk0zYT73-nh-jnqqEATQuQsAMuWPAo3P8ShfB87k?e=l0uWG1

### Source code

- `hub-service` `src/main/java/co/movedin/hub/controller/MortgageSourcingController.java`
- `hub-service` `src/main/java/co/movedin/hub/service/sourcing/MortgageSourcingService.java`
- `hub-service` `src/main/java/co/movedin/hub/service/sourcing/MortgageSourcingInputMapper.java`
- `hub-service` `src/main/java/co/movedin/hub/service/sourcing/CrmToTwenty7TecMapper.java`
- `hub-service` `src/main/java/co/movedin/hub/model/sourcing/MortgageSourcingApplicantDetails.java`
- `hub-service/build/generated/sources/cxf/t7t/co/movedin/twenty7tec/` — `ISourcing`, `SourceInput`, `Applicant`, `AdditionalApplicant`, `Outgoings`, `CreditHistoryDetails`, `AffordabilityStatusFilterEnum`, `AffordabilityResults`, `EoRInput`, `EoAInput`, `Results`
- `integration-service` `src/main/java/co/movedin/integration/integrations/msdc/controller/MsDcController.java` — `GET /opportunities/{opportunityUuid}/sourcing`
- `integration-service` `src/main/java/co/movedin/integration/integrations/msdc/client/sourcing/CrmSourcingQuery.java`
- `integration-service` `src/main/java/co/movedin/integration/integrations/msdc/client/sourcing/CrmOpportunitySourcingDetailsMapper.java`

### Previous specs

- This file revised from the earlier DEAL-2097 draft (UI-heavy). Current behaviour re-verified against named backend repos.

### Human Review

- 2026-09-05: one sourcing product; CRM is an iframe consumer; no CRM vs ODP split (BR-001).
- 2026-09-05: Fact Find mapping and Twenty7tec-mandatory flags are a developer task against the mapping workbook. Frontend contract is SOAP Applicant fields only (no invented contact). Applicant cardinality follows Twenty7tec `SourceInput`.
- 2026-09-05: The sourcing report may include affordability results or omit them (BR-009).
- 2026-09-05: Human Approval in chat (`I approve this spec`); recorded as `oksanaodocuk`.

### Engineering knowledge

- `specs/DEAL-2097/knowledge/sources.md` — intake only
