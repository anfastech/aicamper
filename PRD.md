# PRD — AICamper

*The senior AI expert that camps beside you inside any AI tool. Autopilots the prompting, tool-picking, workflow chaining, and accuracy checking. You bring the intent. Optional Course Mode for those who want to learn too.*

---

## 1. Positioning

Not another AI bootcamp. Not a "learn to prompt" course. **The bootcamp lives inside your tool and does the work for you.**

A camper that walks with you into any AI app and autopilots the four hard skills: prompt engineering, tool selection, workflow automation, accuracy checks. Course Mode is optional, on-demand, never forced.

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
| Curious learner | Wants to level up prompting skills. Course Mode. |

One product. Adjusts tone and depth based on selected persona.

## 3. Problem

- Every AI tool has its own prompt language and quirks
- Beginners freeze at the blank box
- Experts hit walls when they try a new tool
- Pivoters lose weeks relearning basics
- No one has time to "master prompting"
- Courses teach knowledge that is stale within months
- Bias and hallucinations catch users off guard

## 4. Solution — the camper flow

### Walk-in
User opens any AI tool. Camper detects the tool and warms up its expertise for that tool (loads the right prompt patterns, gotchas, and best structures).

### Plan
User types or speaks a rough idea. Camper reads it and quietly plans the full prompt: adds context, tone, format, constraints, examples if useful.

### Suggest
Side bubble shows the enriched prompt with one-line reasons per change. User accepts, edits, or ignores. Camper never sends.

### Predict
Before user hits send, camper shows a one-paragraph preview of the likely output. Saves tokens if preview looks wrong.

### Route & chain
If task needs more than one tool (e.g. write copy + generate image), camper suggests the chain and walks user through it.

### Verify
After real output arrives, camper runs an accuracy pass. Flags likely hallucinations, unsupported claims, out-of-date facts.

### Compare and fix
Diff actual vs predicted. Big gap → suggest refinement line.

### Learn user
Camper remembers user style, past wins, favorite tone. Bubble gets sharper over time.

### Course Mode (optional)
Same camper, teacher hat. Explains its own edits, drops mini-lessons, quizzes user, tracks skill growth. Flip on / off anytime.

## 5. The four autopilot pillars

Traditional AI bootcamps teach these. AICamper autopilots them.

| Pillar | Camper action |
|--------|---------------|
| **Prompt Engineering** | Rewrites rough input into clean, structured prompt tailored to the current tool |
| **Tool Selection** | Detects task type. Suggests best AI tool. Opens it if user says yes. |
| **Workflow Automation** | Chains multi-tool tasks (e.g. Claude draft → Midjourney image → Runway video) with hand-off prompts pre-built |
| **Bias & Accuracy Checks** | Post-response verify: flags hallucinations, unsupported claims, stale facts. Links to sources when possible. |

## 6. Core features (v1)

| # | Feature | What it does |
|---|---------|--------------|
| 1 | Push-to-talk + hover trigger | Speak or type your rough idea |
| 2 | Tool detector | Knows which AI app is open. Loads that tool's expertise. |
| 3 | Prompt planner | Full prompt scaffold from rough input |
| 4 | Prompt enricher | Rewrites with context, tone, format, constraints |
| 5 | Outcome predictor | One-paragraph "expect this" preview |
| 6 | Tool recommender | Picks right AI tool for the task |
| 7 | Workflow chainer | Multi-tool task orchestration |
| 8 | Accuracy verifier | Flags hallucinations + unsupported claims |
| 9 | Gap fixer | Refinement suggestion after real output |
| 10 | Persona presets | beginner, expert, pivoter, founder, student, VA |
| 11 | Memory | Prompts, wins, style. Local only. |
| 12 | Cheap-first router | Small model for most calls. Big only when needed. |
| 13 | Course Mode | On-demand teaching layer over the same camper |

## 7. Intelligence layer

Three stacked models, cheapest first. Router picks the smallest that can do the job.

```
[user rough idea]
   ↓
[Haiku 4.5]     → plan + enrich + predict + tool pick   (fast, cheap, most calls)
   ↓
[user sends to their AI tool]
   ↓
[Haiku 4.5]     → accuracy check on response            (fast)
   ↓
[Sonnet 4.6]    → compare actual vs predicted, fix line (only if gap)
   ↓
[Opus 4.7]      → hard high-stakes prompts + chains     (paid tier)
```

Most user calls stay on Haiku. Cost per user stays low.

