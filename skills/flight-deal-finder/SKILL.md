---
name: "flight-deal-finder"
description: "Find the cheapest flights between a set of origin airports and a destination across date windows under hard constraints (stops, max duration, layover, no self-transfer), mix-and-match airports, and deliver a visual dashboard."
---

# Flight deal finder

Goal: the cheapest itinerary that satisfies every constraint, verified flight by flight, delivered as a visual dashboard (not a text-heavy doc).

## 1. Interview first (AskUserQuestion)
Ask in as many rounds as the spec needs, keeping it reasonable. Group related questions, and only ask what changes the search; skip anything the user already said. A typical split:

Round 1:
- Origin airports (multi-select). Offer nearby cross-border airports as an option, don't assume.
- Date format: two windows (outbound range + return range), window + trip length, or fixed pairs.
- Max stops per direction.
- Max travel time: per direction or outbound only.

Round 2:
- Exact outbound window and return window (free text via Other).
- Passengers and bags: cabin bag only / checked / under-seat only.
- Extra filters: min layover, no red-eyes, airlines to avoid.
- **"Best" threshold N (€):** how far above the cheapest a price can be and still count as best, shown in green. Offer €10 / €25 / €50, with €25 recommended.

Always confirm these defaults, because they change the answer:
- **No self-transfer**: each direction must be one ticket.
- **Mix-and-match allowed?** Fly out of one airport and back into another. Default to yes, and price every pairing.
- **Complete airport list.** Check which origin airports actually have scheduled passenger service, using a quick web search for any you're unsure of. Say which ones you excluded and why.

Restate the full spec in one line, then start.

## 2. Search method (built-in browser + Google Flights)
- Ask for site access once with scope `site` (google.com). If approvals keep coming back as one-time only, tell the user to choose "always allow".
- Reject non-essential cookies on the consent screen.
- **Use query URLs, not the form:**
  `https://www.google.com/travel/flights?q=One%20way%20flights%20from%20LHR%20to%20ATH%20on%202027-03-14&curr=EUR&hl=en`
  - Multi-airport `q=` strings and filtered `tfs=` search URLs often show "Oops, something went wrong". Click Reload once; if it fails again, fall back to one airport per query.
  - Some airport codes are misparsed (e.g. ENS). Check the page title says the right city pair.
- **Price each direction as a one-way**, for every airport × date. Any combination's price is then outbound + return, so this covers all open-jaw pairings.
  - Afterwards, also check round trips that start and end at the same airport. Sometimes a single ticket is cheaper. Note it if not.
- **Don't trust the date grid or price graph alone.** They ignore duration and layover limits and show stale or impossible fares. Use them only to narrow dates, then verify by reading the actual flights.
- Keep browser batches under about 45 s (roughly 3 page loads per batch).

### Extractor (run on a results page)
Parses each flight's aria-label and applies the hard constraints. Change the limits to match the spec.
```js
(()=>{const L=[...new Set([...document.querySelectorAll('[aria-label^="From "]')].map(e=>e.getAttribute('aria-label')))];const all=L.map(s=>{const p=+((s.match(/From ([\d,]+) euros/)||[,'0'])[1]).replace(/,/g,'');const stops=/Nonstop/.test(s)?0:+((s.match(/(\d) stops? flight/)||[0,9])[1]);const d=s.match(/Total duration (\d+) hr(?: (\d+) min)?/);const dur=d?(+d[1])*60+(+(d[2]||0)):9999;const lays=[...s.matchAll(/is a (?:(\d+) hr)?(?: ?(\d+) min)? layover/g)].map(m=>(+(m[1]||0))*60+(+(m[2]||0)));const sep=/Separate tickets|self.transfer|Transfer here/i.test(s);const air=(s.match(/flight with ([^.]+)\./)||[])[1];const t=(s.match(/Leaves (.+?) at ([\d:]+ [AP]M) on \w+, (\w+ \d+)/)||[]);return {p,stops,dur,lays,sep,air,dep:t[2],day:t[3]}});const ok=all.filter(f=>f.stops<=1&&f.dur<=600&&f.lays.every(x=>x>=60)&&!f.sep).sort((a,b)=>a.p-b.p);return document.title+` n=${all.length} valid=${ok.length}: `+(ok.slice(0,3).map(f=>`€${f.p} ${f.air} ${f.day} ${f.dep} ${Math.floor(f.dur/60)}h${String(f.dur%60).padStart(2,'0')} lay${f.lays}`).join(' | ')||'NONE')})()
```
Rules it encodes:
- "Separate tickets" inside one direction counts as a self-transfer: exclude it.
- "Transfer here" means changing airports during the layover: exclude it.
- Separate tickets for outbound and return are fine, but say so in the results.

