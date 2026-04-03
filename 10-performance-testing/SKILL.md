---
name: performance-testing
description: Auto-activating performance testing skill — k6/Locust/JMeter/Artillery load tests, profiling, flame graphs, bottleneck identification, and performance baselines
origin: jeremylongshore/claude-code-plugins-plus-skills/10-performance-testing
tools: Read, Write, Edit, Bash, Grep
---

# Performance Testing Skill

Load testing, profiling, and performance analysis: script generation for k6/Locust/JMeter, CPU/memory profiling, flame graphs, throughput measurement, and baseline creation.

## When to Activate

Activates automatically when you mention: load test, stress test, k6, Locust, JMeter, Artillery, Gatling, performance, latency, throughput, p99, percentile, bottleneck, profiling, flame graph, heap dump, CPU profile, Apdex, benchmark, soak test, spike test.

## Covered Skills (25 total)

| Skill | What it does |
|-------|-------------|
| `k6-script-generator` | k6 JS scripts with stages, thresholds, checks |
| `locust-test-creator` | Locust Python tasks with wait time distribution |
| `jmeter-test-plan-creator` | JMeter JMX with thread groups, assertions |
| `artillery-config-generator` | Artillery YAML with phases and scenarios |
| `gatling-scenario-creator` | Gatling Scala simulation scripts |
| `load-test-scenario-planner` | Define realistic load profiles from traffic data |
| `soak-test-planner` | Extended duration test for memory leaks |
| `spike-test-setup` | Sudden traffic surge test configuration |
| `stress-test-config` | Find system breaking point |
| `cpu-profiler-config` | Node.js/Python CPU profiling setup |
| `memory-profiler-setup` | Heap profiling and leak detection |
| `flame-graph-generator` | Flamegraph from profiling data |
| `database-query-profiler` | Slow query analysis and EXPLAIN plans |
| `bottleneck-identifier` | Systematic performance bottleneck analysis |
| `benchmark-suite-creator` | Repeatable micro-benchmark setup |
| `performance-baseline-creator` | Establish and track performance baselines |
| `percentile-analyzer` | p50/p95/p99 latency analysis |
| `throughput-calculator` | RPS capacity planning |
| `apdex-score-calculator` | Application Performance Index scoring |
| `response-time-analyzer` | Latency distribution analysis |

## k6 Load Test Script

```javascript
import http from 'k6/http'
import { check, sleep } from 'k6'
import { Rate, Trend } from 'k6/metrics'

const errorRate = new Rate('errors')
const loginDuration = new Trend('login_duration')

export const options = {
  stages: [
    { duration: '2m', target: 50 },    // ramp up
    { duration: '5m', target: 50 },    // steady state
    { duration: '2m', target: 100 },   // spike
    { duration: '5m', target: 100 },   // sustain spike
    { duration: '2m', target: 0 },     // ramp down
  ],
  thresholds: {
    http_req_duration: ['p(95)<500', 'p(99)<1000'],
    errors: ['rate<0.01'],
    http_req_failed: ['rate<0.01'],
  },
}

export default function () {
  const loginRes = http.post(`${__ENV.BASE_URL}/auth/login`, JSON.stringify({
    email: 'user@example.com', password: 'secret'
  }), { headers: { 'Content-Type': 'application/json' } })

  loginDuration.add(loginRes.timings.duration)
  const loginOk = check(loginRes, {
    'login status 200': (r) => r.status === 200,
    'has token': (r) => r.json('token') !== undefined,
  })
  errorRate.add(!loginOk)

  if (loginOk) {
    const token = loginRes.json('token')
    const dataRes = http.get(`${__ENV.BASE_URL}/api/data`, {
      headers: { Authorization: `Bearer ${token}` }
    })
    check(dataRes, { 'data status 200': (r) => r.status === 200 })
  }

  sleep(1)
}
```

## Locust Test

```python
from locust import HttpUser, task, between
from locust.contrib.fasthttp import FastHttpUser

class APIUser(FastHttpUser):
    wait_time = between(1, 3)
    token = None

    def on_start(self):
        resp = self.client.post("/auth/login", json={"email": "user@example.com", "password": "secret"})
        if resp.status_code == 200:
            self.token = resp.json()["token"]

    @task(3)
    def get_dashboard(self):
        self.client.get("/api/dashboard", headers={"Authorization": f"Bearer {self.token}"})

    @task(1)
    def create_item(self):
        self.client.post("/api/items", json={"name": "test", "value": 42},
                         headers={"Authorization": f"Bearer {self.token}"})
```

## Performance Baseline Template

```markdown
## Performance Baseline — [Service] — [Date]

### Test Configuration
- Tool: k6 | Users: 100 concurrent | Duration: 10 min
- Environment: production-like (same instance type)

### Results
| Metric | Value | Threshold |
|--------|-------|-----------|
| p50 latency | 45ms | < 100ms |
| p95 latency | 180ms | < 500ms |
| p99 latency | 420ms | < 1000ms |
| Throughput | 850 RPS | — |
| Error rate | 0.02% | < 1% |
| Apdex score | 0.94 | > 0.85 |

### Next Review: [Date + 30 days]
```

## Links

- [Repository](https://github.com/jeremylongshore/claude-code-plugins-plus-skills)
- Category: `10-performance-testing` (25 skills)
