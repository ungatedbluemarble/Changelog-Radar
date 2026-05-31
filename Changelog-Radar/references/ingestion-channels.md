# Ingestion channels

How each way of receiving a vendor's updates works, how to discover and validate it, and how to help a person set it up. The governing rule from the SKILL body applies throughout: investigate live, prefer sources within 30 days, confirm by fetching, never assert a channel's existence or absence from memory.

There are four channel types. A vendor may support several; the person picks which one they want.

## 1. RSS / Atom feed

The cleanest channel. A feed gives structured, dated, parseable entries that map directly onto the digest buckets.

**Discovery, in order:**
1. Search for the vendor's official changelog, release-notes, or "what's new" page and its feed. Queries like `<vendor> changelog rss`, `<vendor> release notes feed url`, `<vendor> developer changelog`. The goal is to surface the actual feed URL as a real search result, because it can only be fetched and validated if it came from a search (a guessed URL cannot be fetched).
2. Know the common conventions so you recognize a feed URL when search surfaces it, and so you can tell the user exactly what to look for: changelog/blog hosts often expose `/feed`, `/rss`, `/rss.xml`, `/atom.xml`, `/feed.xml`, `/changelog.rss`; Statuspage-style status pages expose `<status-host>/history.atom`; GitHub repos expose `https://github.com/<owner>/<repo>/releases.atom`. Do not assert one of these exists by constructing it; surface it via search or get it from the user.
3. If the feed URL surfaces in search, fetch it and validate (below).
4. If the vendor clearly offers a feed (the changelog page says so) but the exact URL does not surface in search, do not guess and do not stall. Default to the user: point them to the changelog page's subscribe control and ask them to paste the feed URL they see. They are the expert with the page in front of them. Once they paste it, fetch and validate.

**Validation (required before recording):** Fetch the surfaced or user-supplied URL. Confirm it returns XML that parses into entries each carrying a title, a link, and a date. A URL that 404s, returns HTML, or yields no dated entries is not a validated feed. Do not record it; go back to the user.

**Classification:** Check whether entries carry `<category>` tags. If they do, record that categories are available for filtering. If they do not, classification falls to reasoning over title and body, which the skill does at digest time.

**If no feed exists:** Say so only after searching. Then offer the person the email channel or, for feedless JS-rendered vendors, the search/scrape fallback.

## 2. Email notice

Used when the person prefers updates in their inbox, or already receives them there. The connected inbox becomes both the source and, for scheduled runs, the unattended trigger: the vendor's own notification is the push, so the person never waits on it.

**How it works:** The vendor must offer a subscription. The person subscribes. Notices land in the connected Gmail. The skill needs three things recorded: the sender identity to match on, a subject or body pattern that identifies a release notice (versus billing, marketing, support), and the rule for extracting the version or article identifier from the message.

**Where vendor subscriptions live (investigate to confirm current location):**
- A developer newsletter or changelog email opt-in, usually on the changelog or developer-docs page.
- Status-page subscriptions: Statuspage and similar let a person subscribe by email to incidents and scheduled maintenance.
- Account-level notification settings inside the product, for product-update emails.

**Validation:** Confirm the person actually receives the notice or has just subscribed. Do not record an email-trigger entry on the assumption that a subscription exists. If they have not subscribed yet, walk them to the specific opt-in and confirm afterward.

**Honest caveat to tell the person:** email means inbox volume, and folder filtering or a label is their mitigation, not something the skill controls. This is exactly why some people prefer RSS; let them choose.

## 3. GitHub releases (special case of feed, called out because it is so reliable)

For any tool hosted on GitHub, the releases Atom feed is often the best source even when the project also maintains a prose changelog, because it is always dated and always parseable. Record the repo and use `releases.atom`. Note that some projects tag without creating releases, in which case `tags.atom` is the fallback, though tags lack release notes bodies.

## 4. Search / scrape fallback

For vendors with no usable feed and a JavaScript-rendered changelog that returns blank on direct fetch. This is the Zoom case and the reason Zoom is the registry's worked example.

**How it works:** Record the search queries that surface the vendor's recent dated content and the specific fetchable URLs (individual entry pages often render even when the index does not). At digest time, run the searches, fetch the surfaced entry URLs directly, and extract dated changes from those.

**When the person hands over an identifier:** If the vendor emails a KB number or posts a version, the person can simply give it to the skill, which fetches that specific entry directly and filters it. This is the lightest-weight path and works for any vendor regardless of feed availability.

**Validation:** Confirm the searches actually surface dated, fetchable content. If even search cannot reach the vendor's notes, tell the person plainly and ask whether the vendor publishes anywhere reachable; do not fabricate a source.

## Recording the choice

Whatever channel is chosen and validated, record it in the registry per `registry-schema.md`: the channel type, the source identity or URL, the extraction rule, and the classification method. One vendor, one primary channel, though a vendor may have a secondary recorded if the person wants both (for example, RSS for features plus status-page email for incidents).
