# PRD — AICamper

*The senior AI expert that camps beside you inside any AI tool. Handles the prompting, planning, and craft. You bring the intent.*

---

## 1. Positioning

Not a teacher. Not a tutorial. Not a chatbot. A **camper**. A senior AI expert that walks with you into any AI tool and does the technical work of talking to it.

Tagline: **"You bring the idea. Camper brings the craft."**

## 2. Target users

| User | Why they need AICamper |
|------|------------------------|
| Beginner | Never used AI. Freezes at blank prompt box. |
| Expert | Fluent in one tool. Needs to move to five more. |
| Pivoter | Jumping between AI tools weekly. Prompt style keeps breaking. |
| Business owner | Needs AI to ship real work, not read tutorials. |
| Student | Wants good answers without becoming a prompt nerd. |
| Assistant / VA | Bills by the hour. Cannot afford slow prompting. |

One product. Adjusts tone and depth based on selected persona.

## 3. Problem

- Every AI tool has its own prompt language and quirks
- Beginners freeze at the blank box
- Experts hit walls when they try a new tool
- Pivoters lose weeks relearning basics
- No one has time to "master prompting"

## 4. Solution — the camper flow

### Walk-in
User opens any AI tool. Camper detects the tool and warms up its expertise for that tool (loads the right prompt patterns, gotchas, and best structures).

### Plan
User types or speaks a rough idea. Camper reads it and quietly plans the full prompt: adds context, tone, format, constraints, examples if useful.

### Suggest
Side bubble shows the enriched prompt with one-line reasons per change. User accepts, edits, or ignores. Camper never sends.

### Predict
Before user hits send, camper shows a one-paragraph preview of the likely output. If preview looks wrong, user tweaks the prompt first. Saves tokens.

### Compare and fix
After real output arrives, camper diffs actual vs predicted. If gap is big, suggests a refinement line.

### Learn user
Camper remembers user style, past wins, favorite tone. Bubble gets sharper over time.

## 5. Core features (v1)

| # | Feature | What it does |
|---|---------|--------------|
| 1 | Push-to-talk + hover trigger | Speak or type your rough idea |
| 2 | Tool detector | Knows which AI app is open. Loads that tool's expertise. |
| 3 | Prompt planner | Full prompt scaffold from your rough input |
| 4 | Prompt enricher | Rewrites with context, tone, format, constraints |
| 5 | Outcome predictor | One-paragraph "expect this" preview |
| 6 | Gap fixer | Refinement suggestion after real output |
| 7 | Persona presets | beginner, expert, pivoter, founder, student, VA |
| 8 | Memory | Your prompts, wins, style. Local only. |
| 9 | Cheap-first router | Small model for most calls. Big model only when needed. |

## 6. Intelligence layer

Three stacked models, cheapest first. Router picks the smallest that can do the job.

```
[user rough idea]
   ↓
[Haiku 4.5]     → plan + enrich + predict          (fast, cheap, most calls)
   ↓
[user sends to their AI tool]
   ↓
[Sonnet 4.6]    → compare actual vs predicted      (only if gap)
   ↓
[Opus 4.7]      → hard high-stakes prompts         (paid tier)
```

Most user calls stay on Haiku. Cost per user stays low.

## 7. Tech implementation

### Reused from Clicky base
- Menu bar app, `LSUIElement=true`, no dock icon
- Screen capture via `ScreenCaptureKit`
- Full-screen transparent `NSPanel` overlay
- Cloudflare Worker proxy for API keys
- TLS warmup HEAD request
- Shared long-lived URLSession
- `AVAudioEngine` mic + streaming STT
- Bezier arc cursor animation

### New parts

| Part | Job |
|------|-----|
| Tool detector | Reads front app bundle ID, matches to AI tool registry |
| Prompt reader | Accessibility API reads live text of front app text field |
| Tool registry | JSON per AI tool: prompt style, gotchas, best prefix, common patterns |
| Prompt planner | Haiku call, structured JSON: `plan`, `enriched_prompt`, `changes[]`, `reasons[]` |
| Predictor | Same Haiku call adds `expected_output_summary`, `confidence` |
| Diff engine | Embed actual + predicted, cosine-diff, trigger fix if gap > threshold |
| Persona tuner | Adds persona line into planner system prompt |
| Memory store | Local SQLite: past prompts, tags, wins. No cloud. |

### Data flow (one cycle)

```
user opens AI tool
   → tool detector reads bundle ID
   → registry loads that tool's expertise
   → user types rough idea
   → accessibility API grabs text
   → Haiku: plan + enrich + predict
   → side bubble shows enriched + preview
   → user accepts / edits
   → user sends to AI tool
   → accessibility API grabs response
   → diff engine compares actual vs predicted
   → gap? Haiku suggests fix line
   → memory stores prompt + outcome
```

No model training. Pure prompt engineering + routing + tailored per-tool expertise.

## 8. Tool registry (v1 targets)

Top ten AI tools supported at launch:

1. ChatGPT (web + desktop)
2. Claude (web + desktop)
3. Gemini
4. Perplexity
5. Cursor
6. Claude Code
7. Copilot
8. Midjourney
9. Runway
10. ElevenLabs

Each entry: prompt style rules, common failure modes, best prefixes, example patterns.

## 9. Pricing

Cheapest-possible is a core belief.

| Tier | Price | Model access | Limits |
|------|-------|--------------|--------|
| Free | $0 | Haiku only | 100 plans / day |
| Student | $3/mo | Haiku + Sonnet | Unlimited plans, 500 predicts / day |
| Pro | $12/mo | All models + Opus | Unlimited |
| Team | $8/seat | Shared memory, brand voice, 5+ seats |

Free tier must feel great. That is the funnel.

## 10. Success metrics

- Prompt acceptance rate: **>50%**
- Outcome prediction accuracy (user marks "matches"): **>70%**
- Free → paid conversion in 30 days: **8%**
- Daily prompts per active user: **>4**
- Time from install to first accepted prompt: **<3 min**

## 11. Out of scope (v1)

- Auto-sending prompts
- Windows / Linux (Mac only)
- Building AI apps for users
- Fine-tuned custom models

## 12. Key risks + fixes

| Risk | Fix |
|------|-----|
| Prediction wrong often | Show confidence score, hide if low |
| Accessibility API blocked in some apps | Fallback: user pastes prompt into camper panel |
| Cost blows up | Hard cap per user, cache repeat enrichments, prefer Haiku |
| Feels intrusive | Trigger only on hover / hotkey, never auto-pop |
| Privacy fear | Memory local. Cloud calls proxied. No logs kept. |
| Tool registry rots as AI tools change | Registry updates ship monthly. Community contributions welcome. |

## 13. Rollout

- Week 1-3: Fork Clicky base, strip pointing, add tool detector + prompt reader
- Week 4-5: Prompt planner + enricher + predictor + side bubble UI
- Week 6: Persona presets + tool registry (top 5 AI apps)
- Week 7: Diff engine + fix suggestions + memory store
- Week 8: Beta with 50 users (mix of beginners, experts, pivoters)
- Week 9-10: Iterate on acceptance rate
- Week 11: Public launch, free tier open

Done.
