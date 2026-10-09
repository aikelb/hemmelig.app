# Hemmelig.app — Agent Instructions

## What is Hemmelig?

Hemmelig.app is a secure secret-sharing application. Users share encrypted messages that self-destruct after being read. "Hemmelig" means "secret" in Norwegian.

### Zero-Knowledge Architecture (CRITICAL)

This is the core security model. Every change must preserve it:

- All encryption/decryption happens **client-side** using the Web Crypto API
- The server **never** sees plaintext secrets — only encrypted blobs
- Decryption keys live in URL fragments (`/s/<id>#<key`), which browsers **never send to servers**
- Compromising this model is a showstopper bug

### Encryption Details

| Component          | Value                                                    |
| ------------------ | -------------------------------------------------------- |
| Algorithm          | AES-256-GCM                                              |
| Key Derivation     | PBKDF2 + SHA-256, 1,300,000 iterations                   |
| IV                 | 96-bit random per encryption                             |
| Salt               | 32-char random string per secret (stored server-side)    |
| Implementation     | `src/lib/crypto.ts`                                      |

## Technology Stack

| Layer         | Tech                           |
| ------------- | ------------------------------ |
| Runtime       | Node.js 25                     |
| Frontend      | React 19 + Vite + TypeScript   |
| Backend       | Hono (RPC mode)                |
| Database      | SQLite + Prisma ORM            |
| Styling       | Tailwind CSS v4                |
| State         | Zustand                        |
| Auth          | better-auth (session + 2FA)    |
| i18n          | react-i18next                  |
| Monitoring    | prom-client                    |

## Project Structure

```
hemmelig.app/
├── api/                          # Backend (Hono)
│   ├── app.ts                    # Main Hono app
│   ├── auth.ts                   # better-auth config
│   ├── routes.ts                 # Route aggregator
│   ├── routes/                   # Route handlers
│   │   ├── secrets.ts            # Secret CRUD
│   │   ├── secret-requests.ts    # Request management
│   │   ├── account.ts            # User account
│   │   ├── files.ts              # File upload/download
│   │   ├── user.ts               # Admin user mgmt
│   │   ├── instance.ts           # Instance settings
│   │   ├── analytics.ts          # Usage analytics
│   │   ├── invites.ts            # Invite codes
│   │   ├── api-keys.ts           # API keys
│   │   ├── setup.ts              # Initial setup
│   │   ├── health.ts             # Health checks
│   │   └── metrics.ts            # Prometheus metrics
│   ├── lib/                      # Backend utilities
│   │   ├── db.ts                 # Prisma client singleton
│   │   ├── password.ts           # Argon2 hashing
│   │   ├── files.ts              # File handling
│   │   ├── settings.ts           # Instance settings helper
│   │   ├── webhook.ts            # Webhook dispatch
│   │   └── utils.ts              # General utilities
│   ├── middlewares/              # Hono middlewares
│   │   ├── auth.ts               # Auth middleware
│   │   ├── ratelimit.ts          # Rate limiting
│   │   └── ip-restriction.ts     # IP allow/blocklist
│   ├── validations/              # Zod schemas
│   └── jobs/                     # Background jobs
├── src/                          # Frontend (React)
│   ├── components/               # Reusable UI
│   │   ├── Button, Input, Chip, Segmented, Tabs, Tag, Table
│   │   ├── StatGrid, Settings, PageHeader, Avatar, Notice
│   │   ├── Modal, ToggleSwitch, LoadingButton, Pagination
│   │   ├── Editor.tsx            # TipTap rich text editor
│   │   ├── SecretForm.tsx        # Secret creation form
│   │   ├── SecretSettings.tsx    # Secret settings panel
│   │   ├── FileUpload.tsx        # Drag-and-drop uploads
│   │   └── Layout/               # Layout wrappers
│   ├── pages/                    # Route-level components
│   │   ├── HomePage.tsx
│   │   ├── SecretPage.tsx        # View/decrypt secret
│   │   ├── LoginPage.tsx
│   │   ├── RegisterPage.tsx
│   │   ├── SetupPage.tsx         # Admin setup wizard
│   │   ├── Verify2FAPage.tsx     # 2FA verification
│   │   ├── Dashboard/            # Admin dashboard pages
│   │   └── ...
│   ├── store/                    # Zustand stores
│   │   ├── secretStore.ts        # Secret creation state
│   │   ├── userStore.ts          # Current user
│   │   ├── hemmeligStore.ts      # Instance settings
│   │   ├── themeStore.ts         # Light/dark mode (persisted)
│   │   └── ...
│   ├── lib/                      # Frontend utilities
│   │   ├── api.ts                # Hono RPC client
│   │   ├── auth.ts               # better-auth client
│   │   ├── crypto.ts             # Client-side encryption
│   │   └── analytics.ts          # Page view tracking
│   ├── i18n/locales/             # Translation files (10 languages)
│   └── router.tsx                # React Router config
├── prisma/schema.prisma          # Database schema
├── server.ts                     # Production entry point
├── docs/                         # User-facing documentation
├── tests/e2e/                    # Playwright e2e tests
└── scripts/                      # Utility scripts
```

