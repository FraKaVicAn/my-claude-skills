---
name: security-auth
description: Security best practices from Trail of Bits — vulnerability scanning, smart contract auditing, SAST/DAST. Better Auth integration — providers, 2FA, organizations, email/password. Auto-activates for auth or security code.
origin: VoltAgent/awesome-agent-skills (trail-of-bits, better-auth)
tools: Read, Write, Edit, Bash
---

# Security & Authentication Skills

Official skills from Trail of Bits (security) and Better Auth (authentication).

## Better Auth Setup

```bash
npm install better-auth
```

```ts
// lib/auth.ts
import { betterAuth } from 'better-auth';
import { prismaAdapter } from 'better-auth/adapters/prisma';
import { twoFactor, organization, emailOTP } from 'better-auth/plugins';

export const auth = betterAuth({
  database: prismaAdapter(prisma, { provider: 'postgresql' }),
  secret: process.env.BETTER_AUTH_SECRET!,
  baseURL: process.env.BETTER_AUTH_URL!,

  emailAndPassword: {
    enabled: true,
    requireEmailVerification: true,
    minPasswordLength: 8,
  },

  socialProviders: {
    google: {
      clientId: process.env.GOOGLE_CLIENT_ID!,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET!,
    },
    github: {
      clientId: process.env.GITHUB_CLIENT_ID!,
      clientSecret: process.env.GITHUB_CLIENT_SECRET!,
    },
  },

  plugins: [
    twoFactor({
      issuer: 'MyApp',
      otpOptions: { digits: 6, period: 30 },
    }),
    organization({
      allowUserToCreateOrganization: true,
      organizationLimit: 5,
    }),
  ],
});

export type Session = typeof auth.$Infer.Session;
```

```ts
// app/api/auth/[...all]/route.ts
import { auth } from '@/lib/auth';
import { toNextJsHandler } from 'better-auth/next-js';
export const { GET, POST } = toNextJsHandler(auth);
```

## Better Auth Client

```ts
// lib/auth-client.ts
import { createAuthClient } from 'better-auth/react';
import { twoFactorClient, organizationClient } from 'better-auth/client/plugins';

export const authClient = createAuthClient({
  baseURL: process.env.NEXT_PUBLIC_BETTER_AUTH_URL!,
  plugins: [twoFactorClient(), organizationClient()],
});

export const { signIn, signUp, signOut, useSession } = authClient;
```

```tsx
// Usage in components
function LoginForm() {
  const { data: session } = useSession();

  const handleSignIn = async () => {
    await signIn.email({ email, password, callbackURL: '/dashboard' });
  };

  const handleGoogleSignIn = async () => {
    await signIn.social({ provider: 'google', callbackURL: '/dashboard' });
  };

  return (/* ... */);
}
```

## Email & Password Auth

```ts
// Sign up
await authClient.signUp.email({
  email: 'user@example.com',
  password: 'securepassword',
  name: 'John Doe',
  callbackURL: '/dashboard',
});

// Sign in
await authClient.signIn.email({
  email: 'user@example.com',
  password: 'securepassword',
});

// Forgot password
await authClient.forgetPassword({
  email: 'user@example.com',
  redirectTo: '/reset-password',
});

// Reset password
await authClient.resetPassword({
  newPassword: 'newpassword',
  token: searchParams.get('token')!,
});
```

## Two-Factor Authentication (TOTP)

```ts
// Enable 2FA for user
const { totpURI, backupCodes } = await authClient.twoFactor.enable({
  password: 'currentpassword',
});
// Show QR code from totpURI

// Verify TOTP code
await authClient.twoFactor.verifyTotp({ code: '123456' });

// Disable 2FA
await authClient.twoFactor.disable({ password: 'currentpassword' });
```

## Organizations / Multi-Tenant

