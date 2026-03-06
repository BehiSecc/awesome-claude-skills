---
name: recommendations
description: 'TasteRay API integration for personalized recommendations across verticals
  (movies, restaurants, products, travel, jobs). Use when you need to: (1) recommend
  movies, restaurants, products, travel, or jobs, (2) answer "what would I like" questions,
  (3) provide personalized recommendations based on preferences, (4) rank or score
  items for a user, (5) explain why something matches a user''s taste, (6) build recommendation
  context from conversation, (7) integrate psychological profiles with recommendation
  systems.'
license: MIT
metadata:
  author: tasteray
  version: "1.0"
---

# Recommendations

Personalized recommendations through the TasteRay API.

## Goal

When making recommendations or reviewing recommendation-related code—whether API integrations, context building, or presentation logic—**your goal is to achieve a 10/10 score**.

Score all work on a 0-10 scale based on adherence to the principles and techniques in this skill. Provide your assessment as **X/10** with specific feedback on what's working and what needs improvement to reach 10/10.

A 10/10 means the work:
- Embodies the core principle (understanding precedes recommendation)
- Builds rich context before calling the API
- Presents recommendations with personalized explanations
- Handles edge cases gracefully (low confidence, rate limits, errors)
- Avoids all anti-patterns

Iterate until you reach 10/10.

---

## Core Principle

**Understanding precedes recommendation.**

Great recommendations come from deep understanding of the person—their preferences, constraints, history, and context. Never call the API without first building meaningful context from the conversation.

Key insight: A recommendation is only as good as the context that informed it.

---

## API Overview

The TasteRay Recommendation API provides personalized recommendations across multiple verticals.

### Base URL

```
https://api.tasteray.com
```

### Authentication

All requests require an API key in the header:

```
X-API-Key: your-api-key
```

### Core Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/v1/recommend` | POST | Get personalized recommendations |
| `/v1/explain` | POST | Get detailed explanation for a single item |
| `/v1/usage` | GET | Check quota and usage statistics |

See: [API Reference](./references/api-reference.md)

---

## The Recommendation Flow

Every recommendation follows this pattern:

```
1. Build context from conversation
   ↓
2. Call POST /v1/recommend
   ↓
3. Interpret confidence scores
   ↓
4. Present with personalized explanations
   ↓
5. Iterate based on feedback
```

### Step 1: Build Context

Extract from the conversation:

- **preferences**: Explicit likes/dislikes and taste patterns
- **profile**: Psychological profile text (from elicitation skill)
- **constraints**: Hard requirements (budget, dietary, location, etc.)
- **history**: Past items they've liked or disliked

### Step 2: Call the API

```json
POST /v1/recommend
{
  "vertical": "movies",
  "context": {
    "preferences": ["dark comedies", "complex characters"],
    "profile": "Values authenticity and depth. Drawn to stories about outsiders...",
    "constraints": {"max_length_minutes": 120},
    "history": [
      {"item": "Parasite", "rating": 5},
      {"item": "The Lobster", "rating": 4}
    ]
  },
  "count": 5
}
```

### Step 3: Interpret Confidence

| Score | Meaning | Action |
|-------|---------|--------|
| 0.9+ | Strong match | Lead with this recommendation |
| 0.7-0.9 | Good match | Present with confidence |
| 0.5-0.7 | Moderate match | Include caveats, explain why it might not fit |
| <0.5 | Weak match | Omit or explain significant uncertainty |

### Step 4: Present Recommendations

Don't just list items. Personalize the explanation:

**Bad:**
> "Based on your preferences, I recommend Parasite."

**Good:**
> "Given your appreciation for dark comedies with complex characters, Parasite would be a strong match. It has that same outsider perspective you responded to in The Lobster, but with sharper social commentary."

### Step 5: Iterate

Use feedback to refine:
- "That's not quite right" → Ask what's missing
- "Tell me more about X" → Call `/v1/explain`
- "Something different" → Adjust constraints and re-query

---

## Context Building

The quality of recommendations depends entirely on context quality.

### Preferences Array

Explicit taste statements extracted from conversation:

```json
"preferences": [
  "dark comedies",
  "complex anti-heroes",
  "slow-burn narratives",
  "dislikes: jump scares",
  "dislikes: predictable endings"
]
```

**Extraction patterns:**
- "I love X" → positive preference
- "I can't stand X" → negative preference (prefix with "dislikes:")
- "Something like X" → reference point
- "Not too X" → constraint or negative preference

### Profile Text

Free-form psychological profile. If using the elicitation skill, summarize findings:

```json
"profile": "Values authenticity over polish. Drawn to outsider narratives
and stories about people who don't fit in. Appreciates moral ambiguity
and complex characters who aren't clearly good or bad. Responds strongly
to themes of class and social hierarchy. Prefers films that trust the
audience's intelligence."
```

### Constraints Object

Hard requirements that filter recommendations:

| Vertical | Common Constraints |
|----------|-------------------|
| Movies | `max_length_minutes`, `release_year_min`, `exclude_genres` |
| Restaurants | `cuisine`, `price_range`, `location`, `dietary` |
| Products | `max_price`, `category`, `brand_exclude` |
| Travel | `budget`, `dates`, `accessibility`, `interests` |
| Jobs | `location`, `salary_min`, `remote`, `experience_level` |

### History Array

Past items with ratings (1-5 scale):

```json
"history": [
  {"item": "The Grand Budapest Hotel", "rating": 5},
  {"item": "Transformers", "rating": 1},
  {"item": "Moonlight", "rating": 4}
]
```

**Guidelines:**
- Include both positive and negative examples
- Maximum 50 items (API limit)
- More recent items are weighted higher
- Include item identifiers when available (IMDB ID, etc.)

See: [Context Patterns Reference](./references/context-patterns.md)

---

## Vertical-Specific Patterns

### Movies & TV

**Key context signals:**
- Genre preferences (and anti-preferences)
- Pacing preferences (slow burn vs. action-packed)
- Era preferences (classic, modern, contemporary)
- Language/subtitle comfort
- Mood they're seeking

**Constraint examples:**
```json
{
  "max_length_minutes": 120,
  "release_year_min": 2010,
  "exclude_genres": ["horror", "musical"],
  "languages": ["en", "fr"],
  "streaming_services": ["netflix", "hulu"]
}
```

### Restaurants

**Key context signals:**
- Cuisine preferences and adventurousness
- Price sensitivity
- Dietary restrictions (must capture accurately)
- Occasion context (date night, business, casual)
- Ambiance preferences

**Constraint examples:**
```json
{
  "location": {"lat": 37.7749, "lng": -122.4194, "radius_miles": 5},
  "price_range": [2, 3],
  "dietary": ["vegetarian-options"],
  "open_now": true
}
```

### Products

**Key context signals:**
- Use case and context
- Quality vs. price tradeoff
- Brand affinities and aversions
- Feature priorities
- Aesthetic preferences

**Constraint examples:**
```json
{
  "max_price": 500,
  "category": "headphones",
  "brand_exclude": ["beats"],
  "features_required": ["noise-cancelling", "wireless"]
}
```

### Travel

**Key context signals:**
- Travel style (adventure, relaxation, cultural)
- Comfort level with unfamiliarity
- Activity preferences
- Social context (solo, couple, family)
- Previous destinations (loved and disliked)

**Constraint examples:**
```json
{
  "budget_per_day": 200,
  "dates": {"start": "2024-06-01", "end": "2024-06-14"},
  "accessibility": ["wheelchair-accessible"],
  "climate": "warm",
  "visa_free_for": "US"
}
```

### Jobs

**Key context signals:**
- Career values and priorities
- Work style preferences
- Growth vs. stability orientation
- Culture fit signals
- Skills and experience level

**Constraint examples:**
```json
{
  "location": "San Francisco",
  "remote": "hybrid",
  "salary_min": 150000,
  "experience_level": "senior",
  "company_size": ["startup", "mid"]
}
```

---

## Presentation Patterns

How you present recommendations matters as much as what you recommend.

### Confidence-Based Presentation

**High confidence (0.9+):**
> "This is a strong match for you. [Item] aligns well with your preference for [specific trait]."

**Good confidence (0.7-0.9):**
> "[Item] should work well. It has [positive trait], though it's [caveat] which might not be exactly what you're looking for."

**Moderate confidence (0.5-0.7):**
> "This one's a bit of a stretch, but [Item] might surprise you. It's [trait], which you haven't explicitly mentioned, but based on [pattern] you might appreciate it."

**Low confidence (<0.5):**
> Generally omit, or: "If you're feeling adventurous, [Item] is quite different from your usual preferences, but [reason for including]."

### Explanation Structure

For each recommendation, explain:

1. **The match** - Why this fits their stated preferences
2. **The connection** - How it relates to items they've liked
3. **The caveat** - Any potential misalignment (if applicable)

### Handling Feedback

**"Tell me more"** → Use `/v1/explain` endpoint:
```json
POST /v1/explain
{
  "vertical": "movies",
  "item": "Parasite",
  "context": { /* same context */ }
}
```

