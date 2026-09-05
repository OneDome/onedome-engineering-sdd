# Sources — DEAL-2097

Recorded from intake. No analysis.

## Repositories named by user

- `hub-service`
- `integration-service`

## Nuclino

- https://app.nuclino.com/OneDome/Reactor/Phase-5-Affordability-b1d4a2b4-3b40-4678-b4b3-63349b8275c8
  - title: Phase 5: Client and Affordability tabs
  - item id: `b1d4a2b4-3b40-4678-b4b3-63349b8275c8`

## Jira attachments

- `Sourcing API.pdf` (Jira attachment id `68422`)
- SharePoint copy linked on the task: https://onedome.sharepoint.com/sites/Platform/Shared%20Documents/Sourcing%20API.pdf

## Extra resources named by user

- https://onedome.atlassian.net/wiki/spaces/Engineerin/pages/581763075/Hub+Service#Mortgage-Sourcing
  - Confluence page id `581763075`, title: Hub Service
- `hub-service/build/generated/sources/cxf/t7t/co/movedin/twenty7tec` — generated Twenty7tec CXF library

## Human Review (this session)

- Mortgage Sourcing is one UI and one functionality. There is no separate CRM vs ODP sourcing. CRM shows it as an iframe.
- Fact Find mapping, mandatory flags, and employment extras: developer applies the mapping workbook and Twenty7tec SOAP.
- EoR affordability: the report may include affordability results or omit them.
- Frontend receives only SOAP Applicant fields used for Twenty7tec. No invented contact email/phone/address.
- Applicant cardinality follows Twenty7tec `SourceInput`: two full Applicants plus name-only AdditionalApplicants.

## Linked from Nuclino (not separately named at intake)

- Field mapping spreadsheet: https://onedome-my.sharepoint.com/:x:/p/svitlana_taran/IQA0tABUk0zYT73-nh-jnqqEATQuQsAMuWPAo3P8ShfB87k?e=l0uWG1