## Commands

```bash
npm install                          # Install deps
npm run dev                           # Frontend (Vite, HMR)
npm run dev:api                       # Backend (tsx watch + migrate dev)
npm run build                         # Production build
npm run start                         # Run production server
npm run migrate:dev                   # Create + apply migration
npm run migrate:deploy                # Apply pending migrations (prod)
npm run migrate:reset                 # Reset database (destructive!)
npm run migrate:status                # Check migration status
npm run set:admin                     # Promote user to admin
npm run seed:demo                     # Seed demo data
npm run format                        # Prettier write
npm run format:check                  # Prettier check
npm run test:e2e                      # Playwright e2e tests
```

## Core Principles

1. **Surgical changes only.** Touch what's asked. No refactoring unrelated code.
2. **Follow existing patterns.** Consistency over preference.
3. **No dependency changes without approval.**
4. **Security-first mindset.** Extra scrutiny on encryption, auth, data handling.

## Frontend Guidelines

### Components

```tsx
export function MyComponent({ prop1 }: Props) {
    const { t } = useTranslation(); // Always translate user-facing text
    const [state, setState] = useState<Type>(initialValue);

    return <div className="bg-surface text-fg border border-line rounded-md" />;
}
```

### Design System

- Compact design, minimal padding.
- `rounded-sm` (4px) for controls, `rounded-md` (6px) for panels.
- **Light/dark mode mandatory** — use semantic tokens, no manual `dark:` variants.
- Mobile-first responsive.
- Fonts: IBM Plex Sans (body), JetBrains Mono (`font-mono`) for labels/links/keys/metadata.

### Styling with Tailwind

Use semantic CSS variables from `src/index.css` `@theme` block. Never hardcode colors when a token exists.

```tsx
// GOOD
className="bg-surface text-fg border border-line rounded-md"

// BAD — manual dark variant over a token that already handles it
className="bg-white dark:bg-dark-800 text-gray-900 dark:text-white"

// BAD — arbitrary value when a token exists
className="bg-[#14181d]"
```

**Color tokens:** `canvas` (page bg), `surface` (panels/inputs), `raised` (hover/active/inline-code), `line-faint/soft/line/strong` (borders), `fg/fg-2/fg-3` (text levels), `muted/faint` (labels/hints), `accent/on-accent` (primary actions), `danger/on-danger` (destructive), `warn` (notices).

**Sizes:** text — `text-2xs` (11px), `text-ui` (13px), `text-body` (15px). Containers — `max-w-page` (1120px), `max-w-content` (820px), `max-w-reading` (720px).

### State Management

```typescript
// src/store/exampleStore.ts
import { create } from 'zustand';
interface ExampleState { data: string; setData: (d: string) => void; }
export const useExampleStore = create<ExampleState>((set) => ({
    data: '',
    setData: (data) => set({ data }),
}));
```

