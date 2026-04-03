---
name: test-automation
description: Auto-activating test automation skill — Jest/pytest/Go test generators, mocks/stubs/spies, contract testing, property-based tests, mutation testing, and coverage analysis
origin: jeremylongshore/claude-code-plugins-plus-skills/09-test-automation
tools: Read, Write, Edit, Bash, Grep
---

# Test Automation Skill

Automated test generation and quality assistance: unit tests, integration tests, mocks, fixtures, contract testing, and coverage analysis across languages and frameworks.

## When to Activate

Activates automatically when you mention: test, jest, pytest, vitest, mocha, go test, mock, stub, spy, fixture, factory, contract test, integration test, property-based test, mutation test, snapshot test, test coverage, flaky test.

## Covered Skills (26 total)

| Skill | What it does |
|-------|-------------|
| `jest-test-generator` | Jest unit tests with describe/it/expect |
| `pytest-test-generator` | pytest fixtures, parametrize, async tests |
| `vitest-test-creator` | Vitest tests with vi.mock, coverage |
| `go-test-generator` | Go table-driven tests with testify |
| `mock-generator` | Jest/unittest mocks for any interface |
| `stub-creator` | HTTP stubs with msw or nock |
| `spy-setup-helper` | Spy patterns for side-effect verification |
| `factory-pattern-creator` | Test data factories (factory-boy, fishery) |
| `fixture-generator` | Reusable test fixtures and setup/teardown |
| `test-data-builder` | Builder pattern for complex test objects |
| `contract-test-creator` | Pact consumer/provider contract tests |
| `integration-test-setup` | Real database integration test patterns |
| `property-based-test-helper` | Hypothesis (Python) / fast-check (TS) |
| `mutation-test-runner` | Stryker / mutmut configuration |
| `snapshot-test-helper` | React component snapshot testing |
| `coverage-report-analyzer` | Identify untested branches and paths |
| `flaky-test-detector` | Identify and quarantine flaky tests |
| `test-parallelizer` | Parallel test execution configuration |

## Jest Test Pattern

```typescript
import { render, screen, userEvent } from '@testing-library/react'
import { vi } from 'vitest'
import { LoginForm } from './LoginForm'

const mockLogin = vi.fn()
vi.mock('../api/auth', () => ({ login: mockLogin }))

describe('LoginForm', () => {
  beforeEach(() => { vi.clearAllMocks() })

  it('submits credentials on form submit', async () => {
    const user = userEvent.setup()
    mockLogin.mockResolvedValueOnce({ user: { id: 1 } })
    render(<LoginForm />)

    await user.type(screen.getByLabelText(/email/i), 'test@example.com')
    await user.type(screen.getByLabelText(/password/i), 'secret')
    await user.click(screen.getByRole('button', { name: /sign in/i }))

    expect(mockLogin).toHaveBeenCalledWith({ email: 'test@example.com', password: 'secret' })
    expect(await screen.findByText(/welcome/i)).toBeInTheDocument()
  })

  it('shows error message on login failure', async () => {
    const user = userEvent.setup()
    mockLogin.mockRejectedValueOnce(new Error('Invalid credentials'))
    render(<LoginForm />)

    await user.click(screen.getByRole('button', { name: /sign in/i }))

    expect(await screen.findByRole('alert')).toHaveTextContent(/invalid credentials/i)
  })
})
```

## pytest Pattern

```python
import pytest
from unittest.mock import AsyncMock, patch

@pytest.fixture
def sample_user():
    return {"id": 1, "email": "test@example.com", "role": "admin"}

@pytest.mark.parametrize("email,password,expected_status", [
    ("valid@example.com", "correct", 200),
    ("valid@example.com", "wrong", 401),
    ("invalid", "pass", 422),
])
async def test_login(client, email, password, expected_status):
    response = await client.post("/auth/login", json={"email": email, "password": password})
    assert response.status_code == expected_status

@pytest.mark.asyncio
async def test_create_user(client, sample_user):
    with patch("app.services.user.send_welcome_email", new_callable=AsyncMock) as mock_email:
        response = await client.post("/users", json=sample_user)
        assert response.status_code == 201
        mock_email.assert_awaited_once_with(sample_user["email"])
```

## Go Table-Driven Tests

```go
func TestCalculate(t *testing.T) {
    tests := []struct {
        name     string
        input    int
        expected int
        wantErr  bool
    }{
        {"positive", 5, 25, false},
        {"zero", 0, 0, false},
        {"negative", -1, 0, true},
    }
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            got, err := Calculate(tt.input)
            if tt.wantErr {
                require.Error(t, err)
                return
            }
            require.NoError(t, err)
            assert.Equal(t, tt.expected, got)
        })
    }
}
```

## Links

- [Repository](https://github.com/jeremylongshore/claude-code-plugins-plus-skills)
- Category: `09-test-automation` (26 skills)
