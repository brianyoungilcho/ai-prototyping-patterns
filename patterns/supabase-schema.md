# Database Schema Design

Data modeling patterns for Supabase projects.

## Schema Design Principles

### 1. Start Minimal

Don't model every possible future feature. Start with what you need for MVP.

**Week 1:** Users, core data, one relationship  
**Week 4:** Add features based on real usage

### 2. Use UUIDs

Supabase default. Better for distributed systems, no enumeration attacks.

```sql
id uuid default gen_random_uuid() primary key
```

### 3. Always Add Timestamps

```sql
created_at timestamptz default now() not null,
updated_at timestamptz default now() not null
```

Add a trigger to auto-update `updated_at`:

```sql
create or replace function update_updated_at()
returns trigger as $$
begin
  new.updated_at = now();
  return new;
end;
$$ language plpgsql;

create trigger update_users_updated_at
  before update on users
  for each row
  execute function update_updated_at();
```

### 4. Use Enums for Fixed Options

```sql
create type subscription_status as enum ('active', 'paused', 'cancelled');

create table subscriptions (
  id uuid default gen_random_uuid() primary key,
  status subscription_status default 'active' not null
);
```

### 5. JSONB for Flexible Data

When structure varies or you're exploring:

```sql
create table quiz_responses (
  id uuid default gen_random_uuid() primary key,
  user_id uuid references users(id) not null,
  responses jsonb not null,  -- flexible structure
  created_at timestamptz default now() not null
);
```

Query JSONB:
```sql
select * from quiz_responses 
where responses->>'dog_breed' = 'golden_retriever';
```

## Common Patterns

### User Profiles

Supabase Auth creates `auth.users`. Extend with a public profile:

```sql
create table profiles (
  id uuid references auth.users(id) primary key,
  display_name text,
  avatar_url text,
  created_at timestamptz default now() not null,
  updated_at timestamptz default now() not null
);

-- Auto-create profile on signup
create or replace function handle_new_user()
returns trigger as $$
begin
  insert into public.profiles (id)
  values (new.id);
  return new;
end;
$$ language plpgsql security definer;

create trigger on_auth_user_created
  after insert on auth.users
  for each row
  execute function handle_new_user();
```

### One-to-Many

User has many orders:

```sql
create table orders (
  id uuid default gen_random_uuid() primary key,
  user_id uuid references profiles(id) not null,
  total_cents integer not null,
  status order_status default 'pending' not null,
  created_at timestamptz default now() not null
);

create index orders_user_id_idx on orders(user_id);
```

### Many-to-Many

Users can have multiple tags, tags can have multiple users:

```sql
create table tags (
  id uuid default gen_random_uuid() primary key,
  name text not null unique
);

create table user_tags (
  user_id uuid references profiles(id) on delete cascade,
  tag_id uuid references tags(id) on delete cascade,
  primary key (user_id, tag_id)
);
```

### Soft Deletes

Keep data but mark as deleted:

```sql
create table products (
  id uuid default gen_random_uuid() primary key,
  name text not null,
  deleted_at timestamptz,  -- null = active
  created_at timestamptz default now() not null
);

-- Query only active
select * from products where deleted_at is null;
```

## Subscription E-Commerce Example

```sql
-- Extend auth.users
create table profiles (
  id uuid references auth.users(id) primary key,
  email text not null,
  stripe_customer_id text,
  created_at timestamptz default now() not null
);

-- User's dogs (one-to-many)
create table dogs (
  id uuid default gen_random_uuid() primary key,
  profile_id uuid references profiles(id) on delete cascade not null,
  name text not null,
  breed text,
  birth_date date,
  weight_lbs numeric,
  activity_level text check (activity_level in ('low', 'medium', 'high')),
  health_conditions text[],
  created_at timestamptz default now() not null
);

create index dogs_profile_id_idx on dogs(profile_id);

-- Products
create table products (
  id uuid default gen_random_uuid() primary key,
  name text not null,
  description text,
  price_cents integer not null,
  suitable_for jsonb,  -- { "min_weight": 10, "max_weight": 50, "activity": ["medium", "high"] }
  active boolean default true,
  created_at timestamptz default now() not null
);

-- Subscriptions
create type subscription_status as enum ('active', 'paused', 'cancelled');

create table subscriptions (
  id uuid default gen_random_uuid() primary key,
  profile_id uuid references profiles(id) not null,
  dog_id uuid references dogs(id) not null,
  product_id uuid references products(id) not null,
  stripe_subscription_id text not null,
  status subscription_status default 'active' not null,
  current_period_end timestamptz,
  created_at timestamptz default now() not null
);

create index subscriptions_profile_id_idx on subscriptions(profile_id);
create index subscriptions_stripe_id_idx on subscriptions(stripe_subscription_id);
```

## Migration Tips

### Use Supabase Migrations

```bash
supabase migration new create_users_table
```

Creates a timestamped file in `supabase/migrations/`.

### One Change Per Migration

**Good:**
- `20240115_create_users.sql`
- `20240116_add_user_avatar.sql`

**Bad:**
- `20240115_create_everything.sql`

### Test Locally First

```bash
supabase db reset  # Runs all migrations from scratch
```

Catches migration errors before production.
