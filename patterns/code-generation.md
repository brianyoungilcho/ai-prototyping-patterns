# Effective Code Generation

Getting better output from AI coding tools.

## The Core Problem

AI code generation often produces:
- Code that looks right but has subtle bugs
- Over-engineered solutions for simple problems
- Inconsistent patterns across files

Better prompting solves most of these issues.

## Prompting Principles

### 1. Provide Context

**Bad:**
```
Create a function to process orders
```

**Good:**
```
I'm building a subscription e-commerce site with Supabase and Stripe.

Create a function that:
- Takes an order object with { userId, productId, quantity }
- Validates the user exists in Supabase
- Creates a Stripe checkout session
- Returns the checkout URL

Use TypeScript. Handle errors gracefully.
```

### 2. Specify the Stack

AI tools know many frameworks. If you don't specify, you'll get generic solutions.

**Bad:**
```
Create a login form
```

**Good:**
```
Create a login form using:
- React with TypeScript
- Supabase Auth (signInWithPassword)
- Tailwind CSS for styling
- React Hook Form for form handling

Include loading state and error display.
```

### 3. Show Existing Patterns

If you have existing code, show it:

```
Here's my existing Button component:

[paste existing code]

Create a similar Input component following the same patterns for:
- TypeScript types
- Tailwind classes
- Prop structure
```

### 4. Ask for Explanations

When learning or reviewing:

```
Create [component] and explain:
- Why you chose this approach
- What trade-offs you made
- What I should watch out for
```

## Cursor-Specific Patterns

### Inline Edits

Select code, then Cmd+K with prompts like:
- "Add error handling"
- "Convert to TypeScript"
- "Add loading state"
- "Make this async"

### Multi-File Changes

Use Claude chat (Cmd+L) for changes spanning files:

```
I need to add a "last_login" field to users.

This requires:
1. Supabase migration to add the column
2. Update the User type in types.ts
3. Update the login function to set last_login
4. Display last_login in the profile component

Make all these changes.
```

### Debugging

Paste errors directly:

```
I'm getting this error:

[paste full error with stack trace]

Here's the relevant code:

[paste code]

What's wrong and how do I fix it?
```

## Quality Control

### Review Generated Code

Never blindly accept. Check for:
- **Security:** Are inputs validated? SQL injection possible?
- **Error handling:** What happens when things fail?
- **Edge cases:** Empty arrays, null values, missing data?
- **Performance:** N+1 queries? Unnecessary re-renders?

### Test Critical Paths

Generated code for:
- Payment processing
- Authentication
- Data mutations

Should always be manually tested, not just visually inspected.

### Understand Before Using

If you can't explain what the code does, don't ship it. Ask Claude to explain until you understand.

## Common Failure Modes

### Over-Abstraction

AI loves to create abstractions. For MVP code, push back:

```
This is too abstract. I only have one type of [thing]. 
Give me a simpler version without the factory pattern.
```

### Outdated Patterns

AI training data has a cutoff. For recent libraries:

```
I'm using Supabase v2 with the new @supabase/ssr package.
Here's the current API: [paste from docs]
Generate code using this API, not the old auth-helpers.
```

### Hallucinated APIs

AI will confidently use APIs that don't exist. Always verify:
- Function names against docs
- Parameter signatures
- Return types

## Speed Tips

### Generate Boilerplate

AI excels at repetitive code:
- CRUD operations
- Form validation schemas
- Type definitions from examples
- Test boilerplate

### Generate, Then Edit

Faster workflow:
1. Generate a first version (accept imperfection)
2. Run it, find issues
3. Fix specific issues with targeted prompts

### Parallel Generation

Need multiple components? Generate them in parallel:

```
Create these three components:
1. ProductCard - displays product image, name, price
2. ProductList - grid of ProductCards
3. ProductDetail - full product page

Output each as a separate code block.
```
