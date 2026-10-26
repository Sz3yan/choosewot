# ChooseWot — Target Audience

Status: draft (v0.1, 2026-10-26). Companion to `docs/PLAN.md`.
Everything here is reasoning from assumptions, not measured data. Items marked **[verify]** need a primary source before anyone acts on them.

---

## 0. The segmentation that actually matters

Don't segment by age or income. Segment by **who the person cannot ask**.

The free competitor to this app is the group chat the user is already in. It has zero friction, real relationships, and replies in 30 seconds. A standalone product only beats it in three situations:

| Situation | Why the group chat fails | Example |
|---|---|---|
| **Too embarrassing** | They'd be judged, or it reveals income/insecurity | "Is $1,099 stupid for a phone when I earn $900 a month" |
| **Friends aren't qualified** | Nobody in their network has the context | "Bank internship at $1,200 vs security startup at $900" |
| **They want volume, not three opinions** | 3 friends who all think alike | "Does this fit work for a dinner on Friday" |

Those three are the real target definition. Everything below is just *where those people physically are*.

## 1. Beachhead: one dense campus community

**Primary: NTU undergraduates, 19–25, living in hall or in a tight course cohort.**

Chosen for mechanical reasons, not because it's the biggest market:

- **Density solves cold start.** One hall is a few hundred people who already share Telegram groups. The feed can feel alive inside one building.
- **Decision frequency is high and the topics are narrow** — gadgets, electives, internships, outfits, trips, dating. Narrow topics mean voters feel competent, which is what makes them vote.
- **Stakes are low**, so the legal and moderation surface is small while the product is fragile.
- **Behaviour already exists** in IG story polls and "eh help me choose" messages in hall chats.
- **You're inside it.** Distribution and product intuition for free. This is the single biggest advantage available and it expires when you graduate — use it now.

**Pilot unit: one hall or one cohort of 300–600 people. Target 50–80 weekly actives.** That number, not TAM, is the only one that matters for the next six months.

## 2. The one piece of arithmetic that decides viability

Votes per decision is the product. It's determined entirely by the per-user ratio of votes cast to decisions posted:

```
votes per decision = (votes per user per week) / (decisions per user per week)
```

- Plausible: 0.5 decisions/week, 6 votes/week → **12 votes per decision.** Works.
- Pessimistic: 0.5 decisions/week, 2 votes/week → **4 votes per decision.** Feels dead (plan's floor is 8).

So the whole business reduces to: **will an ordinary user vote at least 8× more often than they post?** Nothing else in this document matters as much. Measure it in pilot week 1 and treat it as the go/no-go.

This also tells you who the important user is — see Persona B.

## 3. Personas

### A. The big-purchase deliberator — *the poster*
21, uni or poly, funded by part-time work or an allowance. Deciding on a S$400–1,500 purchase: phone, iPad, bike, camera, a trip. Has already read reviews and watched a YouTube comparison; what they actually want is **permission or a veto from people with no stake**.

- Posts rarely — maybe once or twice a month. High intent when they do.
- Wants: a clear verdict with reasons they hadn't thought of.
- Gives the app its content but **will never be the retained user.** Low frequency is intrinsic to the need, not a product flaw to fix.

### B. The lurker-voter — *the user the product actually lives or dies on*
23, scrolls out of nosiness and mild opinionatedness. Votes because it takes 40 seconds, feels like having a say, and other people's spending decisions are entertaining. May never post anything.

- **This is the retained user**, and most plans for apps like this ignore them completely.
- Here's the hard part: they currently get almost nothing in return. Mild entertainment and a star rating. That asymmetry — the retained side being the unpaid side — is the central product risk, well ahead of moderation or scaling.
- What might pay them: being proven right (outcome reports), a visible track record worth having, and genuinely entertaining content in the feed. All three are weak compared to what TikTok pays them for the same 40 seconds.

### C. The career/life fork — *the highest-value, lowest-frequency user*
Final-year or 1–2 years into work. Two job offers, or a decision about further study, or a move. Can't ask coursemates (competing for the same roles) or parents (don't understand the industry).

