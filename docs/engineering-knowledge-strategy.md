# OneDome SDD and Engineering Knowledge

Management view of the strategy. Technical workflow: [README](../README.md).

Confluence: [SDD — engineering knowledge strategy](https://onedome.atlassian.net/wiki/spaces/Engineerin/pages/768933915/SDD+engineering+knowledge+strategy).

We are not only using AI to implement individual tasks. We are building reusable OneDome engineering knowledge.

Today, developers — especially new team members — can spend significant time finding context and understanding where a change belongs. Each completed task should contribute useful reusable knowledge, not only a feature or a fix.

OneDome contains many different business and technical contexts. We do not need to define all of them up front. As engineering tasks are completed, recurring areas of knowledge will naturally become visible. Useful knowledge is accumulated around those areas, and some may eventually become mature bounded contexts. When a context contains enough reusable knowledge, a specialized agent can use it as a starting point for future work while still verifying current behaviour against the source code. We do not create an agent just because we can name a business area.

A bounded context is not the same as a Git repository. A business context may span several services, repositories, integrations, APIs and documentation. One service may also participate in several contexts. Future agents are organized around that useful knowledge boundary, not around `hub-service` or `integration-service`.

The expected result is less repeated investigation, easier onboarding, knowledge that is not trapped in a few people's heads, better AI context, and more time spent implementing.

```mermaid
flowchart TB
    subgraph PROBLEM["1. Problem today"]
        direction LR
        P1[New task] --> P2[Find context]
        P2 --> P3[Search code and docs]
        P3 --> P4[Ask other developers]
        P4 --> P5[Finally implement]
    end

    subgraph FLOW["2. How SDD works"]
        direction TB
        F1[Jira task] --> F2[AI-assisted investigation]
        F2 --> F3[Specification — WHAT / WHY]
        F3 --> F4[Human review]
        F4 --> F5[Technical plan — HOW]
        F5 --> F6[Human review]
        F6 --> F7[Development and tests]
        F7 --> F8[Completed task]
    end

    F8 --> FEAT[Feature / fix]
    F8 --> KNOW[Reusable knowledge]

    KNOW --> KB["3. OneDome Engineering Knowledge Base<br/>Evolving bounded contexts — examples, not a catalogue<br/>Fact Find · DealRoom · Mortgage · Customer · Broker · …<br/>Contexts emerge from real tasks — not defined up front"]

    KB --> AG["4. Specialized agents emerge later<br/>only when a context has enough reusable knowledge<br/>Context A · Context B · Context C · …<br/>Start from that knowledge, then verify against current code"]

    AG --> OUT["5. Value that accumulates<br/>Less investigation · Faster onboarding · Knowledge retention<br/>Less dependency on individuals · More consistent development<br/>Better AI context · Higher velocity"]

    OUT -->|"next task starts with more context"| F1

    subgraph TIME["Expected direction if used consistently — not a guarantee"]
        direction LR
        T0[Today<br/>small knowledge base] --> T3[~3 months<br/>known areas get easier]
        T3 --> T6[~6 months<br/>first mature contexts]
        T6 --> TL[Longer term<br/>agents only where knowledge is mature]
    end

    classDef problem fill:#FFEBE6,stroke:#BF2600
    classDef kb fill:#DEEBFF,stroke:#0747A6,stroke-width:2px
    classDef future fill:#FFFAE6,stroke:#FF991F
    classDef result fill:#E3FCEF,stroke:#006644
    class PROBLEM problem
    class KB kb
    class AG future
    class OUT result
```
