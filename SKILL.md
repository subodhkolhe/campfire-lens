---
name: campfire-lens
description: "Your portfolio has a point of view on the world. Campfire Lens finds it. Pick any market theme — AI inference, China+1 manufacturing, green hydrogen — and Lens maps the full value chain, finds India's position in it, then checks your Zerodha holdings to reveal what you're implicitly betting on, what's missing, and what's at risk. Multi-theme per session. Conviction cards. Overweight and underweight warnings. Thesis drift alerts. Trigger on: campfire lens, value chain, thematic analysis, portfolio themes, what themes am I exposed to, where does my portfolio sit on X, India angle, conviction cards, thesis drift."
author: Subodh Kolhe
license: Apache 2.0
---

# Campfire Lens

---

## For Users: How to Use This Skill

### What it does
Campfire Lens takes any market theme and runs it through a structured 7-layer investment framework — from the raw signal down to named global companies, India listed proxies, and a direct read of where your Zerodha portfolio sits in the chain.

It tells you three things no other tool does together:
- **What you're implicitly betting on** — the themes your portfolio is already expressing
- **What's missing** — high-conviction layers you have zero exposure to
- **What's at risk** — holdings that sit in the Disrupted layer of a theme you're long on

### How to start
Type `/campfire-lens` or just describe a theme:
- *"Run Campfire Lens on AI inference demand"*
- *"What's my portfolio's position on China+1 manufacturing?"*
- *"Lens — EV batteries, green hydrogen, and data centres"*

Or browse the suggested starter themes that appear on launch.

### Portfolio context
Lens works without your portfolio — it'll still give you the full value chain and India proxies. But the best output comes when it can see your holdings.

Portfolio features require **Kite MCP** to be connected in Claude's Customize settings. This is a one-time setup:

