# Output formats

Two modes. The default for every digest is the table (Mode A). Choose the engineering publication (Mode B) only when the person explicitly asks to publish, share with engineering, format for the team, or produce a changelog diff.

Both obey the writing rules in the SKILL body: no em dashes, no emojis, sentence case, summarize rather than quote.

## Mode A: the table (default)

After filtering, the entire output is a single table. Nothing else: no preamble, no buckets, no severity tiers, no prose summary before or after, no list of suppressed items. Just the table. The person wants to scan it, not read around it.

Two columns:
- **Column one**: provider and service, e.g. "Twilio SMS" or "Twilio, SMS".
- **Column two**: the update, two to three sentences maximum, only the detail critical to acting on it. Lead with the consequence (deprecation, deadline, breaking change, new capability), not the marketing framing.

One row per update. Structure the rows like this:
- Each relevant update is its own row. A service with several relevant updates this cycle gets several consecutive rows, repeating the provider and service in column one for each, one update per row.
- When a service's updates are exhausted, move to that provider's next service, then to the next provider.
- A monitored service with nothing relevant this cycle gets exactly one row, with column two reading "no updates available".

So a table monitoring Twilio SMS (three updates), Twilio Phone (one update), and Twilio Video (none) is five rows: three SMS rows, one Phone row, one Video row reading "no updates available". The table's length is the total number of filtered updates plus one row for each service that had none. Repeat "Twilio SMS" in column one for each of the three SMS rows rather than merging cells.

Every update in the table has already passed the only legitimate filter: is it about a service the person monitors. Within an in-scope service, include every update the cycle surfaced, one row each. Do not rank, prioritize, or drop updates based on a judgment of how important or minor they are. That judgment belongs to the person reading the table, not to the skill, and silently dropping "minor" updates is a bias toward a shorter output, not a service to the user. If a monitored service has nine updates this cycle, the table has nine rows for it. The two-to-three-sentence limit governs the density of each row, never whether a row appears. Summarize each update accurately in column two without soft-pedaling or inflating it; if a critical detail needs more room, link to the source inside the cell rather than expanding the prose or omitting the row.

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
- Footer: source attribution.

Title guidance: concise and descriptive of the specific changes, e.g. "Developer platform changelog: Phone API, Meeting SDK, and webhooks." Lead paragraph in prose, no em dashes.

Include only items relevant to the person's stack. Omit pure product-release items (anything with no direct API or SDK impact) from the engineering artifact.

A neutral starting stylesheet lives in `assets/engineering-template.html`. Adapt its header labels and accent variables to the vendor and the person's own branding if they have one.
