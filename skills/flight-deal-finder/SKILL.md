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
  - Some airport codes are misparsed, especially small regional ones. Check the page title says the right city pair.
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

## 4. Deliverable: the fixed dashboard template
The dashboard's look ships with this skill in `assets/dashboard.html`. Every run fills that file with data. Never design or restyle the page during a run; to change the design, see "Design changes" below.

1. **Type:** run the Artifact quickstart with `intent: "other"` and create the dashboard from the **Data (Dashboard) type**. Never use a Doc or a plain HTML page.
2. **Data:** upload three JSON files and add each one as a dataset (`kind: "file"`, `format: "json"`, `fixed: true`, plus `updated`). Use exactly these ids and fields, because the template reads them:
   - `oneway`: one row per direction × airport × date, with direction (`"out"`/`"back"`), airport, date (`YYYY-MM-DD`), price, airline, departs, duration_min, layover_min, via and checked. `price: null` means nothing fits.
   - `extras`: one row per priced `oneway` flight, with direction, airport, date, airline, cabin_add, flex_add, cabin_how, flex_how and checked (a date, or "inferred"). An unknown add-on is `null`.
   - `alternatives`: option, price, note.
3. **Page:** write `files/index.html` as a verbatim copy of `assets/dashboard.html`. Change only the `CONFIG` block at the top of the scripts:
   - title and destination
   - airports, in matrix order
   - the recommended out/back pair
   - the header, "why" and "cheaper, but" sentences
   - the meta line
   - `bestWithin`: the "best" threshold N from the interview
   - `flexFromAirlines`
   - the notes
   
   Nothing below `end CONFIG` changes. The template already declares every total and difference as a `dash.calc` (`combos`, `combosCabin`, `combosFlex`, `legs`), so don't add calcs or work numbers out in a script.

The page has five parts: a header that leads with the recommendation, a boarding-pass card, every airport combination (price line and matrix, with a Fare / + Cabin bag / Flexible switch), fares by date, and folded "Other ways to book" and "Notes and method" sections.

### Design changes
- Change the look only when the user asks. Show them options first (AskUserQuestion with previews, or Visualizer mock-ups on the real numbers); they want a say in the design. If they reject a direction, rethink the encoding itself (ink, bars, dot size, position), not just the palette.
- Before rebuilding, make a backup copy: a new Data artifact with the same datasets and the same `index.html`.
- Make every approved change in `assets/dashboard.html` in the skill's repo and commit it, then copy it to the artifact. The next run then starts from the new design. Where the repo can't be written (e.g. claude.ai), give the user the updated file to commit.

### Design rationale (why the template looks this way; keep it when changing it)
- **Price is shown with ink weight, not colour fills.** The cheapest cell is full ink; others fade to about 38% opacity at the most expensive. No heatmap fills, traffic lights or gradients.
- **One accent colour, deep green (`--fx-acc`), used only for "the answer":** the recommended price and label, the active switch, the "best" cells and dots, and the recommended bar. Everything else is greyscale.
- **Best = within `bestWithin` euros of the cheapest** in each table or line, and the footnote states this.
- **The header leads with the recommendation, never with a different "cheapest" option.** The "X is €N cheaper, but …" sentence appears only when another pair is cheaper.
- No chips, colour legends, card grids or repeated captions. One footnote under the matrix explains the encoding.
- Charts in a closed `<details>` have zero width, so they're drawn on the `toggle` event. Every number carries `data-source`/`data-field`/`data-where` on its value span.

## 5. Hygiene
- Re-check the winning flights at the end. Prices drift within an hour; flag any change.
- State what wasn't verified: airlines not covered, fares inferred from the pattern, airlines that only give "from" prices.
- Cite sources (Google Flights, airline pages).
- Never book or enter payment details.
- Keep chat replies short. The details go in the dashboard.
- After each dashboard write, check the artifact rendered without errors before handing over.