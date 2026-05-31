---
name: changelog-radar
description: Interviews a person about the products and services they actually use, then fetches each vendor's release notes or changelog and filters them down to only the changes relevant to that person's stack, in plain language. Use this skill whenever someone wants to track updates from a software vendor or set of vendors, asks "what changed in X", "did Twilio update their SMS API", "set up update tracking for my tools", "watch this service's changelog", "give me today's vendor updates", "what's relevant to me in this release", or any request to monitor, digest, or personalize software release notes. Also trigger when the user names a service and asks how to receive its updates (email, RSS, or otherwise), when they want to add a new service to track, or when a scheduled task fires to deliver a periodic vendor-update digest. Always use this skill rather than answering changelog questions from memory, because vendor release notes change constantly and must be fetched live.
---

# Changelog Radar

## What this does and why it is different

Most changelog tools dump a vendor's entire release feed at the reader, or match crude keywords. This skill does something keyword matching cannot: it interviews the person as the expert on their own stack, learns what they actually build and depend on, then reads a vendor's full changelog and returns only the items relevant to that person, explained in plain language. The person is the source of truth about their own needs. The skill's job is to interview well, source updates reliably, and filter by genuine relevance rather than string matching.

Two persistent ideas, both verified against live sources, never asserted from memory:
- **The registry** (shared, reusable, lives in the user's repo): for each vendor, how to reach its updates and what its product areas are.
- **The interest profile** (per person): which of a vendor's areas and components matter to them. This is normally elicited fresh each run, because the human regenerates it by talking. It may optionally be saved as a convenience file, but the skill never depends on a stored profile.

## Core operating rule: investigate, never guess

When any question of "does this vendor offer RSS / email notices / a changelog" comes up, the skill investigates live before answering. It runs layered web searches across the vendor's support articles, developer docs, status page, and release-notes pages, prefers sources within the last 30 days, and falls back to the closest available date only after confirming the tool or feed is still offered. It never tells a person a channel does not exist based on recollection. Acceptable outputs are "I searched these sources and confirmed this feed at this URL by fetching it" or "I searched these sources, nothing within 30 days offered a feed, here is where their subscription setting lives instead." Never "I don't think they have one."

## Workflow selection

Three entry points. Read the cue and branch.

1. **Onboarding a vendor** — the person names a service to track, or has none registered yet. Go to "Onboarding a vendor."
2. **Running a digest** — the person wants updates for one or more already-registered vendors, or a scheduled task fired, or they hand over a specific changelog entry / KB number / version. Go to "Running a digest."
3. **Managing the registry or profile** — the person wants to see, edit, or remove tracked vendors, change their interests, or refine what a digest surfaces (for example, reacting to a digest with "I don't need Video" or "stop showing me SDK version updates, I'm pinned to a version"). Go to "Refining the profile."

If no registry exists yet, say so plainly and offer to onboard the first vendor. Do not invent a registry.

## The bundled registry is an example, not a monitoring list

The file at `assets/registry.example.yaml` ships with worked entries (Zoom, Anthropic) purely to document the format and demonstrate the feedless search/scrape case. These are examples, not an active monitoring list. A fresh install monitors nothing until the person onboards their own vendors. Never run a scheduled or on-demand digest against the example entries just because they are present in the bundle: a person who installed this skill did not ask to track Zoom or Anthropic, and surfacing updates for products they may not use is exactly the noise this skill exists to prevent. Treat the person's own onboarded vendors as the only active registry. If the person has onboarded nothing yet and a digest is requested, report that there are no monitored vendors and offer to onboard one, rather than falling back to the examples.

## Refining the profile

The profile is the local filter, and the person sharpens it conversationally over time. When they react to a digest by saying an item or service is not useful to them, that is a standing instruction to stop surfacing it. Record it as an explicit exclusion in the profile per `references/registry-schema.md`, at whichever level they meant:
- A whole service they no longer want: add it to `exclude.services`.
- A component muted within a service they keep: add it to `exclude.components` with the service, what to match, and their reason. The case to support is "I use Twilio Video but I'm bound to a specific SDK version, so I don't need new SDK release notes": Video stays in scope, SDK version updates are muted.

Two rules keep this honest. First, exclusions are always user-initiated. The skill records what the person explicitly asked to mute; it never adds an exclusion on its own judgment that something is minor, because that is the bias the skill exists to avoid. Second, a muted item must never become permanently invisible without the person being able to see it. On request, show the person their active exclusions and let them lift any of them. When unsure whether a passing comment was a real standing instruction, confirm before writing it.

Because the profile lives in the person's repo for distribution, the skill cannot push to it. After updating the profile, hand the changed file back for the person to save or commit, the same as for registry changes.

## Onboarding a vendor

The goal is a validated registry entry plus an interest profile. Do not write anything to the registry until a source is confirmed working.

### Step 1: Identify the vendor and what the person builds

Ask what the product is and what the person actually does with it. Pull out specifics, not just a name: an engineer who says "I build SMS tools on Twilio" should be drawn out on whether they touch Messaging Services, A2P 10DLC registration, short codes, delivery webhooks, specific error codes. These specifics are what filtering matches against later, so a thin answer here produces a noisy digest. Ask one focused question at a time.

### Step 2: Establish how they receive updates today and how they want to going forward

These are two separate questions and the answers can differ. Ask both:
- How do you get this vendor's updates today: already receiving emails, already watching an RSS feed, or nothing yet?
- How do you want to receive them going forward?

The person chooses the channel. The skill does not pick for them. If they want a channel that is not yet set up, the skill walks them through establishing it. For the real per-channel workflows, including how email subscription, RSS discovery, GitHub releases feeds, status-page feeds, and search/scrape fallback each work and how to set them up per vendor, read `references/ingestion-channels.md`.

### Step 3: Discover and validate the source

Investigate per the core operating rule. Find the actual current source for the chosen channel and prove it works:
- **RSS/Atom**: locate the real feed URL, fetch it, confirm it parses into dated entries. A feed that 404s or returns no dated entries is not validated.
- **Email**: confirm the person receives or has subscribed to the vendor's notice; capture the sender identity and a subject/body pattern, and the rule for extracting the version or article identifier.
- **Search/scrape fallback**: only for vendors with no usable feed (Zoom is the canonical example). Record the search queries and fetch targets that do surface dated content.

If no source can be confirmed, do not write a half-entry and do not fabricate a URL. Explain what was searched and guide the person to where the vendor's feed or subscription setting lives so they can supply or enable it.

### Step 4: Map the vendor's product areas and set the lookback window

Once a source is confirmed, capture the vendor's product-area taxonomy (for Twilio: SMS, Voice, Video, Verify, Lookup, etc.). This is what the interest profile selects from and what entries get classified against. If the feed provides category tags, record that they exist; if not, note that classification will rely on reasoning over title and body.

Then ask how far back the person wants the skill to check for updates: one week, thirty days, ninety days, or another span they name. This is the lookback window, a stable per-vendor preference recorded in the profile and applied every run. It is the floor the skill always respects: a digest never reaches further back than the window, and never further back than the vendor's source actually provides. Ask it plainly; do not assume a default without telling the person what you assumed.

### Step 5: Write the registry entry

Only now, with a confirmed source, write the entry to the registry file following `references/registry-schema.md`. Record channel type, source identity or URL, extraction rule, product-area taxonomy, and a classification method (feed categories vs. semantic reasoning). Then capture the person's interest areas from Step 1 into the profile.

Because the registry lives in the person's repo for distribution, the skill cannot push to it. After writing the updated registry content, hand it to the person and tell them plainly it is theirs to save or commit. See `references/repo-and-distribution.md` for what the repo is and is not.

## Running a digest

A digest can be run on demand, or on a schedule. On-demand works on any plan, including free: the person asks and the digest runs. Automated scheduling is a paid-plan capability through Claude Cowork: the person asks Cowork to run this digest on a cadence (for example "every weekday at 9am"), and Cowork runs the skill natively on that schedule because scheduled tasks have access to installed skills. There is nothing extra to build for this. The highest-value pattern is that scheduled morning summary, because changelog awareness is prevention work that busy people do not do proactively. Note honestly that Cowork scheduled tasks run while the desktop app is available, so timing is best-effort rather than guaranteed-clock, and genuinely unattended machine-off delivery would need a hosted runner outside this skill. The digest logic below is identical whether run on demand or on schedule.
### Step 1: Load context

Read the registry for the vendor(s) in scope. Establish the interest profile: use the profile from the conversation if the person has just been interviewed, elicit it briefly if not, and treat the human as the authority on their own needs. If the person handed over a specific identifier (a KB number, a version, a changelog URL), that is the target; fetch it directly.

Set the time bounds. Read the vendor's `lookback_window` from the registry; this is the floor a digest never reaches past. Then recover the last-checked watermark using past-chat search: look up the last time a digest ran for this vendor and treat that as the "since last time" line. A normal digest shows updates dated after the watermark but within the window. If past-chat search finds no prior digest for this vendor, there is no watermark, so show the full window rather than hiding anything. Honor explicit overrides: "show me everything" ignores the watermark and shows the full window; a stated span like "last 90 days" is a one-off range. Overrides never change the stored window. See `references/registry-schema.md` for how the window and watermark combine.

### Step 2: Fetch updates from each vendor's channel

Pull from the channel recorded in the registry: fetch and parse the RSS/Atom feed; or read the vendor notice from the connected inbox and extract the identifier (this is what lets a scheduled task run unattended, the email is the trigger, the person does not wait on it); or run the search/scrape fallback for feedless vendors. Always fetch live. Investigate per the core operating rule if a recorded source has gone stale.

Keep only dated change records. The skill monitors for updates, so an item counts only if it is an actual change with a release or publication date in the cycle: a changelog entry, a release note, a deprecation notice, a dated announcement. Discard reference material even when it looks relevant: API documentation, guides, how-to pages, and behavior descriptions are not updates, and surfacing them as if they were is inventing signal where there was none this cycle. This matters most for the search/scrape channel, which can return docs and guides alongside changelog entries; RSS and email rarely raise the problem because their items are inherently dated changes. If a monitored service has no dated change this cycle, its row reads "no updates available" rather than a note pulled from documentation.

### Step 3: Read everything, filter by service scope

Read the full set of changes. Keep every update that concerns a service the person monitors. The filter is about scope, not importance: it decides whether an update belongs to a service the person tracks, not whether the skill judges the update significant enough to mention. Once a service is in scope, every update to it is included; assessing which ones matter is the person's call when they read the table, not the skill's when it builds it.

Judge scope by what the update actually concerns, not by keyword presence: an update that never names the person's keyword but changes a service they monitor is in scope; an update that mentions their area in passing but actually concerns a different service is not. When in doubt about scope, keep the update and let it appear rather than silently dropping it. Never drop an in-scope update because it seems minor.

Then apply the person's standing exclusions from the profile. A service in `exclude.services` is suppressed from the table; a component matching an `exclude.components` rule is suppressed within its still-in-scope service (the "Video but not the new SDK" case). Exclusions are the person's own mutes, so honoring them is not the skill making an importance judgment. Still evaluate excluded items against the feed each cycle rather than skipping them blind, so that if the person later lifts a mute, nothing was missed, and so you can answer if they ask what their exclusions are currently hiding.

### Step 4: Organize and output

Render the filtered updates. The default and almost always correct output is a single scannable table, one row per update, provider and service in column one and the critical detail in two to three sentences in column two. For the exact table rules, the optional engineering HTML publication mode, and when to use each, read `references/output-formats.md`. Use the table unless the person explicitly asks to publish or format for engineering.

## Writing rules (all output)

Prose, not bullet lists, unless the person asks otherwise. No em dashes; use a colon or plain connective language. No emojis or icons. Lead with the date and the most recent confirmed version where relevant. Flag operationally significant items: deprecations with deadlines, breaking changes, anything requiring explicit enablement, anything affecting the person's specific components. Close with the source URL(s) used so the reader can drill in. Never reproduce large verbatim blocks from a vendor's notes; summarize in your own words and link out.

## Notes and known limitations

- Some vendors abandon or under-maintain their RSS feeds. Validate every feed by fetching it; do not trust that a conventional path exists.
- JavaScript-rendered changelog pages (Zoom's support KB, some developer changelog indexes) return blank on direct fetch. Use the search/scrape fallback documented in the references.
- The skill cannot push to the person's repo. Registry changes are handed back for the person to commit. This is by design: the repo is for distribution, not a live datastore.
- On-demand digests work on any plan, including free. Automated scheduling is a paid-plan capability via Claude Cowork, which runs the skill natively on the cadence the person describes. For email-triggered vendors, the connected inbox is the source; the schedule is the trigger.
- The registry ships with two real example entries (Zoom and Anthropic) so the format is self-documenting and the feedless search/scrape case is demonstrated. They are examples only, not a monitoring list; everything tracked is added through onboarding.
