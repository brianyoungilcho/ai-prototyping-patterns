# Architecture Planning Prompts

How to use Claude for system design before writing code.

## Why Plan First

Jumping straight into code generation leads to:
- Architectural inconsistencies
- Repeated refactoring
- Components that don't fit together

10 minutes of architecture planning saves hours of debugging.

## The Planning Prompt Template

```
I'm building [product description].

Target users: [who]
Core functionality: [what it does]
Technical constraints: [any requirements]

I want to use [your stack - e.g., React, Supabase, Stripe].

Help me design:
1. Data model (entities and relationships)
2. API structure (endpoints and their purposes)
3. Auth flow (how users sign up/in, what they can access)
4. Key user flows (step by step)

Ask clarifying questions if needed before providing the design.
```

## Example: Subscription E-Commerce

**Prompt:**
```
I'm building a dog food subscription service.

Target users: Dog owners who want personalized food recommendations
Core functionality: 
- Quiz about their dog (breed, age, health conditions)
- Personalized food recommendations based on quiz
- Monthly subscription with recurring billing

Technical constraints:
- Need to launch MVP in 2 weeks
- Using Supabase for backend, Stripe for payments
- Mobile-first design

Help me design:
1. Data model
2. API structure  
3. Auth flow
4. Key user flows
```

**Claude's response will include:**

### Data Model
```
users
  - id (uuid, primary key)
  - email
  - created_at

dogs
  - id (uuid, primary key)
  - user_id (foreign key → users)
  - name
  - breed
  - age_years
  - weight_lbs
  - activity_level (enum: low, medium, high)
  - health_conditions (array)
  - created_at

quiz_responses
  - id (uuid, primary key)
  - dog_id (foreign key → dogs)
  - responses (jsonb)
  - recommended_products (array)
  - created_at

subscriptions
  - id (uuid, primary key)
  - user_id (foreign key → users)
  - dog_id (foreign key → dogs)
  - stripe_subscription_id
  - product_id
  - status (enum: active, paused, cancelled)
  - next_delivery_date
  - created_at
```

### API Structure
```
POST /api/quiz/submit
  → Saves quiz responses, returns recommendations

POST /api/subscription/create
  → Creates Stripe subscription, stores in DB

POST /api/webhooks/stripe
  → Handles subscription lifecycle events

GET /api/user/subscriptions
  → Returns user's active subscriptions
```

## Follow-Up Questions

After the initial design, ask:

**For edge cases:**
> "What happens if a user has multiple dogs? How should the data model handle this?"

**For security:**
> "What row-level security policies do I need so users can only see their own data?"

**For scalability:**
> "If I get 10x the expected traffic, what's the first thing that will break?"

## Anti-Patterns

### Too Vague
❌ "Help me build a subscription app"

✅ "Help me build a subscription service for personalized dog food. Users take a quiz, get recommendations, and subscribe to monthly deliveries."

### No Constraints
❌ "Design the best possible architecture"

✅ "Design an architecture I can build solo in 2 weeks using Supabase and Stripe"

### Skipping the Conversation
❌ Taking the first response and implementing it

✅ Asking follow-up questions, challenging assumptions, iterating on the design

## Output Format

Ask Claude to output in formats you can directly use:

**For database schemas:**
> "Output the schema as Supabase SQL migrations"

**For API design:**
> "Output the API as TypeScript type definitions"

**For user flows:**
> "Output as a numbered step-by-step list with the database operations at each step"
