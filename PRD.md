# PRD — AI Learns AI

*A companion that writes prompts with you, predicts what you will get, and steers you to what you actually want.*

---

## 1. Positioning

Not a teacher. Not a tutorial. A **co-pilot for prompting**. Sits next to any AI tool you use, watches what you type, and helps you get the result you want on the first try.

Tagline: **"Write with you, not at you."**

## 2. Target users (one product, many hats)

| User | What they need |
|------|----------------|
| Student | Homework help, essay drafts, study plans |
| Business owner | Emails, marketing copy, market research |
| AI assistant / VA | Faster client work, bulk prompt runs |
| Developer | Better prompts for Cursor, Claude Code, Copilot |
| Curious beginner | A good answer without learning "prompting" |

One product. Adjusts tone and depth based on selected persona.

## 3. Problem

- People write short, vague prompts. They get bad answers. They blame the AI.
- Rewriting a prompt five times wastes tokens and time.
- No one knows what a "good prompt" looks like for their task.
- Different AI tools want different prompt styles.

## 4. Solution — three moves

### Move 1: Tour (once, at start)
Sixty-second fly-through of the AI tool the user opened. Not a lesson. Just "here is where things live." Skippable.

### Move 2: Write-along
User types a prompt in any AI app. Companion reads it live and shows a **side bubble** with:
- Enriched version (adds missing context, tone, format, constraints)
- One-line reason per change
- Accept / edit / ignore

Companion never sends. Companion only proposes.

### Move 3: Outcome preview
Before user hits Enter, companion shows a **one-paragraph preview** of the likely response. If preview looks wrong, user tweaks first. Saves tokens.

After real output arrives, companion diffs actual vs predicted. If gap is big, suggests a refinement line.

## 5. Core features (v1)

| # | Feature | What it does |
|---|---------|--------------|
| 1 | Push-to-talk + hover trigger | Speak or type rough idea |
| 2 | Prompt enricher | Rewrites prompt with best-practice structure |
| 3 | Outcome predictor | One-paragraph "expect this" preview |
| 4 | Gap fixer | Suggests refinement after real output |
| 5 | Tool-aware tips | Different tricks per AI app |
| 6 | Persona presets | student, founder, assistant, dev — tunes voice + depth |
| 7 | Memory | Past prompts, wins, style — gets smarter over time |
| 8 | Cheap mode | Free tier uses small model, paid uses bigger |

## 6. Intelligence layer

Three stacked models, cheapest first. Router picks the cheapest that can do the job.

```
[user types]
   ↓
[Haiku 4.5]      → enrich + predict          (fast, cheap, most calls)
   ↓
[user sends]
   ↓
[Sonnet 4.6]     → compare actual vs predicted, fix line   (only if gap)
   ↓
[Opus 4.7]       → high-stakes prompts only  (paid tier)
```

## 7. Tech implementation

### Reused from Clicky base
- Menu bar app, `LSUIElement=true`, no dock icon
- Screen capture via `ScreenCaptureKit`
- Full-screen transparent `NSPanel` overlay
- Cloudflare Worker proxy for API keys
- TLS warmup HEAD request
- Shared long-lived URLSession
- `AVAudioEngine` mic + AssemblyAI streaming STT
- ElevenLabs TTS via Worker

### New parts

| Part | Job |
|------|-----|
| Prompt reader | Accessibility API reads live text of front app text field |
| Enricher | Haiku call, structured JSON: `enriched_prompt`, `changes[]`, `reason[]` |
| Predictor | Same Haiku call adds `expected_output_summary` |
| Diff engine | Embed actual + predicted, cosine-diff, if > threshold trigger fix |
| Tool registry | JSON per app: prompt style, gotchas, best prefix |
| Persona tuner | Adds persona line into enricher system prompt |
| Memory store | Local SQLite: past prompts, tags, wins. No cloud. |

### Data flow (one cycle)

```
user types  →  accessibility API grabs text
            →  Haiku: enrich + predict
            →  side bubble shows both
            →  user accepts / edits
            →  user sends to AI tool
            →  accessibility API grabs response
            →  compare vs predicted
            →  gap? Haiku suggests fix line
```

No fine-tuning. Pure prompt engineering + routing.

## 8. Pricing

| Tier | Price | Model access | Limits |
|------|-------|--------------|--------|
| Free | $0 | Haiku only | 100 enrich / day |
| Student | $3/mo | Haiku + Sonnet | Unlimited enrich, 500 predict / day |
| Pro | $12/mo | All models + Opus | Unlimited |
| Team | $8/seat | Shared memory, brand voice | 5+ seats |

Free tier must feel great. That is the funnel.

## 9. Success metrics

- Prompt enrich acceptance rate: **>50%**
- Outcome prediction accuracy (user marks "matches"): **>70%**
- Free → paid conversion in 30 days: **8%**
- Daily prompts per active user: **>4**

## 10. Out of scope (v1)

- Auto-sending prompts
- Windows / Linux (Mac only)
- Building AI apps for users
- Fine-tuned custom models

## 11. Key risks + fixes

| Risk | Fix |
|------|-----|
| Prediction wrong often | Show confidence score, hide if low |
| Accessibility API blocked in some apps | Fallback: user pastes prompt into companion panel |
| Cost blows up | Hard cap per user, cache repeat enrichments, prefer Haiku |
| Feels intrusive | Trigger only on hover / hotkey, never auto-pop |
| Privacy fear | All memory local. Cloud calls proxied, no logs kept. |

## 12. Rollout

- Week 1-3: Fork Clicky base, strip pointing, add prompt reader
- Week 4-5: Enricher + predictor + side bubble UI
- Week 6: Persona presets + tool registry (top 5 AI apps)
- Week 7: Diff engine + fix suggestions
- Week 8: Beta with 50 users (10 per persona)
- Week 9-10: Iterate on acceptance rate
- Week 11: Public launch, free tier open

Done.
