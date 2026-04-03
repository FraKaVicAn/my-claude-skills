---
name: devops-basics
description: Auto-activating DevOps fundamentals skill — git workflows, Docker, CI/CD, bash scripts, dotenv, Makefile, GitHub Actions, pre-commit hooks, and release management
origin: jeremylongshore/claude-code-plugins-plus-skills/01-devops-basics
tools: Read, Write, Edit, Bash, Grep
---

# DevOps Basics Skill

Automated assistance for foundational DevOps tasks: version control, containerization, CI/CD pipelines, shell scripting, and configuration management.

## When to Activate

Activates automatically when you mention: git workflow, docker, dockerfile, github actions, bash script, makefile, pre-commit, dotenv, environment variables, changelog, release notes, npm scripts, gitignore, ssh key, version bump, readme, yaml config.

## Covered Skills (29 total)

| Skill | What it does |
|-------|-------------|
| `git-workflow-manager` | Branch strategies, commit conventions, merge/rebase workflows |
| `dockerfile-generator` | Multi-stage builds, layer optimization, security hardening |
| `docker-compose-creator` | Multi-service stacks, networks, volumes, healthchecks |
| `github-actions-starter` | CI/CD workflows, matrix builds, secrets, caching |
| `gitlab-ci-basics` | Pipeline stages, jobs, artifacts, Docker-in-Docker |
| `bash-script-helper` | Script structure, error handling, argument parsing |
| `makefile-generator` | Targets, variables, phony rules, dependency chains |
| `pre-commit-hook-setup` | Hooks for linting, formatting, secret detection |
| `dotenv-manager` | .env structure, validation, secrets separation |
| `changelog-creator` | Keep-a-changelog format, semantic versioning |
| `release-notes-generator` | Git-log to user-friendly release notes |
| `version-bumper` | semver bumping across package.json, pyproject.toml, etc. |
| `readme-generator` | Project README with badges, setup, usage sections |
| `gitignore-generator` | Language/framework-specific .gitignore |
| `ssh-key-manager` | Key generation, agent setup, authorized_keys |
| `npm-scripts-optimizer` | Script composition, lifecycle hooks, parallelism |
| `package-json-manager` | Dependencies, engines, exports, scripts |
| `linux-commands-guide` | File ops, permissions, process management, networking |
| `json-config-manager` | Validation, schema enforcement, merging |
| `yaml-config-validator` | Linting, schema validation, anchor/alias usage |
| `jenkins-pipeline-intro` | Declarative pipelines, stages, agents |
| `environment-variables-handler` | Shell export patterns, secret injection |

## Git Workflow Quick Reference

```bash
# Feature branch workflow
git checkout -b feat/TICKET-123-description
git add -p                          # stage hunks, not files
git commit -m "feat(scope): message"
git push -u origin feat/TICKET-123-description

# Conventional commit types
# feat: new feature | fix: bug fix | docs: docs only
# style: formatting | refactor: no behavior change
# test: adding tests | chore: build/tooling
```

## Dockerfile Best Practices

```dockerfile
# Multi-stage build
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

FROM node:20-alpine AS runtime
WORKDIR /app
COPY --from=builder /app/node_modules ./node_modules
COPY . .
USER node                           # never run as root
EXPOSE 3000
HEALTHCHECK --interval=30s CMD wget -qO- http://localhost:3000/health
CMD ["node", "server.js"]
```

## GitHub Actions Starter

```yaml
name: CI
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 20, cache: npm }
      - run: npm ci
      - run: npm test
      - run: npm run build
```

## Links

- [Repository](https://github.com/jeremylongshore/claude-code-plugins-plus-skills)
- Category: `01-devops-basics` (29 skills)
