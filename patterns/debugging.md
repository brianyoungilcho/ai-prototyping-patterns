# Debugging with AI

Using Claude/Cursor to fix errors faster.

## The Debugging Workflow

```
Error occurs
     ↓
Gather context (error message, stack trace, relevant code)
     ↓
Prompt Claude with full context
     ↓
Understand the explanation (don't just copy-paste fix)
     ↓
Apply fix
     ↓
Verify fix worked
```

## Effective Debug Prompts

### Basic Pattern

```
I'm getting this error:

[Full error message with stack trace]

Here's the relevant code:

[Code that's causing the error]

Here's the context:
- Using [framework/library versions]
- This code is supposed to [what it should do]
- It was working before I [recent change]

What's wrong and how do I fix it?
```

### Example: React/Supabase Error

```
I'm getting this error:

Error: Cannot read properties of null (reading 'id')
    at UserProfile (UserProfile.tsx:15:23)
    at renderWithHooks (react-dom.development.js:14985:18)

Here's the code:

```tsx
export function UserProfile() {
  const { user } = useAuth();
  
  return (
    <div>
      <h1>Welcome, {user.id}</h1>  {/* Line 15 */}
    </div>
  );
}
```

Context:
- Using React 18, Supabase Auth
- useAuth() returns { user, loading }
- Works after login, crashes on page refresh

What's wrong?
```

Claude will explain:
- `user` is null during initial load (before auth state is determined)
- Need to handle the loading/null state
- Will provide a fixed version with proper guards

## Common Error Categories

### 1. Async/Timing Issues

**Symptoms:**
- "Cannot read property of undefined/null"
- Works sometimes, fails other times
- Works in dev, fails in production

**Prompt addition:**
```
Is this a timing/async issue? Show me how to properly handle 
the loading state.
```

### 2. Type Errors (TypeScript)

**Symptoms:**
- Red squiggles in editor
- "Type X is not assignable to type Y"

**Prompt addition:**
```
Here's my type definitions:
[paste relevant types]

What's the type mismatch and how do I fix it?
```

### 3. Database/Query Errors

**Symptoms:**
- Empty results when expecting data
- Constraint violations
- Permission denied

**Prompt addition:**
```
Here's my database schema:
[paste relevant tables]

Here's my RLS policies:
[paste policies]

The query returns [what it returns] but I expected [what you expected].
```

### 4. Build/Deploy Errors

**Symptoms:**
- Works locally, fails in CI/production
- Module not found
- Environment variable issues

**Prompt addition:**
```
Works locally but fails in production.
Build command: [command]
Node version: [version]
Environment: [Vercel/Netlify/etc]
```

## Advanced Debugging

### Rubber Duck Debugging with AI

When you're stuck, explain the problem in detail:

```
I'm trying to [goal].

Here's my current approach:
1. [step 1]
2. [step 2]
3. [step 3]

At step 2, I expected [X] but got [Y].

I've already tried:
- [thing 1]
- [thing 2]

What am I missing?
```

Often, writing this out reveals the issue. If not, Claude will spot it.

### Trace Through Logic

```
Here's my function:
[code]

Walk me through what happens when it's called with this input:
[specific input]

What's the value of each variable at each step?
```

### Compare Working vs. Broken

```
This code works:
[working code]

This similar code doesn't:
[broken code]

What's the difference that causes the failure?
```

## When AI Can't Help

### Missing Context

AI can only work with what you give it. If the error depends on:
- Runtime state you haven't captured
- Environment configuration you haven't shared
- Code in files you haven't shown

You need to gather more context first.

### Non-Deterministic Issues

Race conditions, memory leaks, and intermittent failures are hard to debug via chat. You'll need:
- Logging to capture state when it fails
- Profiling tools for performance issues
- Debugger for stepping through code

Share your findings with Claude for analysis.

### Library Bugs

Sometimes the bug is in a dependency. If Claude's fixes don't work and you're using the library correctly, check:
- GitHub issues for the library
- Recent version changes
- Known incompatibilities

## Debugging Mindset

### Understand, Don't Just Fix

When Claude provides a fix:
1. Read the explanation
2. Make sure you understand *why* it works
3. Ask follow-up questions if unclear

Blindly applying fixes leads to more bugs later.

### Reproduce First

Before debugging, create a minimal reproduction:
1. Smallest code that shows the bug
2. Clear steps to trigger it
3. Expected vs. actual behavior

This often reveals the issue without AI help.

### One Change at a Time

When applying fixes:
1. Make one change
2. Test
3. If not fixed, revert and try next option

Don't apply multiple fixes at once — you won't know which worked.
