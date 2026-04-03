---
name: enterprise-workflows
description: Auto-activating enterprise workflow skill — user stories, Jira/Linear/GitHub Issues, sprint planning, OKRs, KPI dashboards, risk assessment, SLA monitoring, roadmaps, and change management
origin: jeremylongshore/claude-code-plugins-plus-skills/20-enterprise-workflows
tools: Read, Write, Edit, Bash, Grep
---

# Enterprise Workflows Skill

Automated assistance for enterprise project management: agile ceremonies, issue tracking, OKRs, governance, compliance checklists, SLA monitoring, and executive communications.

## When to Activate

Activates automatically when you mention: user story, acceptance criteria, Jira ticket, Linear issue, GitHub issue, sprint planning, backlog grooming, OKR, KPI dashboard, roadmap, risk assessment, SLA, stakeholder communication, change request, audit trail, executive summary, status report, Confluence, post-mortem.

## Covered Skills (26 total)

| Skill | What it does |
|-------|-------------|
| `user-story-generator` | User stories with As/I want/So that format |
| `acceptance-criteria-creator` | Given/When/Then acceptance criteria |
| `definition-of-done-generator` | Team Definition of Done checklist |
| `sprint-planning-helper` | Sprint goal and capacity planning |
| `backlog-grooming-assistant` | Story sizing, prioritization, splitting |
| `jira-ticket-generator` | JIRA issue JSON with fields and subtasks |
| `jira-workflow-creator` | JIRA workflow state transitions |
| `github-issue-creator` | GitHub issue with labels and milestones |
| `linear-issue-generator` | Linear issue with team and priority |
| `gitlab-epic-creator` | GitLab epic with child issues |
| `github-project-setup` | GitHub Projects board configuration |
| `confluence-page-generator` | Confluence page with structured content |
| `executive-summary-creator` | C-suite executive summary format |
| `status-report-generator` | RAG status reports with metrics |
| `stakeholder-communication-template` | Stakeholder update communications |
| `okr-tracker-creator` | OKR framework with key results and check-ins |
| `kpi-dashboard-template` | KPI dashboard structure and metrics |
| `roadmap-generator` | Product/technical roadmap structure |
| `risk-assessment-creator` | Risk register with impact/probability matrix |
| `impact-analysis-helper` | Change impact analysis template |
| `change-request-generator` | Change request documentation |
| `governance-checklist-generator` | Compliance and governance checklists |
| `audit-trail-helper` | Audit log schema and reporting |
| `sla-monitor-setup` | SLA tracking and breach alerting |
| `asana-task-creator` | Asana task with dependencies |

## User Story + Acceptance Criteria

```markdown
## US-4821: Email Notification on Order Shipment

**As a** customer who has placed an order  
**I want to** receive an email when my order ships  
**So that** I know when to expect my package and can track it

**Story Points:** 3  
**Priority:** High  
**Sprint:** 2025-Q2-S3

### Acceptance Criteria

**Scenario 1: Order ships successfully**
- **Given** a customer has a confirmed order in "Processing" status  
- **When** the fulfillment team marks the order as "Shipped" and enters a tracking number  
- **Then** the system sends a shipping confirmation email within 5 minutes  
- **And** the email contains: order number, items shipped, carrier name, tracking number, estimated delivery date, and a tracking link

**Scenario 2: Email delivery failure**
- **Given** the email service is unavailable  
- **When** the shipping confirmation email fails to send  
- **Then** the system retries 3 times with exponential backoff  
- **And** logs the failure for manual review if all retries fail

### Definition of Done
- [ ] Feature implemented and passing unit/integration tests
- [ ] Email template reviewed by Marketing
- [ ] End-to-end test with real carrier tracking URL
- [ ] Staged deployment verified
- [ ] Documentation updated
```

## OKR Template

```markdown
## Q2 2025 OKRs — Engineering

### Objective: Ship a world-class developer experience

**KR1:** Reduce API p99 latency from 450ms to < 200ms  
- Owner: Platform Team | Check-in: Weekly | Current: 380ms | Target: 200ms

**KR2:** Achieve 99.95% uptime (from 99.8%)  
- Owner: SRE Team | Check-in: Weekly | Current: 99.82% | Target: 99.95%

**KR3:** Increase developer satisfaction score from 6.2 to 8.0/10  
- Owner: DX Team | Check-in: Monthly | Current: 6.2 | Target: 8.0

### Progress: 🟡 On Track (65%)
```

## RAG Status Report

```markdown
## Project Status — API Modernization — Week 14

| Dimension | Status | Notes |
|-----------|--------|-------|
| Schedule | 🟢 Green | On track for May 26 launch |
| Budget | 🟡 Amber | 8% over forecast — monitoring |
| Quality | 🟢 Green | Test coverage at 94% |
| Risks | 🔴 Red | Third-party API dependency delayed |

### This Week
- ✅ Completed database migration for orders service
- ✅ Auth service integration tests passing
- ⚠️ Shipping API vendor delayed by 2 weeks → mitigation: mock for UAT

### Next Week
- Deploy to staging environment
- Begin load testing (target: 1000 RPS)

### Decisions Needed
- [ ] **By April 10**: Approve rollback plan for DB migration
```

## Risk Register

```markdown
## Risk Register — [Project Name]

| ID | Risk | Probability | Impact | Score | Mitigation | Owner |
|----|------|-------------|--------|-------|-----------|-------|
| R1 | Third-party API unavailable at launch | Medium (3) | High (4) | 12 | Build mock fallback; SLA in contract | Eng Lead |
| R2 | Database migration data loss | Low (2) | Critical (5) | 10 | Full backup pre-migration; dry run; rollback plan | DBA |
| R3 | Team capacity — 2 engineers on leave | High (4) | Medium (3) | 12 | Cross-train; hire contractor; descope v1 | PM |

**Score = Probability × Impact (1-5 scale)**  
**🔴 Critical: 15-25 | 🟠 High: 10-14 | 🟡 Medium: 5-9 | 🟢 Low: 1-4**
```

## Links

- [Repository](https://github.com/jeremylongshore/claude-code-plugins-plus-skills)
- Category: `20-enterprise-workflows` (26 skills)
