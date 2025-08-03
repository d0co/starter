# Starter - B2B SaaS Monorepo

Full-stack TypeScript monorepo with authentication, database, and deployment pre-configured.

## Quick Start

```bash
git clone https://github.com/d0co/starter.git app && cd app
pnpm install
cp .env.example .env.local
pnpm db:push
pnpm dev
```

## What's Included

| Category | Technology | What It Solves |
|----------|------------|----------------|
| **Frontend** | Next.js 15 | Production React with server rendering, zero-config optimization |
| **Auth** | Clerk | Complete authentication - login, signup, orgs, SSO, 2FA |
| **Database** | PostgreSQL + Drizzle | Type-safe queries, no SQL injection, migrations |
| **API** | tRPC | End-to-end TypeScript from database to UI |
| **UI** | shadcn/ui | Copy-paste components you own and can modify |
| **Edge** | Cloudflare Workers | Global low latency API endpoints |
| **Jobs** | Inngest | Background tasks with automatic retries |
| **Email** | Resend | Transactional emails that reach inboxes |
| **Monitoring** | Sentry | Error tracking before users complain |
| **DX** | TypeScript + Turborepo | Type safety + fast monorepo builds |

## Project Structure

```
apps/
├── web/                    # Next.js frontend (http://localhost:3000)
│   ├── app/               # Pages, layouts, and API routes
│   └── components/        # Page-specific React components
└── api/                    # Cloudflare Worker (optional edge API)

packages/
├── db/                     # Database schemas, migrations, and client
├── api/                    # tRPC routers and business logic  
├── ui/                     # Shared React components (shadcn/ui base)
├── config/                 # Environment validation and app config
└── jobs/                   # Background job definitions (Inngest)
```

## Development

```bash
pnpm dev
pnpm build
pnpm type-check
pnpm lint
pnpm db:push       # Dev database sync
pnpm db:migrate    # Production migrations
pnpm db:studio
pnpm test
```

### Environment Variables

| Variable | Required | Description | Example |
|----------|----------|-------------|---------|
| `DATABASE_URL` | ✓ | PostgreSQL connection string | `postgresql://user:pass@host:5432/db` |
| `NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY` | ✓ | Clerk public key | `pk_test_...` |
| `CLERK_SECRET_KEY` | ✓ | Clerk secret key | `sk_test_...` |
| `NEXT_PUBLIC_APP_URL` | ✓ | Your app URL | `http://localhost:3000` |
| `RESEND_API_KEY` | | Email service | `re_...` |
| `INNGEST_EVENT_KEY` | | Background jobs | `test_...` |
| `SENTRY_DSN` | | Error tracking | `https://...@sentry.io/...` |
| `CLOUDFLARE_API_TOKEN` | | Edge deployment | `...` |

### Code Patterns

#### Database Schema
```typescript
// packages/db/src/schema/users.ts
export const users = pgTable('users', {
  id: uuid('id').primaryKey().defaultRandom(),
  email: text('email').notNull().unique(),
  organizationId: uuid('organization_id').references(() => organizations.id),
  createdAt: timestamp('created_at').defaultNow(),
});
```

#### API Route (tRPC)
```typescript
// packages/api/src/routers/user.ts
export const userRouter = createTRPCRouter({
  list: protectedProcedure
    .input(z.object({ limit: z.number().default(10) }))
    .query(({ ctx, input }) => {
      return ctx.db.query.users.findMany({
        where: eq(users.organizationId, ctx.auth.orgId),
        limit: input.limit,
      });
    }),
});
```

#### React Component
```typescript
// apps/web/components/users/list.tsx
export function UsersList() {
  const { data, isLoading } = api.user.list.useQuery({ limit: 20 });
  
  if (isLoading) return <Spinner />;
  
  return (
    <div>
      {data?.map(user => <UserCard key={user.id} user={user} />)}
    </div>
  );
}
```

#### Authentication
```typescript
// Server Component
import { auth } from '@clerk/nextjs/server';

export default async function Page() {
  const { userId } = await auth();
  if (!userId) redirect('/login');
  // Protected content
}

// Client Component  
import { useAuth } from '@clerk/nextjs';

export function Component() {
  const { userId, isLoaded } = useAuth();
  if (!isLoaded) return <Spinner />;
  // Protected content
}
```

### Common Issues & Fixes

#### Module not found
```bash
pnpm build --filter=@starter/db
```

#### Database connection failed
```bash
# Check DATABASE_URL and test with:
pnpm db:studio
```

#### Type errors after schema change
```bash
pnpm db:push && pnpm type-check
```

#### Clerk auth not working
```bash
# Ensure BOTH keys are set:
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_...
CLERK_SECRET_KEY=sk_...
```

## Best Practices

**DO:**
- Handle errors explicitly
- Use tRPC for all APIs
- Test critical paths (auth, payments, core features)
- Keep packages focused (one concern per package)

**DON'T:**
- Use `any` type
- Build custom auth (use Clerk)
- Create unnecessary abstractions
- Use npm/yarn (only pnpm)


## Deployment & Costs

| Service | What It Provides | Monthly Cost |
|---------|-----------------|--------------|
| **Railway** | Next.js hosting + PostgreSQL database | $10 |
| **Cloudflare Pro** | Workers, KV cache, R2 storage, CDN | $25 |
| **Clerk** | Authentication (free up to 5K users) | $25 |
| **Resend** | Email delivery | $20 |
| **Sentry** | Error tracking | $26 |
| **Total** | | **~$106** |

Deploy: Create PR → Merge to `develop` → Merge to `main` → Railway auto-deploys

## Stack Trade-offs

| Choice | Why Chosen | Trade-off |
|--------|------------|-----------|
| Next.js 15 | Best-in-class React framework with built-in optimizations | Best on Railway/Cloudflare, not Vercel |
| Clerk | Complete auth solution that handles edge cases | Monthly cost, less control over auth flow |
| PostgreSQL | Battle-tested, ACID compliant, great tooling | Requires more setup than SQLite |
| Drizzle | Type-safe queries without learning new syntax | Less mature than Prisma |
| tRPC | End-to-end type safety without code generation | Only works with TypeScript backends |
| Turborepo | Fast incremental builds, great monorepo DX | Extra complexity vs single package.json |
| pnpm | Faster installs, handles monorepos better | Team needs pnpm installed (not just npm) |

## License

MIT