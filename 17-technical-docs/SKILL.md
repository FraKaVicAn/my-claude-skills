---
name: technical-docs
description: Auto-activating technical documentation skill — README, ADRs, API reference, runbooks, architecture docs, changelogs, migration guides, Docusaurus/MkDocs, and post-mortems
origin: jeremylongshore/claude-code-plugins-plus-skills/17-technical-docs
tools: Read, Write, Edit, Bash, Grep
---

# Technical Docs Skill

Automated documentation assistance: generate and maintain READMEs, ADRs, API references, runbooks, architecture documents, changelogs, migration guides, and documentation site configs.

## When to Activate

Activates automatically when you mention: README, ADR, architecture decision record, runbook, API reference, post-mortem, migration guide, contributing guide, changelog, release notes, quickstart, Docusaurus, MkDocs, VitePress, JSDoc, deprecation notice.

## Covered Skills (25 total)

| Skill | What it does |
|-------|-------------|
| `readme-generator` | Comprehensive README with badges, setup, usage |
| `adr-generator` | Architecture Decision Record in MADR format |
| `api-reference-creator` | API reference from code/OpenAPI spec |
| `runbook-creator` | Operational runbook with procedures and escalation |
| `architecture-doc-creator` | C4 model architecture documentation |
| `design-doc-template` | Design document for new systems/features |
| `incident-postmortem-template` | Blameless post-mortem template |
| `changelog-generator` | Keep-a-changelog format from git history |
| `migration-guide-creator` | Step-by-step version migration guide |
| `contributing-guide-creator` | CONTRIBUTING.md with PR and code standards |
| `quickstart-guide-generator` | 5-minute quickstart guide |
| `installation-guide-creator` | Multi-platform installation instructions |
| `troubleshooting-guide-creator` | Common issues and solutions |
| `faq-generator` | FAQ from common support questions |
| `jsdoc-comment-generator` | JSDoc comments for functions and classes |
| `sdk-documentation-generator` | SDK reference with examples |
| `deprecation-notice-generator` | Deprecation notices with migration paths |
| `docusaurus-config-setup` | Docusaurus site configuration |
| `mkdocs-config-generator` | MkDocs with Material theme |
| `vitepress-config-creator` | VitePress docs site config |
| `tutorial-outline-creator` | Tutorial structure with learning objectives |
| `code-of-conduct-generator` | Contributor Covenant Code of Conduct |

## ADR Template (MADR Format)

```markdown
# ADR-0001: Use PostgreSQL as Primary Database

**Date:** 2025-04-03  
**Status:** Accepted  
**Deciders:** [team names]

## Context and Problem Statement

We need a reliable, scalable relational database for our user and order data.

## Decision Drivers

- ACID compliance required for financial transactions
- Team familiarity with SQL
- Need for complex joins and reporting queries

## Considered Options

1. PostgreSQL
2. MySQL
3. MongoDB

## Decision Outcome

**Chosen: PostgreSQL**

Rationale: Best-in-class JSON support alongside relational features, strong ecosystem (pgvector for future ML features, TimescaleDB for time series), and team expertise.

### Positive Consequences

- Full ACID compliance for transactions
- Rich querying with window functions and CTEs
- pgvector available for embedding search

### Negative Consequences

- Vertical scaling limits (mitigated by read replicas)
- Schema migrations require downtime (mitigated by expand/contract pattern)

## Pros and Cons of Options

### PostgreSQL
- ✅ ACID, mature, rich ecosystem
- ❌ Horizontal write scaling limited

### MySQL
- ✅ Simpler operational model
- ❌ Less feature-rich JSON support

### MongoDB
- ✅ Flexible schema
- ❌ No ACID across collections, team less familiar
```

## Runbook Template

```markdown
# Runbook: High API Latency

**Service:** API Gateway  
**Owner:** Platform Team  
**Last Updated:** 2025-04-03

## Symptoms

- p99 latency > 2s (alert: `api-latency-p99-high`)
- Error rate increase in dependent services
- User-reported slowness

## Immediate Actions (first 5 minutes)

1. Check [Grafana dashboard](https://grafana.internal/d/api-latency)
2. Identify affected endpoints: `kubectl logs -l app=api --since=5m | grep "duration"`
3. Check database connections: `psql $DB_URL -c "SELECT count(*) FROM pg_stat_activity"`

## Escalation

- **15 min**: Page database team if DB connections > 80%
- **30 min**: Engage on-call engineer

## Common Causes & Fixes

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| All endpoints slow | Database connection pool exhausted | Restart connection pool, check for long transactions |
| Specific endpoint slow | N+1 query | Check query logs, add eager loading |
| Latency spikes | Memory pressure / GC | Check memory metrics, restart pod |

## Rollback

```bash
kubectl rollout undo deployment/api
kubectl rollout status deployment/api
```
```

## Links

- [Repository](https://github.com/jeremylongshore/claude-code-plugins-plus-skills)
- Category: `17-technical-docs` (25 skills)
