# AI Prototyping Patterns

Practical patterns for building full-stack products with AI-native tools. Based on shipping real products (e.g. [Pupsday](https://pupsday.com)) and what we operationalize at [BARO](https://heybaro.com).

## Who this is for

Solo founders and small teams who want to:

- Ship faster using **Claude**, **Cursor**, **Codex**, **Vercel**, and **Supabase**
- Build full-stack applications without a large team
- Know when AI helps vs. when you need hard engineering or ops discipline

## The stack (today)

| Tool | Role | When to use |
|------|------|-------------|
| [Claude](https://claude.ai) | Reasoning, architecture, reviews, long-context work | Ambiguous specs, system design, careful refactors |
| [Cursor](https://cursor.com) | Day-to-day IDE + repo navigation | Almost all implementation; multi-file edits |
| **Codex** | Fast implementation passes in real repos | Boilerplate, broad refactors, test generation |
| [Vercel](https://vercel.com) | Hosting, previews, production | Next.js / edge-friendly backends; PR previews |
| [Supabase](https://supabase.com) | Postgres, auth, RLS, edge functions | Default data plane until you outgrow it |
| [GitHub](https://github.com) | Source of truth, review, light CI | Collaboration and shipping discipline |

## Pattern index

| Pattern | When to use it | Anti-patterns |
|--------|----------------|---------------|
| [Architecture planning prompts](patterns/architecture-planning.md) | Before coding; locking contracts between services | One-shot “build me an app” without constraints |
| [Effective code generation](patterns/code-generation.md) | Boilerplate, CRUD, repetitive modules | Generating core IP or security-critical paths untested |
| [Debugging with AI](patterns/debugging.md) | Log traces, repro steps, hypothesis churn | Pasting secrets; accepting fixes you don’t understand |
| [Database schema design](patterns/supabase-schema.md) | New features needing clear data model | Over-normalizing for v0; skipping migrations discipline |
| [Row-level security](patterns/supabase-rls.md) | Multi-tenant or user-scoped data | RLS as the only auth story; no service-role hygiene |
| [Edge functions](patterns/supabase-edge-functions.md) | Webhooks, light orchestration, glue code | Long-running jobs; heavy CPU in edge |
| [Quiz → recommendation](patterns/quiz-recommendation.md) | Guided selling; personalization | Opaque scoring; no eval of recommendations |
| [Subscription billing](patterns/subscription-billing.md) | Recurring revenue flows | Home-grown billing when Stripe + patterns suffice |
| [User authentication](patterns/user-auth.md) | Sessions, magic links, OAuth | Rolling crypto by hand |

## Philosophy

### Speed over perfection

AI-generated code isn't always optimal—but it’s *fast*. Generate a working version, then optimize what matters.

### Own your core logic

Use AI for boilerplate and standard patterns. Design and review your competitive advantage carefully.

### Iterate in production

With cheap deployments and good observability, production becomes a learning environment. Ship small changes frequently.

## Examples

This repo includes example implementations. They're intentionally simple — the goal is to understand the pattern, not copy-paste production code.

## License

MIT — use however you want. Attribution appreciated but not required.

## Connect

- [LinkedIn](https://linkedin.com/in/bcho)  
- [GitHub](https://github.com/brianyoungilcho)  
- [whynow.ai](https://whynow.ai)  
