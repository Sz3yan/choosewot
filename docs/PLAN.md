# ChooseWot — Product & Technical Plan

Status: planning draft (v0.1, 2026-10-26)
Repo: greenfield, no code yet.

---

## 1. The idea in one line

Post a decision you're stuck on, get a crowd verdict with reasons attached — not just a yes/no tally.

Example: "Should I buy the Pixel 10 at $1,099 when my S23 still works?" → people vote **Buy / Don't buy** and must say *why*. You get a verdict screen: 68% Don't buy, top reasons ranked by other voters' upvotes.

## 2. Assumptions (stated, not verified)

- Audience v1: Singapore students + young working adults, 18–30. Decisions are consumer/life-sized: gadgets, outfits, courses, job offers, apartments, whether to text someone back.
- First platform: **mobile web (PWA), not an app-store app.** Reasoning below (§6).
- Solo/small-team build, part-time. Budget ~S$0–30/month until there are real users.
- Not a validated business. This plan is an engineering plan for a testable product, not a claim that demand exists.

## 3. Why most "ask the crowd" apps die — design the fixes in from day one

| Failure mode | What it looks like | Fix built into v1 |
|---|---|---|
| **Cold start** | You post, nobody votes, you leave | Answer-first feed: you must vote on 2 open decisions before your post goes live ("pay with a vote"). Seeds supply from demand. |
| **Garbage votes** | "buy lah", "no need" | Reason is mandatory, min 20 chars, and the vote doesn't count toward the tally without it. |
| **Bandwagon / anchoring** | First 3 votes decide the rest | **Blind voting**: tally and reasons are hidden until you've voted. Classic Delphi-style debiasing. |
| **No reason to come back** | One-shot usage | Outcome loop: 7 days later the poster reports what they actually did. Voters who backed the chosen option get credit. That's the retention hook *and* the reputation signal. |
| **Popularity ≠ good advice** | Loudest wins | Weighted verdict: show raw % AND a weighted % using voter credibility (§5.3). Show both, never hide the raw number. |
| **Harassment / doxxing** | "Should I dump my bf, here's his IG" | Rules + report flow + no named private individuals (§8). |

The honest risk: #1 and #4 are the ones that actually kill it. Everything else is polish.

## 4. MVP scope — what ships first

**In scope (v1):**
1. Auth (email magic link + Google).
2. Create decision: title, 1–4 options (default template `Buy` / `Don't buy`), optional body ≤500 chars, 0–3 images, category, deadline (6h / 24h / 3d / 7d).
3. Vote: pick one option + write reason (≥20 chars). One vote per user per decision, editable for 5 minutes.
4. Blind reveal: after you vote (or after the deadline), see tally + reasons.
5. Rank reasons: 👍 helpful on other people's reasons. Reasons sort by helpful count.
6. Feed: Open decisions, sorted by `needs votes` (fewest votes, nearest deadline) — not by popularity. Plus a Trending tab once volume exists.
7. Verdict page, publicly shareable by link (this is the growth loop — people paste it in group chats).
8. Outcome follow-up: push/email at deadline + 7 days → "What did you decide?"
9. Report + block. Basic admin view to hide content.