**"Not quite right"** → Ask clarifying questions:
> "What about that recommendation missed the mark? Too [X]? Not enough [Y]?"

**"Something completely different"** → Adjust approach:
> "Let me try a different angle. If we set aside [previous focus], what are you actually in the mood for?"

See: [Presentation Patterns Reference](./references/presentation-patterns.md)

---

## Integration with Elicitation

The elicitation skill builds psychological profiles. Use these for richer recommendations.

### From Elicitation to Profile

**Self-defining memories reveal:**
- Formative experiences that shape taste
- Emotional patterns that resonate in content
- Identity themes (agency, communion, redemption)

**Values elicitation reveals:**
- Core priorities that should be reflected in recommendations
- What makes something feel "right"

**Schema detection reveals:**
- Patterns that might influence reception
- Topics or themes to handle carefully

### Profile Construction

After elicitation, construct a profile string:

```json
"profile": "Core value: authenticity. Drawn to narratives about
self-reinvention (redemption themes in life story). Responds to
understated emotion over melodrama. Achievement-oriented but values
creative expression over status markers. Formative memory involves
finding belonging through shared creative work—look for collaborative
or community themes."
```

### Example Integration

1. Use elicitation skill to understand the person
2. Summarize findings in profile text
3. Call recommendation API with rich context
4. Present recommendations that connect to their story

---

## Error Handling

### Rate Limits

The API returns `429 Too Many Requests` when limits are exceeded.

| Tier | Requests/min | Requests/day |
|------|--------------|--------------|
| Free | 10 | 100 |
| Pro | 60 | 5,000 |
| Enterprise | 300 | Unlimited |

**Handling:**
- Check `Retry-After` header for wait time
- Queue requests if hitting limits
- Consider caching frequent queries

### Authentication Errors

`401 Unauthorized` - Invalid or missing API key.

**Handling:**
- Verify API key is set correctly
- Check key hasn't expired
- Ensure key has appropriate permissions

### Server Errors

`500+` - Server-side issues.

**Handling:**
- Implement exponential backoff
- Have fallback behavior (graceful degradation)
- Log for debugging

See: [API Reference](./references/api-reference.md) for complete error codes.

---

## Anti-Patterns

**What NOT to do:**

### The Empty Context Call
Calling the API without building context first.

Instead: Always gather preferences, constraints, and history before requesting recommendations.

### The Raw Dump
Presenting API response directly without personalization.

Instead: Interpret confidence scores and craft explanations that connect to the person's stated preferences.

### The Overconfident Low-Score
Presenting weak matches (confidence < 0.5) with the same confidence as strong ones.

Instead: Either omit low-confidence recommendations or clearly caveat them.

### The Ignored Constraint
Missing or misremembering stated constraints (budget, dietary, etc.).

Instead: Capture constraints explicitly and verify before each API call.

### The Context Amnesia
Not maintaining context across conversation turns.

Instead: Accumulate preferences, history, and constraints throughout the conversation.

### The Explanation Vacuum
Recommending without explaining why.

Instead: Every recommendation should include a personalized explanation connecting to stated preferences.

### The Feedback Ignore
Not using negative feedback to improve subsequent recommendations.

Instead: When something misses, ask what went wrong and adjust context accordingly.

---

## References

Detailed guides:

- [API Reference](./references/api-reference.md) - Full endpoint documentation, parameters, responses
- [Context Patterns](./references/context-patterns.md) - Building effective recommendation context
- [Presentation Patterns](./references/presentation-patterns.md) - Confidence interpretation, explanation structure

---

## Quick Reference

### Minimum Viable Request

```json
POST /v1/recommend
Headers: X-API-Key: your-key

{
  "vertical": "movies",
  "context": {
    "preferences": ["at least one preference"]
  },
  "count": 5
}
```

### Full Request

```json
POST /v1/recommend
{
  "vertical": "movies",
  "context": {
    "preferences": ["dark comedies", "complex characters"],
    "profile": "Values authenticity. Drawn to outsider narratives...",
    "constraints": {
      "max_length_minutes": 120,
      "release_year_min": 2010
    },
    "history": [
      {"item": "Parasite", "rating": 5, "id": "tt6751668"},
      {"item": "The Lobster", "rating": 4, "id": "tt3464902"}
    ]
  },
  "count": 5,
  "explain": true
}
```

### Confidence Quick Guide

| Score | Action |
|-------|--------|
| ≥0.9 | Lead recommendation, strong match |
| 0.7-0.9 | Include confidently |
| 0.5-0.7 | Include with caveats |
| <0.5 | Omit or heavily caveat |
