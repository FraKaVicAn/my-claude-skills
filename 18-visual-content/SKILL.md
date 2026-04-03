---
name: visual-content
description: Auto-activating visual content skill — Mermaid diagrams (flowchart, sequence, ER, class, Gantt), PlantUML, architecture diagrams, mind maps, Chart.js, and data visualization
origin: jeremylongshore/claude-code-plugins-plus-skills/18-visual-content
tools: Read, Write, Edit, Bash, Grep
---

# Visual Content Skill

Automated diagram and visualization generation: Mermaid, PlantUML, architecture diagrams, data charts, network diagrams, and org charts — all in text-based, version-controllable formats.

## When to Activate

Activates automatically when you mention: Mermaid, diagram, flowchart, sequence diagram, ER diagram, class diagram, Gantt chart, PlantUML, architecture diagram, mind map, network diagram, org chart, Chart.js, Plotly, visualization, infographic, presentation.

## Covered Skills (25 total)

| Skill | What it does |
|-------|-------------|
| `mermaid-flowchart-creator` | Process and decision flowcharts |
| `mermaid-sequence-diagram` | Service interaction sequence diagrams |
| `mermaid-er-diagram` | Entity-relationship database diagrams |
| `mermaid-class-diagram` | UML class diagrams with relationships |
| `mermaid-state-diagram` | State machine diagrams |
| `mermaid-gantt-creator` | Project timeline Gantt charts |
| `plantuml-diagram-creator` | PlantUML component and deployment diagrams |
| `architecture-diagram-creator` | C4 model system architecture diagrams |
| `api-flow-diagram-creator` | API request/response flow diagrams |
| `network-diagram-generator` | Network topology diagrams |
| `org-chart-creator` | Organizational hierarchy charts |
| `mindmap-generator` | Topic mind maps |
| `user-journey-mapper` | User journey flow diagrams |
| `process-flow-generator` | Business process flow diagrams |
| `database-schema-visualizer` | Database schema as ER diagram |
| `chartjs-config-generator` | Chart.js configuration for web dashboards |
| `plotly-chart-creator` | Interactive Plotly charts |
| `data-visualization-helper` | Choose and configure the right chart type |
| `svg-icon-generator` | Simple SVG icon code |
| `technical-diagram-analyzer` | Analyze and improve existing diagrams |
| `infographic-outline-creator` | Infographic structure and content plan |
| `presentation-slide-outliner` | Slide deck structure and talking points |

## Mermaid Flowchart

```mermaid
flowchart TD
    A([User Request]) --> B{Authenticated?}
    B -- No --> C[Return 401]
    B -- Yes --> D{Rate Limited?}
    D -- Yes --> E[Return 429]
    D -- No --> F[Process Request]
    F --> G{Cache Hit?}
    G -- Yes --> H[Return Cached Response]
    G -- No --> I[Query Database]
    I --> J[Cache Response]
    J --> K([Return Response])
    H --> K
```

## Mermaid Sequence Diagram

```mermaid
sequenceDiagram
    actor User
    participant FE as Frontend
    participant API as API Gateway
    participant Auth as Auth Service
    participant DB as Database

    User->>FE: Login (email, password)
    FE->>API: POST /auth/login
    API->>Auth: Validate credentials
    Auth->>DB: SELECT user WHERE email=?
    DB-->>Auth: User record
    Auth-->>API: JWT token
    API-->>FE: {token, expiresAt}
    FE->>FE: Store token in memory
    FE-->>User: Redirect to dashboard
```

## Mermaid ER Diagram

```mermaid
erDiagram
    USER {
        uuid id PK
        string email UK
        string name
        timestamp created_at
    }
    ORDER {
        uuid id PK
        uuid user_id FK
        decimal total
        string status
        timestamp created_at
    }
    ORDER_ITEM {
        uuid id PK
        uuid order_id FK
        uuid product_id FK
        int quantity
        decimal unit_price
    }
    PRODUCT {
        uuid id PK
        string name
        decimal price
        int stock
    }

    USER ||--o{ ORDER : "places"
    ORDER ||--|{ ORDER_ITEM : "contains"
    PRODUCT ||--o{ ORDER_ITEM : "included in"
```

## Mermaid Gantt

```mermaid
gantt
    title Project Timeline Q2 2025
    dateFormat  YYYY-MM-DD
    section Design
        Requirements      :done, req, 2025-04-01, 7d
        UI Mockups        :done, ui, after req, 7d
        Design Review     :milestone, 2025-04-15, 0d
    section Development
        Backend API       :active, api, 2025-04-15, 21d
        Frontend          :fe, 2025-04-22, 14d
        Integration       :int, after api, 7d
    section Testing
        QA Testing        :qa, after int, 7d
        UAT               :uat, after qa, 5d
    section Launch
        Go Live           :milestone, 2025-05-26, 0d
```

## C4 Architecture (Mermaid)

```mermaid
C4Context
    title System Context — E-Commerce Platform
    Person(customer, "Customer", "Shops and places orders")
    System(ecommerce, "E-Commerce Platform", "Handles products, orders, payments")
    System_Ext(payment, "Stripe", "Payment processing")
    System_Ext(email, "SendGrid", "Email notifications")
    System_Ext(shipping, "ShipStation", "Order fulfillment")

    Rel(customer, ecommerce, "Uses", "HTTPS")
    Rel(ecommerce, payment, "Charges cards", "HTTPS/REST")
    Rel(ecommerce, email, "Sends emails", "HTTPS/REST")
    Rel(ecommerce, shipping, "Creates shipments", "HTTPS/REST")
```

## Links

- [Repository](https://github.com/jeremylongshore/claude-code-plugins-plus-skills)
- Category: `18-visual-content` (25 skills)