## 3. Extras: cabin bag and flexible (cancellable) fares
Budget fares often exclude the overhead bin; Google shows "price does not include overhead bin access". Always price a cabin bag and a flexible fare as add-ons.
- **Airlines with fare tiers in Google Flights** (e.g. Pegasus: Light / Saver / Saver Plus / Comfort Flex): click the cheapest valid flight. The booking page lists each tier with its price and what it includes. The extra cost over the basic tier is usually the same across flights on a route. Spot-check a few flights and apply the extra cost to the rest, labelled as inferred.
- **Airlines whose add-ons aren't priced in Google Flights**: use the airline's own pages (extra-baggage page for the cabin-bag price; flex/cancellation add-on page). Add-ons advertised "from" a minimum go in as lower bounds (≥).
- **Bot checks / CAPTCHAs** (e.g. "Press & Hold"): never solve them. Ask the user to do it in the browser pane, and carry on with other airlines in another tab meanwhile.

## 4. Deliverable: visual dashboard (Data artifact type)
Visuals, not prose, and a calm, designed look, not an "AI-coded" one. If you've already built one of these dashboards, reuse its `index.html` as the template, replacing only the data files and the route-specific words. Don't redesign from scratch unless asked.

Data: fixed JSON files.
- `oneway`: direction, airport, date, price, airline, departs, duration_min, layover_min, via. `price: null` means nothing fits.
- `extras`: cabin_add, flex_add, how it was priced, checked or inferred.
- `alternatives`: other ways to book.

All totals and differences are `dash.calc`s, never worked out in a script:
- `combos`, `combosCabin`, `combosFlex`: every out × back pair, with `total` and `delta` (the difference from the cheapest).
- `legs`: per one-way, the fare, cabin and flex prices and their deltas from the cheapest.

Layout, top to bottom, inside a centred column (max-width ~880px, `margin: 0 auto`):
1. **Header**: title, then one sentence that leads with the **recommendation**, never with a different "cheapest" option. Only when another combination is cheaper, add "Out from X is €N cheaper (€M), but …" as a conditional drawing. Constraints go on one muted meta line; no chips.
2. **Boarding-pass card**: route in large serif (e.g. LHR ⇄ ATH), the price, a dashed perforation line, one row per leg (date, time, airline, flight time, via, price), and a bottom strip with "With cabin bag" and "Flexible, cabin bag included" totals.
3. **Every airport combination**, with a segmented switch: Fare only / + Cabin bag / Flexible. The switch redraws the price line, the matrix and both date tables.
   - **Price line** above the matrix: every combination as a dot on one price axis. Nearby prices share one range label above and one "N options" label below.
   - Dots are r≈7 (cheapest r≈9), nudged apart to at least 22px between centres, kept in price order, and never resized by price.
   - Hovering a dot highlights its matrix cell and the reverse (`data-key` on cells, `data-k` on dots).
   - **Matrix**: rows are out-from, columns are back-to, with the city name under each code. Each cell shows the total and, beneath it, "best" or "+€delta".
4. **Fares by date**: outbound and return one-way tables in the same style. Each cell shows the price and "best" or "+delta"; the airline, time, duration and stopover go in a tooltip.
5. **Folded `<details>`** at the bottom:
   - "Other ways to book": bar chart with each label on its own line above its bar, and the recommended bar in ink.
   - "Notes and method".

### Visual rules (learned the hard way)
- **Encode price with ink weight, not colour fills.** The cheapest cell is full ink; others fade linearly to about 38% opacity at the most expensive. No heatmap fills, no traffic lights, no blue gradients: they read as punchy or muddy.
- **One accent colour, only for "the answer":** deep green, `--fx-acc: #1d7a5a`, or `#5cc49a` under `prefers-color-scheme: dark`. It appears only on:
  - the recommended price and the "Recommended" label
  - the active switch button
  - every "best" cell and its sub-label
  - the best dots on the price line and the cheapest group's label
  - the recommended bar in "Other ways to book"
  Everything else stays greyscale.
- **Best = within N euros of the cheapest** in that table or line, where N comes from the interview. Store it as one constant in the page script (`BEST_WITHIN`), use it in an `isBest(v, min)` helper, and draw it into the footnote. The cheapest cell's sub-label says "best"; others within N show their "+delta" in green.
- No chips, colour legends, card grids of notes, or repeated captions. One short footnote under the matrix explains it: "Darker means cheaper; small numbers are the extra cost over the cheapest. Green: within €N of the cheapest. ≥ means at least."
- Charts inside a closed `<details>` have zero width, so text measures as 0. Skip drawing while `clientWidth` is 0, and call `dash.draw(name)` on the `toggle` event when the section opens. Prefer layouts that don't need text measuring.
- Mark every number: put `data-source`/`data-field`/`data-where` on the value span (and on delta spans), not on the cell.

### Design process
- Before any redesign, **make a backup copy** first: a new Data artifact with the same datasets (re-upload the JSON assets) and the same `index.html`.
- Give the user design options to pick from (AskUserQuestion with previews, or inline Visualizer mock-ups on the real numbers) before rebuilding. They want a say in the design.
- When they reject a direction, rethink the encoding itself (ink, bars, dot size, position), not just the palette.

## 5. Hygiene
- Re-check the winning flights at the end. Prices drift within an hour; flag any change.
- State what wasn't verified: airlines not covered, fares inferred from the pattern, airlines that only give "from" prices.
- Cite sources (Google Flights, airline pages).
- Never book or enter payment details.
- Keep chat replies short. The details go in the dashboard.
- After each dashboard write, check the artifact rendered without errors before handing over.