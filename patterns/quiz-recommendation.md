# Quiz to Recommendation Flow

Multi-step quiz with personalized results. Common pattern for e-commerce, lead gen, and personalization.

## The Pattern

```
[Quiz Start]
     ↓
[Question 1] → [Question 2] → ... → [Question N]
     ↓
[Processing / Loading State]
     ↓
[Personalized Results]
     ↓
[Call to Action]
```

## Data Model

```sql
-- Quiz configuration (optional, for dynamic quizzes)
create table quizzes (
  id uuid default gen_random_uuid() primary key,
  name text not null,
  questions jsonb not null,  -- Array of question objects
  active boolean default true,
  created_at timestamptz default now() not null
);

-- User responses
create table quiz_responses (
  id uuid default gen_random_uuid() primary key,
  quiz_id uuid references quizzes(id),
  user_id uuid references profiles(id),  -- null for anonymous
  session_id text not null,  -- Track anonymous users
  responses jsonb not null,
  recommendations jsonb,  -- Computed results
  completed_at timestamptz,
  created_at timestamptz default now() not null
);

create index quiz_responses_session_idx on quiz_responses(session_id);
```

## Frontend Implementation

### Quiz State Management

```typescript
// types.ts
interface Question {
  id: string;
  type: 'single' | 'multiple' | 'scale' | 'text';
  question: string;
  options?: { id: string; label: string; value: any }[];
  required: boolean;
}

interface QuizState {
  currentStep: number;
  answers: Record<string, any>;
  isComplete: boolean;
  isLoading: boolean;
}
```

```typescript
// useQuiz.ts
import { useState } from 'react';

export function useQuiz(questions: Question[]) {
  const [state, setState] = useState<QuizState>({
    currentStep: 0,
    answers: {},
    isComplete: false,
    isLoading: false,
  });

  const currentQuestion = questions[state.currentStep];
  const progress = ((state.currentStep + 1) / questions.length) * 100;

  const answer = (questionId: string, value: any) => {
    setState(prev => ({
      ...prev,
      answers: { ...prev.answers, [questionId]: value },
    }));
  };

  const next = () => {
    if (state.currentStep < questions.length - 1) {
      setState(prev => ({ ...prev, currentStep: prev.currentStep + 1 }));
    } else {
      setState(prev => ({ ...prev, isComplete: true }));
    }
  };

  const back = () => {
    if (state.currentStep > 0) {
      setState(prev => ({ ...prev, currentStep: prev.currentStep - 1 }));
    }
  };

  return {
    currentQuestion,
    progress,
    answers: state.answers,
    isComplete: state.isComplete,
    answer,
    next,
    back,
  };
}
```

### Quiz Component

```tsx
// Quiz.tsx
export function Quiz({ questions, onComplete }: QuizProps) {
  const { currentQuestion, progress, answers, isComplete, answer, next, back } = useQuiz(questions);

  if (isComplete) {
    return <QuizResults answers={answers} onComplete={onComplete} />;
  }

  return (
    <div className="max-w-lg mx-auto p-6">
      {/* Progress Bar */}
      <div className="h-2 bg-gray-200 rounded-full mb-8">
        <div 
          className="h-full bg-blue-600 rounded-full transition-all"
          style={{ width: `${progress}%` }}
        />
      </div>

      {/* Question */}
      <h2 className="text-2xl font-bold mb-6">{currentQuestion.question}</h2>

      {/* Options */}
      <div className="space-y-3">
        {currentQuestion.options?.map(option => (
          <button
            key={option.id}
            onClick={() => {
              answer(currentQuestion.id, option.value);
              next();
            }}
            className="w-full p-4 text-left border rounded-lg hover:border-blue-600 
                       hover:bg-blue-50 transition-colors"
          >
            {option.label}
          </button>
        ))}
      </div>

      {/* Navigation */}
      <div className="flex justify-between mt-8">
        <button 
          onClick={back}
          className="text-gray-600 hover:text-gray-900"
        >
          Back
        </button>
      </div>
    </div>
  );
}
```

## Recommendation Engine

### Simple Scoring

```typescript
// recommendation.ts
interface Product {
  id: string;
  name: string;
  tags: string[];
  scores: Record<string, number>;  // e.g., { "active": 8, "senior": 3 }
}

export function getRecommendations(
  answers: Record<string, any>,
  products: Product[]
): Product[] {
  const scored = products.map(product => {
    let score = 0;

    // Score based on answer matches
    if (answers.activity_level === 'high' && product.scores.active) {
      score += product.scores.active;
    }
    if (answers.age === 'senior' && product.scores.senior) {
      score += product.scores.senior;
    }
    // ... more scoring rules

    return { product, score };
  });

  return scored
    .sort((a, b) => b.score - a.score)
    .slice(0, 3)
    .map(s => s.product);
}
```

### Edge Function for Complex Logic

```typescript
// supabase/functions/recommendations/index.ts
import { serve } from 'https://deno.land/std@0.168.0/http/server.ts'
import { createClient } from 'https://esm.sh/@supabase/supabase-js@2'

serve(async (req) => {
  const { answers, sessionId } = await req.json();
  
  const supabase = createClient(
    Deno.env.get('SUPABASE_URL')!,
    Deno.env.get('SUPABASE_SERVICE_ROLE_KEY')!
  );

  // Get products
  const { data: products } = await supabase
    .from('products')
    .select('*')
    .eq('active', true);

  // Score and rank
  const recommendations = scoreProducts(answers, products);

  // Save response
  await supabase.from('quiz_responses').insert({
    session_id: sessionId,
    responses: answers,
    recommendations: recommendations.map(p => p.id),
    completed_at: new Date().toISOString(),
  });

  return new Response(
    JSON.stringify({ recommendations }),
    { headers: { 'Content-Type': 'application/json' } }
  );
});
```

## Conversion Optimization

### Key Metrics

Track at each step:
- Step completion rate
- Drop-off points
- Time per question
- Final conversion rate

### Quick Wins

1. **Progress indicator** — Users complete more when they see progress
2. **12 questions max** — Beyond this, completion drops significantly
3. **Big tap targets** — Mobile users need 44px+ buttons
4. **Auto-advance** — Move to next question on single-select without "Next" button
5. **Loading state with personality** — "Analyzing your answers..." feels better than a spinner

### Results Page Structure

```
[Personalized Headline]
"Based on your answers, here's what we recommend for [Dog Name]"

[Top Recommendation - Highlighted]
- Why it's #1 for them
- Key benefits
- Price + CTA

[Alternative Options]
- 2-3 other good fits

[Social Proof]
- Reviews from similar customers

[Clear CTA]
- "Start Subscription" or "Add to Cart"
```