**Explicitly out of scope for v1** (write them down so they don't creep in):
- Comments/threads. Reasons *are* the discussion layer in v1.
- DMs, follows, notifications feed.
- Native apps, payments, "expert panels", AI summarisation of reasons, groups/private circles.

Private circles ("ask only my friends") is the strongest v2 candidate — it fixes cold start differently and is what people will actually ask for.

## 5. Data model

Postgres. Tables with the fields that matter:

```sql
users            (id, handle unique, display_name, avatar_url, created_at, is_banned)
decisions        (id, author_id, title, body, category, deadline_at, status,
                  visibility, created_at, hidden_at, vote_count, reveal_policy)
decision_options (id, decision_id, label, position)            -- 2..4 rows per decision
decision_images  (id, decision_id, storage_path, position)
votes            (id, decision_id, option_id, voter_id, reason, created_at, edited_at,
                  helpful_count, UNIQUE(decision_id, voter_id))
reason_votes     (vote_id, user_id, PRIMARY KEY(vote_id, user_id))  -- "helpful"
outcomes         (decision_id PK, chosen_option_id, note, satisfaction smallint, created_at)
reports          (id, target_type, target_id, reporter_id, reason, created_at, resolved_at)
user_stats       (user_id PK, votes_cast, helpful_received, outcome_agreements,
                  outcome_opportunities, credibility numeric)
```

**5.1 Invariants (enforce in DB, not just app code)**
- `UNIQUE(decision_id, voter_id)` — one vote each.
- `votes.option_id` must belong to `votes.decision_id` → composite FK on `(decision_id, option_id)`.
- Can't vote on your own decision, can't vote after `deadline_at` → trigger or `CHECK` via function.
- `char_length(reason) >= 20`.
- Row-level security: a non-voter cannot `SELECT` other people's votes on an open decision. **The blind-vote rule has to live in the database**, or anyone with devtools reads the tally early. This is the one place where sloppiness breaks the core mechanic.

**5.2 Verdict computation**
- Raw: `count(votes) group by option_id`.
- Weighted: `sum(0.5 + credibility)` per option, so a new user still counts for something.
- Store a frozen snapshot at `deadline_at` so verdict pages are immutable and cacheable.

**5.3 Credibility (keep it dumb and legible)**
```
credibility = 0.6 * helpful_rate + 0.4 * outcome_agreement_rate   -- both in [0,1]
helpful_rate        = helpful_received / max(votes_cast, 5)       -- 5 = anti-small-sample
outcome_agreement   = agreements / max(opportunities, 5)
```
Recompute nightly, not per-request. Show it as a 1–5 badge, never a raw number — raw scores invite gaming. Known weakness: outcome-agreement rewards *agreeing with the poster*, not being right. Flag it as a v2 research problem; don't pretend the metric means more than it does.

**5.4 Anti-abuse, v1-level only**
Rate limits (5 decisions/day, 50 votes/day), one account per email, duplicate-reason detection (exact + near-dup hashing) to catch copy-paste farming, shadow-hide instead of delete so brigades don't notice. Not aiming for Sybil resistance at this stage — say so rather than implying the tally is trustworthy at scale.

## 6. Stack — recommendation and the trade-off

**Default: Next.js (App Router) + Supabase, deployed on Vercel, installable PWA.**

- Next.js for SSR'd public verdict pages → link previews in WhatsApp/Telegram, which *is* the distribution mechanism.
- Supabase: Postgres + auth + storage + RLS + realtime in one box. RLS is what enforces blind voting (§5.1).
- Tailwind + shadcn/ui for speed. Zod for validation shared between client and server actions.
- Web push via the Push API for the outcome follow-up; email (Resend) as fallback on iOS.

**Trade-off:** a PWA gets worse push notifications on iOS and no app-store discovery. In exchange you ship weeks earlier, update instantly, and shared links open the real thing instead of an install prompt. For a product whose entire growth loop is *pasting a link into a group chat*, that's the right call. If retention proves out, wrap it with Expo/React Native later — the Supabase layer is reused unchanged.

Alternative worth considering only if you're sure: Expo + Supabase from day one. Better notifications and feel, but app review latency and a much weaker share loop while you still have zero users.

## 7. Build order

| Phase | Work | Rough effort | Done when |
|---|---|---|---|
| 0. Skeleton | Repo, Next.js, Supabase project, migrations, auth, CI (lint + typecheck + tests) | 2–3 days | You can log in and see an empty feed |
| 1. Core loop | Create → vote-with-reason → blind reveal → verdict page | 1 week | You can run the full loop end to end on your own phone |
| 2. Quality | RLS lockdown, rate limits, report/hide, helpful-ranking, reason validation | 4–5 days | A hostile tester with devtools can't read a tally early or vote twice |
| 3. Loop closers | Deadline job, outcome prompt, credibility job, share-card OG images | 4–5 days | A decision completes its whole lifecycle unattended |
| 4. Pilot | 30–50 real users (one NTU hall / Telegram group), instrument everything | 2 weeks running | You have the §9 numbers |
| 5. Decide | Private circles, or mobile wrap, or kill it | — | Based on §9, not on vibes |

## 8. Legal & safety (Singapore)

Not legal advice — these are the items to actually check before a public launch.

- **PDPA**: you're collecting personal data. Need a privacy policy, a named purpose, consent at signup, deletion on request, and reasonable security. Minimise: don't collect phone numbers or precise location in v1.
- **Content about real people**: posting "should I dump X, here's their photo" is a defamation and PDPA problem *and* a harassment problem (POHA). Rule: no identifiable private individuals in posts. Enforce by policy + report flow, and strip image EXIF on upload.
- **Online Criminal Harms Act / Broadcasting Act obligations** scale with service size — a pilot-scale service has light obligations, but know they exist before you grow.
- **Don't be an advice service**: financial, medical, and legal questions need a visible disclaimer and ideally a blocked category. "Should I put my savings into X" is a liability you don't want.
- **Age**: 13+ minimum, 18+ is simpler. State it and don't collect data you can't justify.
- **Minimum security hygiene**: RLS on every table (default deny), service-role key server-side only, signed image URLs, CSP, no secrets in the client bundle, Dependabot on. Write a one-page threat model (abuse of the tally, scraping, account takeover, IDOR on decision/vote IDs) before the pilot — this is the part most side projects skip and it's cheap now.

## 9. Pilot success criteria — set the bar before you look at the data

Measure, per pilot cohort:
- **Supply/demand ratio**: median votes per decision within 24h. **Target ≥ 8.** Below ~4 the product feels dead and nothing else matters.
- **Reason quality**: % of reasons rated helpful by ≥1 other voter. Target ≥ 40%.
- **Outcome reporting rate**: Target ≥ 30%. This is the retention loop; if nobody reports back, credibility and re-engagement both collapse.
- **W2 retention** (voted in week 2 having voted in week 1): Target ≥ 25%.
- **Poster-reported usefulness**: 1–5 at outcome time, target median ≥ 4.

If votes/decision < 4 and outcome reporting < 15% after two weeks of active nudging, the open-feed model is wrong. Pivot to private circles (ask your own group chat) rather than adding features to the public feed.

## 10. Immediate next actions

1. `npx create-next-app@latest` + Supabase project + first migration (`users`, `decisions`, `decision_options`, `votes`). Commit, push, open a draft PR.
2. Write the RLS policies *with the schema*, including a test that proves a non-voter cannot read votes on an open decision. That test is the product.
3. Hand-run the loop offline first: post 5 real decisions in a group chat, require reasons, see whether people engage. One afternoon, zero code, and it tells you more about §9 than phase 1 does.
4. Decide platform (§6) explicitly and record it in `docs/decisions/0001-platform.md`.

## 11. Open questions

- Binary-only, or 2–4 options? (Binary is cleaner and shares better; multi-option covers "which of these 3 phones".) Leaning: binary default, multi allowed, max 4.
- Anonymous posting? Helps for embarrassing decisions, hurts accountability. Leaning: pseudonymous handles by default, no real names anywhere.
- Deadline-forced reveal vs vote-to-reveal — currently both. Simplify if it confuses testers.
