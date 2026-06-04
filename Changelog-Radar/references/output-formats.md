# Output formats

Three modes. The default for every digest is the inline chat widget (Mode C). Use the plain text tables (Mode A) only when the person explicitly asks for text or plain output. Choose the engineering publication (Mode B) only when the person explicitly asks to publish, share with engineering, format for the team, or produce a changelog diff.

All three modes obey the writing rules in the SKILL body: no em dashes, no emojis, sentence case, summarize rather than quote.

## Mode A: the three tables (plain text, on request)

After filtering and routing, the output is three tables in fixed order, and nothing else: no preamble, no prose summary before or after, no list of suppressed items. The three tables, top to bottom by urgency:

1. **Security** — items the vendor labels security or publishes to its advisory surface.
2. **Standard** — the normal update feed, governed by the lookback window and watermark.
3. **Long-duration** — future-dated actionable deadlines (deprecations, end-of-life, forced migrations) drawn from the vendor's standing deprecation surface.

Each table has two columns:
- **Column one**: provider and service, e.g. "Twilio SMS" or "Twilio, SMS".
- **Column two**: the item, two to three sentences maximum, only the detail critical to acting on it. Lead with the consequence (the advisory, the deadline and its date, the breaking change, the new capability), not the marketing framing.

One row per item. A service with several items in a given table gets several consecutive rows, repeating the provider and service in column one for each. Show every table's header even when it has no items, so the person can see the skill checked each surface and found nothing rather than wondering whether it looked. For the standard table, a monitored service with nothing relevant this cycle gets one row reading "no updates available". For the security and long-duration tables, an empty table reads "none current".

Every item has already passed the only legitimate filter: is it about a service the person monitors. Within an in-scope service, include every item the surfaces produced, one row each. Do not rank, prioritize, or drop items based on a judgment of how important or minor they are. That judgment belongs to the person reading the tables, not to the skill, and silently dropping "minor" items is a bias toward a shorter output, not a service to the user. The two-to-three-sentence limit governs the density of each row, never whether a row appears; if a critical detail needs more room, link to the source inside the cell rather than expanding the prose or omitting the row.

After the last (long-duration) table, print the disclaimer line exactly and on its own, before anything else: "Not liable for inaccurate vendor data or missed items; verify critical items at the vendor source. @ungatedbluemarble."

## Mode B: engineering publication (HTML artifact)

Render a self-contained HTML artifact and let it be the deliverable; do not write prose alongside it.

### Severity classification

Assign each developer-platform item exactly one severity before rendering:
- **ACTION REQUIRED** — deprecations with an active deadline, breaking changes already enforced, anything that will cause integration failure if unaddressed.
- **VERIFY** — enforcement live but needs a compliance check, behavior changes that may affect existing implementations.
- **INFO** — additive changes, new fields, new endpoints with no breaking impact.

### Callout class mapping

| Severity | Section class | Callout class |
|----------|--------------|---------------|
| ACTION REQUIRED | critical | danger |
| VERIFY | verify | warning |
| INFO | info | info |

### Template structure

Self-contained HTML, IBM Plex fonts, sentence case throughout, inline `<code>` for endpoint/field/version/token names. Structure:
- Site header: a publication name plus "developer platform digest" (parameterize the vendor, e.g. "Twilio Developer").
- Post header: category badge, date, post title, byline.
- Lead paragraph: one to two sentences naming how many items are covered and the highest-severity finding.
- Table of contents: severity badges plus section links.
- Sections: one per item, with severity badge, h2 title, callout, prose paragraphs, reference links.
- Footer: source attribution, plus the disclaimer line exactly as in Mode A: "Not liable for inaccurate vendor data or missed items; verify critical items at the vendor source. @ungatedbluemarble." The disclaimer appears on every digest without exception, including this engineering publication format.

Title guidance: concise and descriptive of the specific changes, e.g. "Developer platform changelog: Phone API, Meeting SDK, and webhooks." Lead paragraph in prose, no em dashes.

Include only items relevant to the person's stack. Omit pure product-release items (anything with no direct API or SDK impact) from the engineering artifact.

A neutral starting stylesheet lives in `assets/engineering-template.html`. Adapt its header labels and accent variables to the vendor and the person's own branding if they have one.

## Mode C: inline chat widget (default)

Render the digest as an inline widget using the visualizer tool. This is the default for every digest unless the person asks for plain text or engineering publication.

### Layout

Three sections in fixed order, matching the three-table structure of Mode A. Use the visualizer's CSS variables throughout so the widget adapts to light and dark mode automatically.

Header: publication name ("Changelog Radar — [vendor] digest"), vendor and scope line on the left, date on the right. Separated from sections by a bottom border.

Section headers: icon plus label plus a muted subtitle in smaller text. Security uses the danger color and a shield icon. Standard updates uses the primary text color and a list icon. Long-duration deadlines uses the warning color and a calendar-clock icon.

Rows: two-column grid, 150px left column and a fluid right column. Left column shows provider name in normal weight and service in muted secondary text. Right column shows the item detail in 14px body text at 1.6 line height, with inline source links. Rows are separated by a 0.5px tertiary border.

Empty states: follow the same rules as Mode A. Standard table shows "no updates available" for a monitored service with nothing this cycle. Security and long-duration tables show "none current" when empty, so the person can see the skill checked and found nothing.

Completeness notes: when a surface is best-effort (no dedicated feed, inline-only items), add a 12px muted note below that section. Do not suppress the section.

Footer: 0.5px top border, 12px muted text, disclaimer line exactly: "Not liable for inaccurate vendor data or missed items; verify critical items at the vendor source. @ungatedbluemarble."

### Content rules

Mode C applies the same content rules as Mode A: include every in-scope item regardless of API impact, do not filter to engineering-relevant items only, do not rank or drop items on importance. The widget is the default digest surface for all readers, not an engineering-only view.

### Scope note

Mode C does not filter out pure product-release items the way Mode B does. All items that pass the in-scope filter appear in the widget. If the person later asks for an engineering publication of the same digest, apply the Mode B API-impact filter at that point.