```ts
// Create organization
const org = await authClient.organization.create({
  name: 'Acme Corp',
  slug: 'acme-corp',
});

// Invite member
await authClient.organization.inviteMember({
  email: 'colleague@example.com',
  role: 'member',
  organizationId: org.id,
});

// Switch active organization
await authClient.organization.setActive({ organizationId: org.id });

// Get members
const members = await authClient.organization.listMembers();
```

## Protecting Routes

```ts
// Next.js middleware
import { auth } from '@/lib/auth';
import { NextResponse } from 'next/server';

export async function middleware(request: Request) {
  const session = await auth.api.getSession({ headers: request.headers });

  if (!session && request.url.includes('/dashboard')) {
    return NextResponse.redirect(new URL('/login', request.url));
  }

  return NextResponse.next();
}
```

## Trail of Bits Security Practices

### Dependency Security

```bash
# Audit dependencies
npm audit
npm audit fix

# Python
pip-audit
safety check

# Use lockfiles — always commit package-lock.json / yarn.lock / poetry.lock
# Pin exact versions in production
```

### Input Validation & Sanitization

```ts
// NEVER trust user input
import { z } from 'zod';

const UserInput = z.object({
  email: z.string().email().max(254),
  name: z.string().min(1).max(100).regex(/^[a-zA-Z\s-]+$/),
  age: z.number().int().min(0).max(150),
});

// SQL injection prevention — ALWAYS use parameterized queries
// BAD:
const query = `SELECT * FROM users WHERE id = ${userId}`;
// GOOD:
const result = await db.query('SELECT * FROM users WHERE id = $1', [userId]);
```

### Secret Management

```bash
# Never hardcode secrets — use environment variables
# Use secret scanning to detect leaks
git secrets --install
git secrets --register-aws

# Scan for secrets
trufflehog git file://./  --only-verified
gitleaks detect --source .

# Rotate secrets immediately if exposed
```

### SAST (Static Analysis)

```bash
# JavaScript/TypeScript
npx semgrep --config auto src/

# Python
bandit -r src/ -ll

# Security linting
eslint --plugin security src/

# Snyk
snyk code test
snyk test  # dependency vulnerabilities
```

### Smart Contract Auditing (Trail of Bits)

```bash
# Install Slither
pip install slither-analyzer

# Run analysis
slither contracts/ --print human-summary
slither contracts/ --detect reentrancy-eth,arbitrary-send-eth

# Echidna fuzzing
echidna-test . --contract MyContract --test-mode assertion

# Manticore symbolic execution
manticore contracts/MyContract.sol
```

### Security Headers

```ts
// Apply security headers to all responses
const securityHeaders = {
  'Content-Security-Policy': "default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline'",
  'X-Frame-Options': 'DENY',
  'X-Content-Type-Options': 'nosniff',
  'Referrer-Policy': 'strict-origin-when-cross-origin',
  'Permissions-Policy': 'camera=(), microphone=(), geolocation=()',
  'Strict-Transport-Security': 'max-age=63072000; includeSubDomains; preload',
};
```

### OWASP Top 10 Checklist

1. **Broken Access Control** — verify every request, use allowlists
2. **Cryptographic Failures** — TLS everywhere, bcrypt for passwords, no MD5/SHA1
3. **Injection** — parameterized queries, ORM, input validation
4. **Insecure Design** — threat modeling, secure defaults
5. **Security Misconfiguration** — disable debug in prod, update deps
6. **Vulnerable Components** — `npm audit`, `pip-audit` in CI
7. **Auth Failures** — MFA, rate limiting, secure session management
8. **Integrity Failures** — verify package integrity, code signing
9. **Logging Failures** — log auth events, suspicious activity, anomalies
10. **SSRF** — validate and allowlist URLs for outbound requests

## Sources
- better-auth/best-practices
- better-auth/create-auth
- better-auth/emailAndPassword
- better-auth/twoFactor
- better-auth/organization
- better-auth/providers
- trail-of-bits security skills