## 8. Tech implementation

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
| Predictor | Same call adds `expected_output_summary`, `confidence` |
| Tool recommender | Task-type classifier → tool ranking |
| Workflow chainer | Detects multi-step intent, builds hand-off prompt sequence |
| Accuracy verifier | Post-response Haiku call: `claims[]`, `flags[]`, `sources_needed[]` |
| Diff engine | Embed actual + predicted, cosine-diff, trigger fix if gap > threshold |
| Persona tuner | Adds persona line into planner system prompt |
| Course engine | Wraps every camper action with "why" + optional mini-lesson trigger |
| Memory store | Local SQLite: past prompts, tags, wins. No cloud. |

### Data flow (one cycle)

```
user opens AI tool
   → tool detector reads bundle ID
   → registry loads that tool's expertise
   → user types rough idea
   → accessibility API grabs text
   → Haiku: plan + enrich + predict + suggest tool
   → side bubble shows enriched + preview + tool choice
   → user accepts / edits
   → user sends to AI tool
   → accessibility API grabs response
   → Haiku: accuracy check → flag hallucinations
   → diff engine compares actual vs predicted
   → gap? Haiku suggests fix line
   → memory stores prompt + outcome
   → Course Mode (if on) → explain what camper did + why
```

No model training. Pure prompt engineering + routing + tailored per-tool expertise.

## 9. Tool registry (v1 targets)

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

## 10. Course Mode

Optional teaching layer. Free to enable. Same brain, teacher hat.

- **Explain-as-you-go** — every enrichment shows the "why" as a mini-lesson
- **Skill tracks** — "Prompting 101", "Tool Selection", "Spotting Hallucinations", "Workflow Chaining"
- **Practice mode** — user writes prompt cold, camper grades it, suggests upgrade
- **Skill map** — visual progress across the four pillars
- **Certificate** — after track completion (paid tier)

Toggle: off by default. Non-intrusive. Never blocks the autopilot flow.

## 11. Pricing

Cheapest-possible is a core belief.

| Tier | Price | Model access | Limits | Course Mode |
|------|-------|--------------|--------|-------------|
| Free | $0 | Haiku only | 100 plans / day | Basic tracks |
| Student | $3/mo | Haiku + Sonnet | Unlimited plans, 500 predicts / day | Full tracks |
| Pro | $12/mo | All models + Opus | Unlimited | Full + certificates |
| Team | $8/seat | Shared memory, brand voice, 5+ seats | Team tracks |

Free tier must feel great. That is the funnel.

## 12. Success metrics

- Prompt acceptance rate: **>50%**
- Outcome prediction accuracy: **>70%**
- Free → paid conversion in 30 days: **8%**
- Daily prompts per active user: **>4**
- Time from install to first accepted prompt: **<3 min**
- Course Mode opt-in rate: **>20%** (bonus retention metric)
- Hallucination flag precision: **>80%**

## 13. Out of scope (v1)

- Auto-sending prompts
- Windows / Linux (Mac only)
- Building AI apps for users
- Fine-tuned custom models
- Live human tutoring

## 14. Key risks + fixes

| Risk | Fix |
|------|-----|
| Prediction wrong often | Show confidence score, hide if low |
| Accessibility API blocked in some apps | Fallback: user pastes prompt into camper panel |
| Cost blows up | Hard cap per user, cache repeat enrichments, prefer Haiku |
| Feels intrusive | Trigger only on hover / hotkey, never auto-pop |
| Privacy fear | Memory local. Cloud calls proxied. No logs kept. |
| Tool registry rots as AI tools change | Registry updates ship monthly. Community contributions welcome. |
| Accuracy checker misses hallucinations | Flag confidence + link to source. Never claim "verified". |
| Course Mode feels like homework | Fully optional. Off by default. Never blocks flow. |

## 15. Rollout

- Week 1-3: Fork Clicky base, strip pointing, add tool detector + prompt reader
- Week 4-5: Prompt planner + enricher + predictor + side bubble UI
- Week 6: Persona presets + tool registry (top 5 AI apps) + tool recommender
- Week 7: Workflow chainer + accuracy verifier
- Week 8: Diff engine + fix suggestions + memory store
- Week 9: Course Mode v1 (explain-as-you-go + one track)
- Week 10: Beta with 50 users (mix of beginners, experts, pivoters)
- Week 11-12: Iterate on acceptance rate + course opt-in
- Week 13: Public launch, free tier open

Done.