- Highest willingness to pay, if there's ever anything to pay for.
- Needs *qualified* voters, not a crowd — which the open feed cannot provide. This persona is the argument for topic-scoped or invite-only panels later, and the reason "expert panels" keeps coming up.
- Also the riskiest content: adjacent to financial advice. Keep a disclaimer and no money-allocation questions.

### Anti-persona: who this is explicitly not for
- **People seeking emotional support.** "Should I break up with him" looks like a decision and is not one. It's a different product, a moderation problem, and a safety problem. Route it away in the category picker.
- **Anyone asking where to put money.** Blocked category, per the plan's legal section.
- **Under-18s.** 18+ minimum keeps PDPA and duty-of-care obligations simpler while you're one person with no legal budget.

## 4. Expansion rings — in order, only after the previous one works

1. **Other SG campuses** (NUS, SMU, the polys). Same behaviour, same density, needs no product change. Cheap.
2. **Young working adults, 25–32.** Bigger-ticket decisions (laptops, travel, car, first flat) and more disposable income — but much weaker community density, so the cold-start problem comes back in full. Needs private circles to work at all.
3. **Private circles for existing friend groups.** Not a demographic, a different distribution model: the app becomes the structured-poll tool a group chat reaches for, rather than a destination. **This is the most likely survivable form of the product** and the plan already flags it as the pivot if open-feed metrics miss.
4. **Regional SEA.** Ignore for now. Nothing about the first 12 months is improved by thinking about it.

## 5. Where these people actually are

Realistic acquisition channels, roughly in order of cost-effectiveness:

- **Hall and course Telegram groups.** The whole pilot. Ask the group admin rather than cold-posting.
- **Short-form video of real verdicts.** Post anonymised verdict screens as TikTok/IG content: "34 people voted on whether she should buy the $1,099 phone — 68% said no, here's the best reason." The verdict page *is* the content, which makes this the strongest growth channel and the main reason the plan builds share-ready verdict pages. Reverse funnel: content first, app second.
- **r/SGExams, r/NTU, r/askSingapore.** Real audiences with the exact behaviour, but self-promotion rules are enforced — read them and ask mods first, or you burn the channel permanently.
- **HardwareZone / Carousell-adjacent communities** for Persona A's tech purchases specifically.
- **Club and CCA committees** — one committee adopting it for internal decisions seeds 20 accounts at once.

### Getting the first 50 users
**Recruit voters, not posters.** A visitor who lands on an empty feed never comes back. Before any public link exists, seed 20–30 real decisions (your own, and friends' with permission), so the first arrival sees something live and can vote immediately. The "pay with a vote" gate is worthless on day one because there's nothing to vote on — plan to turn it on only once there are ~15 open decisions at any time.

## 6. Messaging, per segment

- **Persona A:** "Get 30 honest opinions before you spend $1,000."
- **Persona B:** "Judge other people's decisions. 40 seconds each."  (Blunt, accurate, and the honest reason they'd open it.)
- **Persona C:** "Ask people who don't know you and have no stake in your answer."

Note how different B is from the others. If the marketing only ever speaks to posters, the app has no voters and nothing works.

## 7. What to verify before committing

- **[verify]** NTU enrolment and hall-resident counts, from NTU's published enrolment statistics — I don't have a reliable figure and shouldn't guess one.
- **[verify]** SG full-time tertiary population, from MOE/SingStat education statistics.
- **[verify]** Prior art and why it died. From memory, Wishbone (teen this-or-that voting, big around 2015–17, ended after a major data breach), Thumb, Swayy and Polls apps all spiked and collapsed; Reddit's decision subreddits are the durable winner. **Those recollections are unreliable — confirm each before quoting them.** The pattern worth checking is whether any standalone "crowd verdict" app ever retained users past the novelty spike, and if not, why. That's the most informative research you can do this week, and it costs an afternoon.
- **Unverifiable without building:** whether Persona B votes 8× for every post. Pilot only.

## 8. Honest summary

The addressable behaviour is real and extremely common. The *market* for a standalone app serving it is unproven and the graveyard is large. The audience to aim at is not "people making decisions" — it's **the nosy voter who will judge a stranger's purchase for free**, because they are the supply, the retention, and the reason a poster gets an answer at all. Build and measure for them first; the posters will arrive on their own.
