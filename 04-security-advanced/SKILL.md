---
name: security-advanced
description: Auto-activating advanced security skill — GDPR/HIPAA/SOC2/PCI compliance scanning, threat modeling, penetration testing, IAM review, zero trust, container security, and SIEM rules
origin: jeremylongshore/claude-code-plugins-plus-skills/04-security-advanced
tools: Read, Write, Grep, Bash
---

# Security Advanced Skill

Enterprise-grade security assistance: compliance auditing, threat modeling, penetration testing planning, cloud security posture, and security operations.

## When to Activate

Activates automatically when you mention: GDPR, HIPAA, SOC2, PCI-DSS, ISO 27001, compliance, threat model, penetration test, IAM, zero trust, RBAC, container security, WAF, SIEM, forensics, incident response, encryption, key rotation, certificate management.

## Covered Skills (26 total)

| Skill | What it does |
|-------|-------------|
| `gdpr-compliance-scanner` | Audit code/config for GDPR requirements |
| `hipaa-audit-helper` | PHI handling, audit logs, encryption requirements |
| `soc2-compliance-checker` | Trust service criteria gap analysis |
| `pci-dss-validator` | Cardholder data environment requirements |
| `iso27001-gap-analyzer` | ISO 27001 control gap assessment |
| `threat-model-creator` | STRIDE-based threat modeling for architectures |
| `attack-surface-analyzer` | Enumerate exposed attack surfaces |
| `penetration-test-planner` | Scoped pentest methodology and checklists |
| `incident-response-planner` | IR runbooks, escalation paths, containment steps |
| `iam-policy-reviewer` | Least-privilege analysis for AWS/GCP IAM |
| `kubernetes-rbac-analyzer` | K8s RBAC role/binding audit |
| `zero-trust-config-helper` | Zero trust architecture patterns |
| `container-security-auditor` | Dockerfile and image security scanning |
| `cloud-security-posture` | CIS benchmark checks for AWS/GCP/Azure |
| `encryption-at-rest-checker` | Verify data-at-rest encryption configuration |
| `key-rotation-manager` | Key rotation schedules and automation |
| `certificate-lifecycle-manager` | TLS cert renewal, expiry monitoring |
| `siem-rule-generator` | Detection rules for common attack patterns |
| `waf-rule-creator` | WAF rule sets for OWASP Top 10 |
| `vulnerability-report-generator` | Structured vuln reports with CVSS scoring |
| `log-analysis-security` | Security event detection in logs |
| `forensics-data-collector` | Evidence collection procedures |

## STRIDE Threat Model Template

```markdown
## Threat Model: [System Name]

### Trust Boundaries
- External users → API Gateway
- API Gateway → Microservices (internal)
- Microservices → Database

### STRIDE Analysis per Component

#### API Gateway
| Threat Type | Threat | Mitigation |
|-------------|--------|------------|
| Spoofing | Fake client identity | mTLS, API keys, JWT validation |
| Tampering | Request modification | Request signing, HTTPS only |
| Repudiation | Deny actions | Structured audit logging |
| Info Disclosure | Leak internals | Strip server headers, sanitize errors |
| DoS | Rate exhaustion | Rate limiting, DDoS protection |
| Elevation | Bypass auth | AuthZ on every endpoint |
```

## IAM Least-Privilege Pattern (AWS)

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": [
      "s3:GetObject",
      "s3:PutObject"
    ],
    "Resource": "arn:aws:s3:::my-bucket/*",
    "Condition": {
      "StringEquals": {"s3:prefix": ["uploads/"]},
      "Bool": {"aws:SecureTransport": "true"}
    }
  }]
}
```

## Container Security Checklist

```dockerfile
# ✅ Minimal base image
FROM gcr.io/distroless/python3-debian12

# ✅ Non-root user
USER nonroot:nonroot

# ✅ Read-only filesystem (set in k8s securityContext)
# securityContext:
#   readOnlyRootFilesystem: true
#   runAsNonRoot: true
#   allowPrivilegeEscalation: false

# ✅ No new privileges
# securityContext:
#   seccompProfile: { type: RuntimeDefault }
#   capabilities: { drop: [ALL] }
```

## Links

- [Repository](https://github.com/jeremylongshore/claude-code-plugins-plus-skills)
- Category: `04-security-advanced` (26 skills)
