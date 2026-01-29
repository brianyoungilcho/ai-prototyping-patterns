# AI Prototyping Patterns

Practical patterns for building full-stack products with AI tools. Based on real experience shipping products like [Pupsday.com](https://pupsday.com).

## Who This Is For

Solo founders and small teams who want to:
- Ship products faster using AI tools
- Build full-stack applications without a large team
- Understand when to use AI vs. traditional approaches

## The Stack

| Tool | Role | When to Use |
|------|------|-------------|
| [Claude](https://claude.ai) | Architecture, code generation, debugging | Complex reasoning, multi-file changes |
| [Cursor](https://cursor.sh) | AI-augmented IDE | Daily development workflow |
| [Lovable](https://lovable.dev) | UI prototyping | Initial design exploration |
| [Supabase](https://supabase.com) | Backend (DB, Auth, Edge Functions) | Most full-stack projects |
| [Vercel](https://vercel.com) | Deployment | React/Next.js projects |

## Patterns

### Prompting & Code Generation
- [Architecture Planning Prompts](patterns/architecture-planning.md) — How to use Claude for system design
- [Effective Code Generation](patterns/code-generation.md) — Getting better output from AI coding tools
- [Debugging with AI](patterns/debugging.md) — Using Claude to fix errors faster

### Supabase Patterns
- [Database Schema Design](patterns/supabase-schema.md) — Data modeling for common use cases
- [Row-Level Security](patterns/supabase-rls.md) — Securing data with Postgres policies
- [Edge Functions](patterns/supabase-edge-functions.md) — Custom backend logic

### Full-Stack Recipes
- [Quiz to Recommendation Flow](patterns/quiz-recommendation.md) — Multi-step quiz with personalized results
- [Subscription Billing](patterns/subscription-billing.md) — Stripe integration with Supabase
- [User Authentication](patterns/user-auth.md) — Auth flows and session management

## Philosophy

### Speed Over Perfection

AI-generated code isn't always optimal. But it's *fast*. Generate a working version, then optimize the parts that matter.

### Own Your Core Logic

Use AI for boilerplate and standard patterns. Design and review your competitive advantage carefully.

### Iterate in Production

With cheap deployments, production becomes your testing environment. Ship small changes frequently.

## Examples

This repo includes example implementations. They're intentionally simple — the goal is to understand the pattern, not copy-paste production code.

## License

MIT — use however you want. Attribution appreciated but not required.

## Connect

- [LinkedIn](https://linkedin.com/in/bcho)
- [GitHub](https://github.com/brianyoungilcho)
- [whynow.ai](https://whynow.ai)