### API Calls

Use the typed Hono RPC client from `src/lib/api.ts`:

```typescript
import { api } from '../lib/api';
const response = await api.secrets.$post({ json: { secret: encryptedData } });
const data = await response.json();
```

### React Router v7 Loaders

```typescript
// router.tsx
{ path: '/dashboard/secrets', element: <SecretsPage />, loader: async () => {
    const res = await api.secrets.$get();
    return await res.json();
}}

// Component
const data = useLoaderData();
```

## Internationalization (i18n)

**Golden rule:** When adding or modifying any user-facing string, update ALL locale files.

Supported languages: English, Danish, German, Spanish, French, Italian, Dutch, Norwegian, Swedish, Chinese.

Files: `src/i18n/locales/{en,da,de,es,fr,it,nl,no,sv,zh}/{en,da,de,es,fr,it,nl,no,sv,zh}.json`

Add to English first, then add the key (English placeholder is fine) to every other locale file. Use nested keys matching existing structure.

## Backend Guidelines

### Route Pattern

```typescript
// api/routes/example.ts
import { Hono } from 'hono';
import { zValidator } from '@hono/zod-validator';
import { z } from 'zod';

const exampleRoute = new Hono().post(
    '/',
    zValidator('json', z.object({ field: z.string().min(1).max(1000) })),
    async (c) => {
        const { field } = c.req.valid('json');
        return c.json({ success: true });
    }
);
export default exampleRoute;
```

### Database

- Always use `api/lib/db.ts` singleton.
- Use transactions for multi-operation writes.
- Schema changes: edit `prisma/schema.prisma`, run `npm run migrate:dev --name descriptive_name`, test locally, commit both.

### Validation & Errors

- Validate all input with Zod at route level via `zValidator`.
- Reusable schemas go in `api/validations/`.
- Return `{ error: 'message' }` with appropriate status codes.

## Security Checklist

When touching security-sensitive code:

- [ ] Encryption/decryption stays client-side only
- [ ] Decryption keys never reach the server
- [ ] Input validation on all endpoints
- [ ] Auth checks in place where required
- [ ] Rate limiting on sensitive endpoints
- [ ] No sensitive data in logs or error messages
- [ ] File uploads validated and sanitized

## Hard Rules — No Exceptions

1. Never modify encryption logic without explicit approval
2. Never log or store plaintext secrets server-side
3. Never send decryption keys to the server
4. Never bypass input validation
5. Never add/remove/update dependencies without approval
6. Never modify unrelated code "while you're in there"
7. Never use `any` types in TypeScript
8. Never commit `.env` files or secrets
9. Never disable security features "temporarily"
10. Never add user-facing strings without updating ALL translation files

## Managed Mode

When `HEMMELIG_MANAGED=true`, instance settings are controlled via environment variables, not the database. The admin dashboard becomes read-only. Full list in `docs/managed.md`. Key vars:

- `HEMMELIG_INSTANCE_NAME`, `HEMMELIG_INSTANCE_DESCRIPTION`
- `HEMMELIG_DEFAULT_THEME` (dark/light/system)
- `HEMMELIG_ALLOW_REGISTRATION`, `HEMMELIG_REQUIRE_INVITE_CODE`
- `HEMMELIG_ALLOWED_EMAIL_DOMAINS`
- `HEMMELIG_DEFAULT_SECRET_EXPIRATION`, `HEMMELIG_DEFAULT_MAX_VIEWS`, `HEMMELIG_MAX_SECRET_SIZE`
- `HEMMELIG_WEBHOOK_ENABLED/URL/SECRET`, `HEMMELIG_METRICS_ENABLED/SECRET`

## Documentation

User-facing docs live in `docs/`: encryption, env vars, docker, helm, social login, webhooks, API, CLI, MCP server, upgrade path. Reference these before implementing features they cover.

---

_This file is the source of truth for development practices in this repository._
