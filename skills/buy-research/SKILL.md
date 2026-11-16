---
name: buy-research
description: Research what to buy — tiered product research with trusted evidence, local/cross-border/China price checks and a verdict-first pick. Use for "which X should I get", "best X under €Y", "is X worth it".
---

# Buy Research

Turn "I need X" into a trustworthy, well-priced pick without hours of tab-hopping. Works for any product — toothpicks to laptops.

Goals, in order: avoid a bad pick, avoid untrustworthy info, avoid overpaying, save time.

## 0. Profile (fill in once)

Edit this block after installing. If a field is blank, ask the user once (only when it affects the pick) and suggest they save the answer here.

```
Country / city:      <e.g. Netherlands>
Currency:            <e.g. EUR>
Devices:             <e.g. Windows laptop, Android phone (USB-C)>
Second-hand OK?:     <yes / no>
```

Devices are for compatibility checks only (USB-C vs Lightning, iOS-only vs Android apps, Windows vs macOS support), not a brand preference.

## 1. Pick the tier

Classify before doing anything. Weigh: price, how long it's kept, spec complexity, cost of a wrong pick. State the tier on the first line of the answer; the user can override ("go deep", "just quick").

| Tier | Typical | Intake | Research |
|---|---|---|---|
| **Quick** | Cheap, low-stakes, commodity (toothpicks, cables, batteries) | No questions — state assumptions | A few searches; best-value pick + where to buy |
| **Standard** | Mid-price or a few specs matter (headphones, kettle, backpack) | 1–3 questions, only if the answer would change the pick | Shortlist ~3, evidence check, price check |
| **Deep** | Expensive, long-lived or spec-heavy (laptop, phone, appliance) | Interview: use case, budget, dealbreakers, compatibility, deadline | Full evidence + price check + buy-now-or-wait |

Rough price guide only: Quick < ~€30, Deep > ~€300 (convert to the profile currency). Complexity and longevity can move an item up or down a tier.

Ask intake questions in one batch, not drip-fed. If the user already gave the info, don't ask again.

## 2. Defaults (apply unless the user says otherwise)

- **Philosophy: best value.** Recommend the price/quality sweet spot. Mention the upgrade option, but flag diminishing returns explicitly instead of recommending it.
- **Compatibility** with the profile's devices.
- **Second-hand / refurbished** only if the profile allows it.

## 3. Evidence rules

Weight sources in this order:

1. **Lab / measured tests** — RTINGS, Which?, Consumer Reports, Notebookcheck, national consumer associations (e.g. Consumentenbond, Stiftung Warentest), and similar independent testers.
2. **Long-term owner experience** — Reddit, local tech forums (e.g. Tweakers), owner reports after months of use. Mine these for recurring failure modes.
3. **Expert hands-on reviewers** — established outlets and YouTube reviewers who actually tested the product.
4. **Bulk shop reviews** — aggregate ratings and recurring complaints on large retailers (Amazon and the big local shops). Discount suspicious patterns (review bursts, generic 5-star text, incentivised reviews).

**Affiliate "best X of <year>" listicles are low-trust**: use them only to discover candidates, never as support for a pick.

Look for consensus across tiers. When sources conflict, say so and say which you believe and why. For Quick tier, a light version is fine (one or two credible signals + no red flags).

## 4. Price check

Only report prices actually seen in search/fetch results — never estimate or invent a price. Give the date and a link; treat prices as indicative.

- **Local retailers** (always): the country's price aggregators plus its big shops and relevant specialist shops. Examples:
  - NL: Tweakers Pricewatch, Kieskeurig; bol.com, Coolblue, Amazon.nl, MediaMarkt
  - DE: idealo.de, Geizhals; Amazon.de, MediaMarkt, Otto
  - UK: PriceRunner, PriceSpy; Amazon.co.uk, Currys, Argos
- **Cross-border** (Standard/Deep, or when local is clearly overpriced): neighbouring markets' aggregators and Amazon stores. Flag shipping cost, harder cross-border returns, and regional variants (keyboard layouts, plug types, warranty region).
- **China direct** (only where sensible — generic items, accessories, consumables): AliExpress and similar. Flag delivery time, import VAT/duties (check current rules rather than assuming), and quality/counterfeit risk.

Compare total landed cost (item + shipping + any import costs), not sticker price.

## 5. Timing (Deep tier only)

Give a **buy now / wait** call based on:
- Price history (local aggregators' price graphs, Keepa/camelcamelcamel for Amazon) — is the current price high, normal or low?
- Upcoming model releases or announced successors.
- Nearby sale events (Black Friday, Prime Day, etc.) and whether this category actually discounts during them.

## 6. Output (verdict-first, in chat)

```
**Tier:** <Quick|Standard|Deep> — <assumptions made / requirements as understood>

**Pick: <product>** — <price> at <shop> (<date>)
<2–3 lines: why this one, for this use case>

| Option | Price | Key difference | Best for |
|---|---|---|---|
| Budget: ... |
| Upgrade: ... |
| Wildcard: ... |

**Where to buy:** <best 2–3 offers with links, landed cost>
**Caveats:** <weak evidence, known failure modes, compatibility notes>
**Timing:** <Deep only — buy now / wait, and why>
```

Quick tier collapses this to ~5 lines: tier + assumption, pick, price/shop/link, one-line why, any caveat.

## 7. Honesty

- If evidence is thin or conflicting, say so plainly rather than projecting confidence.
- If the honest answer is "the cheap generic one is fine", say that.
- If nothing meets the user's requirements within budget, say so and show the nearest options.
- Disclose when a source is affiliate-funded if it's being cited.