1. Open [claude.ai](https://claude.ai) → click your profile → **Customize**
2. Go to **Connections** or **MCP Servers**
3. Add a new MCP server with this URL: `https://mcp.kite.trade/mcp`
4. Authenticate with your Zerodha credentials
5. Come back and run Campfire Lens — portfolio features will activate automatically

Once connected, Lens pulls a fresh snapshot of your holdings each session and never stores them.

### What you get
For each theme, Lens produces a **tabbed HTML dashboard**:
- **Heatmap Tab** — your portfolio plotted across all themes run in the session. Overweight warnings, underweight warnings, sectoral cross-mapping, and thesis drift alerts (if running a repeat theme)
- **Per-Theme Tabs** — one tab per theme, containing the full 7-layer Lens output with conviction cards at The Gap section

### What it doesn't do
Lens does not execute trades, place orders, or modify your account in any way. All company references are for analytical and educational purposes only — not recommendations to buy, sell, or hold any security.

---

## For Skill Authors: Framework Reference

### Overview

Campfire Lens is a thematic investment research skill that maps any market signal through a 7-layer framework and connects the output directly to the user's live portfolio. It runs multiple themes per session and produces a single tabbed HTML dashboard — one Heatmap tab plus one tab per theme.

**Core principle:** Most research tools tell you about the world. Campfire Lens tells you about *your* world — where you're positioned, where you're exposed, and where conviction demands you act.

---

## Step 0: Launch Sequence

### 0.1 Greet and offer starters

On `/campfire-lens` invocation, respond with:

> **Campfire Lens** — what theme do you want to map?
>
> Type any theme, or pick a starter:
> - AI inference demand
> - China+1 manufacturing
> - India defence & aerospace
> - Green hydrogen
> - Data centre infrastructure
> - EV batteries & supply chain
> - Indian pharma CDMO boom
> - Semiconductor packaging
> - Domestic consumption recovery
> - Cybersecurity infrastructure

Accept free-form input at any time. Multiple themes can be stated upfront or added mid-session.

### 0.2 Ask for portfolio context

After the theme is confirmed:

> Want me to include your Zerodha portfolio? It unlocks Your Bet, overweight/underweight warnings, and thesis drift alerts.
> [Yes, connect Zerodha] [Skip — just the value chain]

If yes → proceed to Step 0.3
If skip → run the full 7-layer framework without portfolio-specific layers (Your Bet and The Gap will use generic investor framing instead)

### 0.3 Pull portfolio data (if authorized)

**First, detect whether Kite MCP is connected:**

Attempt `kite:get_holdings`. If the tool is unavailable or returns an auth error:

> **Kite MCP not connected.** Portfolio features need a one-time setup:
>
> 1. Open [claude.ai](https://claude.ai) → click your profile → **Customize**
> 2. Go to **Connections** or **MCP Servers**
> 3. Add a new MCP server — URL: `https://mcp.kite.trade/mcp`
> 4. Authenticate with your Zerodha credentials
> 5. Return here and re-run Campfire Lens — portfolio features will activate automatically
>
> Running in **research-only mode** for now. You'll still get the full value chain, India proxies, and conviction cards.

Then proceed with the full 7-layer framework in research-only mode. Do not block or halt — the skill should always deliver value regardless of connection status.

**If Kite MCP is connected and working:**

```
kite:get_holdings        → All direct equity positions
kite:get_mf_holdings     → All mutual fund positions (optional, flag if unavailable)
kite:get_margins         → Cash balance
```

Compute for every holding:
- Current value = quantity × last_price
- Return % = (last_price − average_price) / average_price × 100
- Sector classification (use same sector taxonomy as Campfire Analyst)

Store holdings in memory for use across all themes in the session. Do not re-pull between themes.

### 0.4 Check for thesis drift

Before running any theme, check: has this user run this same theme in a prior session?

**Thesis Drift Detection logic:**
- Prior session data is not available to Claude between sessions by design
- Thesis Drift Alert activates when the user explicitly references a prior run: *"I ran EV batteries last month"* or *"check if my position has changed on this"*
- When triggered: compare current holdings mapped to this theme against what the user describes as their prior position. Flag any holdings that have moved from Winner → Disrupted or vice versa, or key names that have been added/removed
- Present as a **Thesis Drift Alert card** at the top of the theme tab

---

## Step 1: Run the 7-Layer Framework

Run all 7 layers for each theme. Layers 1–4 (Spark through Disrupted) run silently as structured analysis. Layers 5–7 (India Angle, Your Bet, The Gap) are the user-facing output.

**All 7 layers feed the per-theme tab in the dashboard. Do not output any layer inline during computation.**

---

### Layer 0 — Spark

**What it is:** The signal in one sentence. Why now — not just what.

**Output format:**
> [Theme name] — [one sentence stating what is happening] — [one phrase stating why this moment is structurally different from 2 years ago]

**Example:**
> AI Inference Demand — Every major AI model is now in production serving billions of daily queries — inference has crossed from R&D cost to core infrastructure spend.

**Quality check:** If the Spark reads like a Wikipedia definition, rewrite it. It must capture the *timing urgency*, not just the phenomenon.

---

### Layer 1 — Winners

**What it is:** Companies and sectors that directly own the problem. They benefit structurally, not cyclically.

**Scope:** Global companies only at this layer. India proxies come at Layer 4 (India Angle).

**For each winner:**
- Company / sector name
- Why they win (one line — mechanism, not description)
- Time horizon: Near-term (0–12 months) / Structural (1–3 years) / Infrastructure (3+ years)

**Aim for 4–6 winners maximum.** Ruthlessly prioritise. If a company is a marginal winner, it belongs in Enablers.

**V2 addition — Conviction Scoring:**
Score each Winner: **High / Medium / Low** structural tailwind based on:
- Is the demand driver structural or cyclical?
- Is the company's moat durable or replicable?
- Is India's policy environment supportive?

---

### Layer 2 — Enablers

**What it is:** Picks and shovels. Companies that enable the Winners to win — they don't own the problem but they're essential infrastructure for the solution.

**Same format as Winners.** Time horizon + one-line mechanism.

**Key distinction from Winners:** Enablers have a different risk profile — they benefit from volume, not outcome. They win whether the dominant player wins or not.

---

### Layer 3 — Disrupted

**What it is:** Who loses. Companies or sectors whose current business model is structurally undermined by this theme. This is the bear case layer — essential for honest portfolio analysis.

**For each disrupted player:**
- Company / sector name
- Why they lose (mechanism — not just "competition")
- Disruption timeline: Immediate / Gradual / Eventual
- **Avoid signal:** Should a portfolio that is long this theme also avoid or reduce this?

**This layer is non-negotiable.** Every theme has disruption. If you can't find it, dig harder.

---

### Layer 4 — India Angle

**What it is:** India's specific position across all three layers — Winners, Enablers, and Disrupted. Listed companies only (BSE / NSE main board + BSE SME / NSE Emerge).

**Use web search aggressively here.** This is the layer where Claude's training data is most likely to be stale or incomplete. Always search before populating.

```
Search queries to run per theme:
- "[theme] India listed companies BSE NSE"
- "[theme] India beneficiaries stocks 2025"
- "[specific sub-segment] India proxy listed"
- "[theme] BSE SME NSE Emerge companies"   ← SME board search
- "[theme] India PLI scheme beneficiaries listed"
```

**For each India proxy:**
- Company name
- Exchange: NSE / BSE / BSE SME / NSE Emerge
- Which layer they sit in: Winner / Enabler / Disrupted
- One-line rationale

**Policy Tailwind / Headwind flag** — mandatory for every India Angle:
- 🟢 **Policy Tailwind** — active PLI scheme, DPIIT push, recent budget allocation, or regulatory support
- 🔴 **Policy Headwind** — regulatory uncertainty, import duty risk, or policy reversal risk
- ⚪ **Policy Neutral** — no significant policy dimension

Search for current policy status:
```
"[theme] PLI scheme India 2025"
"[theme] Union Budget India allocation"
"[theme] SEBI RBI regulation India"
```

**V2 addition — Smart Money Alignment:**
For each India proxy, fetch FII/DII flow data:
```
Search: "[company name] FII DII holding change latest quarter"
```
Flag each proxy:
- 📈 **FII Buying** — institutional accumulation signal
- 📉 **FII Selling** — distribution signal despite fundamental tailwind
- ➡️ **Neutral** — no significant directional flow

---

### Layer 5 — Your Bet

**What it is:** What your portfolio is implicitly saying about this theme. This layer only runs if portfolio data was pulled.

**Compute:**
1. Match every holding against Winners, Enablers, and Disrupted lists for this theme
2. Calculate theme exposure = sum of matched holdings as % of total portfolio value
3. Identify which layer each matched holding sits in

**Present as:**
> Your portfolio has [X]% exposure to [theme name].
> - [Holding A] — [X%] — Winner layer
> - [Holding B] — [X%] — Enabler layer
> - [Holding C] — [X%] — ⚠️ Disrupted layer

**Sectoral cross-mapping:** Note if this theme's exposure cuts across multiple sectors in the portfolio. Example: *"Your AI inference exposure sits across IT (TCS) and Energy (NTPC for data centre power) — two different sector bets expressing the same theme."*

**If no portfolio holdings match any layer:**
> Your portfolio has no current exposure to [theme name].

---

### Layer 6 — The Gap

**What it is:** One or two things conviction demands you look at. Not a list. Not a screener. A pointed observation.

**Conviction Card format** — one card per gap identified (maximum 2 cards per theme):

```
┌─────────────────────────────────────────┐
│  [Company Name]                          │
│  [Exchange: NSE / BSE / BSE SME]        │
│  Layer: Winner / Enabler / Disrupted    │
│                                          │
│  Why it fits:                           │
│  [2–3 sentences — mechanism, not hype]  │
│                                          │
│  [V2] Earnings watch:                   │
│  [Next results date if within 30 days]  │
│  [Expected catalyst — what to listen    │
│   for in management commentary]         │
│                                          │
│  [V2] Smart Money:                      │
│  [FII/DII flow direction this quarter]  │
└─────────────────────────────────────────┘
```

**The Gap logic:**
- If portfolio has no exposure: flag the highest-conviction Winner or Enabler from the India Angle
- If portfolio has exposure but in Disrupted layer only: flag the contrast — you're long the losers, here's a winner
- If portfolio has good Winner exposure: flag a non-obvious Enabler the market hasn't fully priced
- Never manufacture a gap. If the portfolio is well-positioned, say so.

---

## Step 2: Heatmap Computation

After all themes are run, compute the Heatmap data silently:

### 2.1 Portfolio theme exposure matrix

Build a matrix: Holdings × Themes
- For each holding, which themes does it appear in, and at which layer?
- Compute per-theme exposure %

### 2.2 Overweight Warning

**Trigger:** Any single theme accounts for >25% of total portfolio value across matched holdings.

> ⚠️ **Overweight — [Theme Name]**
> [X]% of your portfolio is exposed to this theme across [N] holdings ([list them]). This may be intentional concentration — or an unintended bet you haven't explicitly made.

### 2.3 Underweight Warning

**Trigger:** A theme was run through Lens but portfolio exposure is <2% or zero.

> 📍 **Underweight — [Theme Name]**
> You ran this theme but your portfolio has near-zero exposure. If the thesis is sound, this is the gap that matters most.

### 2.4 Sectoral cross-mapping

For each theme, identify how many different sectors in the portfolio it touches. Surface any theme that cuts across 3+ sectors — this reveals hidden concentration the sector view alone misses.

> 💡 **Cross-sector theme:** Your [theme] exposure spans [Sector A], [Sector B], and [Sector C]. Your sector allocation understates this thematic concentration.

### 2.5 Thesis Drift Alert

When a user flags a prior session on the same theme:

> 🔄 **Thesis Drift — [Theme Name]**
> Based on what you've described from your prior position:
> - [Holding X] has moved from Winner → Disrupted layer since you last ran this theme
> - [Holding Y] was a key gap then — you've since added it / still haven't added it
> Re-examine your conviction on [theme] given these shifts.

---

## Step 3: Generate the Dashboard

Single self-contained HTML file. Same design language as Campfire Analyst — Notion-like minimalist.

```css
/* Design tokens — consistent with Campfire Analyst */
--bg: #ffffff;
--surface: #f7f7f5;
--border: #e8e8e4;
--text: #37352f;
--text-secondary: #787774;
--warning-orange: #f59e0b;
--signal-green: #10b981;
--signal-red: #ef4444;
--signal-neutral: #6b7280;
```

**Font:** Inter. **Max-width:** 720px, centered. **No dark theme, no shadows, no gradients.** Self-contained HTML with inline CSS and inline JS. Vanilla JS only — no React.

---

### Tab 1 — Portfolio Heatmap

**Header:**
> Campfire Lens — [N] themes analysed · Portfolio snapshot: [date]

**Theme exposure table:**

| Theme | Exposure % | Primary Layer | Warning |
|-------|-----------|---------------|---------|
| AI Inference | 18% | Winner | — |
| EV Batteries | 31% | Enabler | ⚠️ Overweight |
| Green Hydrogen | 0.4% | — | 📍 Underweight |

**Sectoral cross-mapping section** — for any theme touching 3+ sectors, show a visual tag cluster.

**Overweight and Underweight warning cards** — styled as alert banners, expandable for detail.

**Thesis Drift Alert cards** — if applicable, shown at top of Heatmap tab with orange border.

---

### Tab 2–N — Per Theme Tabs

One tab per theme. Each tab contains:

1. **Spark** — styled as a large pull quote
2. **Winners** — card grid, each card showing company/sector name, time horizon badge, one-line mechanism. V2: conviction score badge (High / Medium / Low)
3. **Enablers** — same card format
4. **Disrupted** — same card format, red-tinted border. Avoid signal shown as a tag.
5. **India Angle** — table layout:
   - Company | Exchange | Layer | Policy flag | V2: FII/DII flow arrow
6. **Policy Tailwind / Headwind summary** — one-line read on India's policy environment for this theme
7. **Your Bet** — portfolio holdings mapped to layers. If no portfolio: show a neutral framing. Sectoral cross-mapping note shown here.
8. **The Gap** — conviction cards, styled distinctly from the rest of the tab. Card border, company name large, rationale in secondary text, V2 earnings watch and smart money as footer rows inside the card.

---

## Execution Sequence

```
1. Launch → show starters + theme input
2. Confirm theme(s) → ask for portfolio context
3. Pull portfolio data (if authorized) → kite MCP
4. For each theme:
   a. Run layers 0–6 silently
   b. Compute Your Bet against holdings
   c. Identify The Gap
5. Compute Heatmap data (exposure matrix, warnings, drift)
6. Generate single HTML dashboard → save → present
```

Steps 4–6 run as a single response after all themes are confirmed. Do not stream partial results per theme — compute everything then generate the dashboard in one pass.

If the user adds a new theme mid-session: recompute layers for new theme, update Heatmap, regenerate dashboard.

---

## Web Search Protocol

Campfire Lens is a live research tool. Web search is mandatory at Layer 4 (India Angle) and for V2 features. Never rely on training data alone for:
- India listed proxy discovery
- Policy status (PLI, budget, SEBI)
- FII/DII flow data (V2)
- Earnings dates (V2)
- SME board listings (V2)

**Always search before populating India Angle.** A stale India proxy is worse than none — it misleads the portfolio analysis that depends on it.

---

## Key Principles

### Investment-first, not education-first
Every layer exists to help a decision, not explain a concept. Spark is timed. Winners are actionable. Disrupted is an avoid signal. The Gap is pointed. There is no "background" section.

### The Disrupted layer is non-negotiable
Every theme creates losers. Showing only winners is incomplete analysis. The Disrupted layer is what makes Lens honest — and what makes the Overweight Warning meaningful.

### India Angle is the moat
Global value chain decomposition is widely available. India's specific position — by listed company, by exchange, with policy context — is what no generic tool provides. This layer gets the most search effort.

### The Gap must be singular
Two conviction cards maximum. Never a list. If everything looks like a gap, nothing is. Force the ranking.

### Zero prior bias
Every session starts fresh. No default themes, no assumed sector preferences, no carried-over portfolio states. The data speaks.

### V2 features are additive, not required
Smart Money Alignment, Earnings Cycle, SME proxies, and Conviction Scoring enhance the output when available but do not block V1 functionality if data is unavailable. Gracefully degrade: if FII data search returns nothing useful, omit the flag silently.

---

## Version Notes

**V1 — Core (current)**
Full 7-layer framework · Multi-theme · Heatmap tab · Conviction cards · Listed proxies (NSE/BSE main board) · Overweight/Underweight warnings · Sectoral cross-mapping · Policy Tailwind/Headwind flag · Thesis Drift Alert

**V2 — Depth (active)**
Smart Money Alignment (FII/DII flows) · Earnings cycle awareness on conviction cards (next 30 days + catalyst) · SME listed proxies (BSE SME / NSE Emerge) · Conviction scoring per theme (High / Medium / Low)

**V3 — Planned**
Pre-IPO and VC-backed India proxies · Domain Knowledge registry (seed companies not discoverable via search) · Conviction scoring with quantitative backing
