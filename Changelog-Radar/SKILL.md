---
name: changelog-radar
description: Interviews a person about the products and services they actually use, then fetches each vendor's release notes or changelog and filters them down to only the changes relevant to that person's stack, in plain language. Use this skill whenever someone wants to track updates from a software vendor or set of vendors, asks "what changed in X", "did Twilio update their SMS API", "set up update tracking for my tools", "watch this service's changelog", "give me today's vendor updates", "what's relevant to me in this release", or any request to monitor, digest, or personalize software release notes. Also trigger when the user names a service and asks how to receive its updates (email, RSS, or otherwise), or when they want to add a new service to track. Always use this skill rather than answering changelog questions from memory, because vendor release notes change constantly and must be fetched live.
---

# Changelog Radar

## What this does and why it is different

Most changelog tools dump a vendor's entire release feed at the reader, or match crude keywords. This skill does something keyword matching cannot: it interviews the person as the expert on their own stack, learns what they actually build and depend on, then reads a vendor's full changelog and returns only the items relevant to that person, explained in plain language. The person is the source of truth about their own needs. The skill's job is to interview well, source updates reliably, and filter by genuine relevance rather than string matching.

Two persistent ideas, both verified against live sources, never asserted from memory:
- **The registry**: for each vendor, how to reach its updates and what its product areas are. In this chat version the registry is not stored in a file. It is built fresh through onboarding and recovered on later runs by searching the person's own past conversations (see "How this version remembers").
- **The interest profile** (per person): which of a vendor's areas and components matter to them. This is normally elicited fresh each run, because the human regenerates it by talking, and is recovered the same way the registry is.

## How this version remembers

This is the standard-chat version of the skill and it is stateless by design. It writes no registry file, no profile file, and nothing to any repo. It does not put anything into the person's account instructions. Continuity between sessions comes from one mechanism only: when the person asks for a digest against a vendor, the skill quietly searches the relevant past conversations to recover that vendor's setup, the same silent past-chat search it uses to recover the watermark. This recovery runs only when there is a vendor in scope to recover, never on a bare invocation, and it is never narrated to the person. If the search finds the prior setup, the digest proceeds against it. If it does not, the skill onboards from scratch.

This has one hard consequence the person must be told plainly, both at onboarding and whenever relevant: because the only record of their tracked vendors lives in the conversations where they set them up, deleting those conversations erases the setup. There is no backup file and no stored copy to fall back on. If the person deletes the chat history that holds their onboarding, the skill cannot recover their vendors and they will have to onboard again from the beginning. State this directly; do not soften it into a vague suggestion. The person who wants their setup to survive guaranteed, independent of their chat history, should use the Projects version of this skill, where the registry persists as a Project file. That is the correct fix, not a backup file.

## Core operating rule: investigate, never guess

When any question of "does this vendor offer RSS / email notices / a changelog" comes up, the skill investigates live before answering. It runs layered web searches across the vendor's support articles, developer docs, status page, and release-notes pages, prefers sources within the last 30 days, and falls back to the closest available date only after confirming the tool or feed is still offered. It never tells a person a channel does not exist based on recollection. Acceptable outputs are "I searched these sources and confirmed this feed at this URL by fetching it" or "I searched these sources, nothing within 30 days offered a feed, here is where their subscription setting lives instead." Never "I don't think they have one."

## On invocation: determine intent before doing anything

When the skill is invoked, read it and determine what the person actually wants before taking any action. Do not run a past-chat search, and do not fetch anything, until there is a concrete target: either a named vendor, or an explicit request to run a digest against vendors already in scope. A bare invocation with no vendor and no request is not a run. In that case, do not search anything: greet briefly, state in one line what the skill does, and ask what they want to track or whether they want a digest. Searching the person's history on a bare invocation is exactly the noise this skill exists to prevent, so it never happens.

Recovery by past-chat search is a silent internal step, never narrated. The skill does not announce that it is searching past conversations, does not list what it found, and does not recite build history or unrelated chats. Recovering setup is like reading its own files: it happens quietly and only its result is used. If recovery finds the prior setup, use it. If it finds nothing, proceed to onboarding without reporting the search. The person should never see the mechanism, only the outcome.

## Workflow selection

Three entry points, reached only after the invocation gate above has established intent. Read the cue and branch.

1. **Onboarding a vendor** — the person names a service to track. Go to "Onboarding a vendor." Onboarding never triggers recovery: a newly named vendor is treated as new, and only if it turns out the person clearly means an existing one do you recover that one specifically.
2. **Running a digest** — the person asks for updates against one or more specific vendors, or for "my digest" when they have onboarded before, or they hand over a specific changelog entry / KB number / version. Go to "Running a digest."
3. **Managing the registry or profile** — the person wants to see, edit, or remove tracked vendors, change their interests, or refine what a digest surfaces (for example, reacting to a digest with "I don't need Video" or "stop showing me SDK version updates, I'm pinned to a version"). Go to "Refining the profile."

On a bare invocation with no vendor and no request, do not branch and do not search to find out whether a registry exists. Follow the gate: ask what they want to track or whether they want a digest. Never invent a registry, and never go probing history to discover one uninvited.

## The bundled registry is an example, not a monitoring list

The file at `assets/registry.example.yaml` ships with worked entries (Zoom, Anthropic) purely to document the format and demonstrate the feedless search/scrape case. These are examples, not an active monitoring list. A fresh install monitors nothing until the person onboards their own vendors. Never run a scheduled or on-demand digest against the example entries just because they are present in the bundle: a person who installed this skill did not ask to track Zoom or Anthropic, and surfacing updates for products they may not use is exactly the noise this skill exists to prevent. Treat the person's own onboarded vendors as the only active registry. If the person has onboarded nothing yet and a digest is requested, report that there are no monitored vendors and offer to onboard one, rather than falling back to the examples.

## Refining the profile

The profile is the local filter, and the person sharpens it conversationally over time. When they react to a digest by saying an item or service is not useful to them, that is a standing instruction to stop surfacing it. Record it as an explicit exclusion in the profile per `references/registry-schema.md`, at whichever level they meant:
- A whole service they no longer want: add it to `exclude.services`.
- A component muted within a service they keep: add it to `exclude.components` with the service, what to match, and their reason. The case to support is "I use Twilio Video but I'm bound to a specific SDK version, so I don't need new SDK release notes": Video stays in scope, SDK version updates are muted.

Two rules keep this honest. First, exclusions are always user-initiated. The skill records what the person explicitly asked to mute; it never adds an exclusion on its own judgment that something is minor, because that is the bias the skill exists to avoid. Second, a muted item must never become permanently invisible without the person being able to see it. On request, show the person their active exclusions and let them lift any of them. When unsure whether a passing comment was a real standing instruction, confirm before writing it.

A profile change in this version is held in the conversation, not written to a file. It takes effect for the rest of the session immediately, and it is recovered on a later run by the same past-chat search that recovers the registry, provided the conversation that holds it is not deleted.

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

Then, separately from the standard channel, investigate two more surfaces, because they feed the security and long-duration tables and they are usually distinct from the standard changelog:
- **Security/advisory surface**: does the vendor publish a dedicated security advisory page, security bulletin feed, or consistently security-labeled changelog entries? Find it and confirm it. If the vendor has no standing security surface and only mentions security fixes inline in the general changelog, record that, because it makes the security table best-effort for this vendor: a security item can then only be caught while it is recent enough to appear in the standard channel.
- **Deprecation/end-of-life surface**: does the vendor maintain a standing list of pending deprecations, sunset dates, or end-of-life schedules, the kind of page that shows a 2027 deadline today and keeps showing it until the date passes? Find it and confirm it. If the vendor has no standing deprecation surface and only announces sunsets once in a changelog post, record that, because it makes the long-duration table best-effort: a far-out deadline announced once will fall off once it ages out of the standard window, and the skill has no standing source to re-find it on.

Recording the absence of a surface honestly is as important as recording its presence. It is what lets the digest tell the person which of their tables are trustworthy for a given vendor and which are best-effort, rather than implying coverage the vendor's own publishing cannot support.

### Step 4: Map the vendor's product areas and set the lookback window

Once a source is confirmed, capture the vendor's product-area taxonomy (for Twilio: SMS, Voice, Video, Verify, Lookup, etc.). This is what the interest profile selects from and what entries get classified against. If the feed provides category tags, record that they exist; if not, note that classification will rely on reasoning over title and body.

Then ask how far back the person wants the skill to check for standard updates: one week, thirty days, ninety days, or another span they name. This is the lookback window, a stable per-vendor preference recorded on the vendor's registry entry and applied every run. It governs the standard table only: that table never reaches further back than the window, and never further back than the vendor's source actually provides. The security and long-duration tables are not window-bound; they reflect whatever the vendor is currently surfacing regardless of date. Ask it plainly; do not assume a default without telling the person what you assumed.

### Step 5: Build the registry entry

Only now, with a confirmed standard source, assemble the entry following `references/registry-schema.md`. Record channel type, source identity or URL, extraction rule, product-area taxonomy, classification method (feed categories vs. semantic reasoning), and the security and deprecation surfaces discovered in Step 3 including, explicitly, where one is absent. Then capture the person's interest areas from Step 1 into the profile.

In this chat version the entry is not written to a file or a repo. It is held in the conversation and becomes part of what a later run recovers by past-chat search. Do not instruct the person to save, commit, or upload anything. The schema in `references/registry-schema.md` defines the entry's shape so the in-conversation record is complete and consistent; it is the structure to capture, not a file to write.

### Step 6: Close onboarding and state the persistence limit

Onboarding is complete once the entry is captured and the interest areas recorded. Before moving on, tell the person plainly how this version remembers and what that costs them, so they are not surprised later. Say, in your own words and without softening it: their tracked vendors are not saved to a file; the skill recovers them next time by searching this and other past conversations, so this chat and any chat where they onboard vendors must not be deleted, because deleting it erases the setup and they will have to onboard again from scratch. If they want their setup to persist guaranteed, independent of chat history, the Projects version stores the registry as a Project file and the Cowork version stores it as a working file with scheduled delivery; this chat version trades that durability for needing nothing installed or saved.

Do not offer to schedule recurring digests in this version. Unattended scheduled runs are self-contained and do not reliably carry this conversation's recovered registry, so a scheduled chat-version digest would run against nothing or against a stale embedded copy. Scheduling belongs to the Cowork version, where a working file loads before each run. If the person asks for a morning brief or recurring delivery, tell them that is what the Cowork version is for, and that in this version they run the digest by asking for it.

## Running a digest

A digest runs on demand when the person asks. This chat version does not run on a schedule; recurring delivery is the Cowork version's job. Each run re-derives all three tables live from the vendor's current surfaces, so the result does not depend on anything stored between runs. The only continuity between sessions is recovery by past-chat search: the skill recovers the person's onboarded vendors, their interests, their ignored-security list, and the last-checked watermark from prior conversations at the start of a run. Nothing is held in a file. If the conversations that hold the setup were deleted, recovery finds nothing and the skill onboards fresh.

### Step 1: Load context

This step runs only once there is a vendor in scope, per the invocation gate above. A digest is always against specific vendors: either ones the person just named, or ones recovered because the person asked for "my digest" and a prior onboarding exists. If neither is true, do not run this step and do not search anything; return to the gate and ask what they want to track.

When a vendor is in scope, recover its setup quietly by a targeted past-chat search: search for that vendor's onboarding specifically, not the person's history at large, and reconstruct its entry (channel, sources, surfaces, product areas, lookback window) from what comes back. This is a silent internal step. Do not narrate it, do not list what the search returned, and do not surface unrelated conversations; use the recovered setup and move on. If the targeted search finds no onboarding for that vendor, treat the vendor as new and onboard it, again without reciting the search. Establish the interest profile the same way: use the profile from the current conversation if the person was just interviewed, recover it by the same targeted search if not, and treat the human as the authority on their own needs. If the person handed over a specific identifier (a KB number, a version, a changelog URL), that is the target; fetch it directly.

Set the time bounds, and understand that they apply to the standard-updates table only. This skill outputs three tables (security, standard, long-duration; see Step 4), and only the middle standard-updates table is window-bound. The security and long-duration tables are not time-filtered at all: they reflect whatever the vendor is currently surfacing on its security and deprecation surfaces, regardless of when an item was first announced. So the lookback window and watermark below govern the standard table and nothing else.

Recover the vendor's `lookback_window` from its onboarding by the same targeted, silent past-chat search; this is the floor the standard table never reaches past. Then recover the last-checked watermark the same way: look up the last digest for this specific vendor and treat that as the "since last time" line. None of this recovery is narrated. A normal digest shows standard updates dated after the watermark but within the window. If past-chat search finds no prior digest for this vendor, there is no watermark, so show the full window rather than hiding anything. Honor explicit overrides: "show me everything" ignores the watermark and shows the full window; a stated span like "last 90 days" is a one-off range. Overrides never change the stored window, and overrides apply to the standard table only; the security and long-duration tables always show the vendor's current full set. See `references/registry-schema.md` for how the window and watermark combine.

This three-table model is stateless by design. The skill keeps no item history for deduplication and re-pulls the vendor's current data every run, re-sorting it into the three tables from scratch. It also keeps no setup file: the registry, interests, ignore list, and watermark are all recovered by targeted past-chat search when a digest is requested against a vendor, from the conversations where the person established them. A long-duration deadline shows because it is still listed on the vendor's deprecation surface and drops off once the date passes and the vendor removes it. A security item shows because the vendor is still surfacing it on its advisory surface; note that a vendor advisory commonly stays published after the user has patched, so security items do not reliably disappear on their own, which is exactly why the user can ignore them by name. Nothing is held in a file or a repo; the durable record is the person's own conversation history, which is why deleting the onboarding conversations forces a fresh onboarding.

### Handling an ignore-security command

Ignoring a security item. When the user has resolved a security item on their side and names it ("ignore the AWS-LC item", "ignore bulletin 2026-005"), record it as an ignored security item per `references/registry-schema.md` and drop it from that point on. Most security items carry a clean identifier (a bulletin ID or CVE) to key the ignore on; a best-effort item that surfaced only through a general changelog may not, in which case key the ignore on a distinctive title fragment instead, per the schema. The ignore takes effect immediately for the rest of this session, and on later runs it is recovered by the same past-chat search that recovers the registry, so it holds as long as the conversation that records it is not deleted. The user can ask what security items they are currently ignoring and lift any of them at any time. Ignore is scoped to the single named item: it does not stop tracking the vendor, does not disable the security table, and does not affect any other item; new and different advisories still surface normally.

### Step 2: Fetch updates from each vendor's surfaces

Each vendor has up to three surfaces, recorded in the registry: the standard changelog or release channel (RSS, email, GitHub releases, or search/scrape fallback), the security or advisory surface, and the standing deprecation or end-of-life surface. Pull all that the vendor maintains. For the standard channel: fetch and parse the RSS/Atom feed; or read the vendor notice from the connected inbox and extract the identifier; or run the search/scrape fallback for feedless vendors. For the security surface: pull the vendor's current security advisories or security-labeled changelog entries. For the deprecation surface: pull the vendor's standing list of pending end-of-life and deprecation deadlines. Always fetch live. Investigate per the core operating rule if a recorded source has gone stale.

Not every vendor maintains all three surfaces. Where a vendor has no standing security surface or no standing deprecation surface, that was recorded at onboarding, and the corresponding table is best-effort: it can only catch a security or deadline item while that item still appears in the standard channel within the window, after which the skill has no standing source to re-find it on. This is a limitation of the vendor's publishing, not of the skill, and the digest should not pretend otherwise.

Keep only dated change records for the standard table. An item counts toward the standard table only if it is an actual change with a release or publication date in the cycle: a changelog entry, a release note, a dated announcement. Discard reference material even when it looks relevant: API documentation, guides, how-to pages, and behavior descriptions are not updates, and surfacing them as if they were is inventing signal where there was none this cycle. This matters most for the search/scrape channel, which can return docs and guides alongside changelog entries. The security and long-duration tables are populated from their own surfaces rather than from dated cycle changes, so this docs-versus-changes rule governs the standard table specifically.

### Step 3: Read everything, filter by service scope, route by type

Read the full set of items pulled from all surfaces. Keep every item that concerns a service the person monitors. The filter is about scope, not importance: it decides whether an item belongs to a service the person tracks, not whether the skill judges it significant enough to mention. Once a service is in scope, every item to it is included; assessing which ones matter is the person's call when they read the tables, not the skill's when it builds them.

Judge scope by what the item actually concerns, not by keyword presence: an item that never names the person's keyword but changes a service they monitor is in scope; an item that mentions their area in passing but actually concerns a different service is not. When in doubt about scope, keep the item and let it appear rather than silently dropping it. Never drop an in-scope item because it seems minor.

Then route each in-scope item to one of three tables by what it is, reading the item's own labeling rather than inferring:
- **Security (top table)**: items the vendor labels as security in the header or publishes to its security/advisory surface. These appear first because they are what an engineer scans for before anything else.
- **Long-duration (bottom table)**: future-dated actionable deadlines, the kind where the vendor is removing or breaking something you depend on by a stated date (deprecations, end-of-life, forced migrations, sunset dates). Routed here by the obligation-direction in the item's own language: "will be deprecated", "will be removed", "must migrate by", "no longer supported after". Additive and roadmap items that merely carry a future date ("coming in 2027", "now available", "planned") are NOT deadlines and never go here; they are standard items.
- **Standard (middle table)**: everything else, the normal update feed, governed by the window and watermark.

Then apply the person's standing exclusions and ignored security items from the profile. A service in `exclude.services` is suppressed from all three tables; a component matching an `exclude.components` rule is suppressed within its still-in-scope service (the "Video but not the new SDK" case); an item in the ignored-security list is suppressed from the security table by its identifier. Exclusions and ignores are the person's own choices, so honoring them is not the skill making an importance judgment. Still evaluate excluded items against the surfaces each cycle rather than skipping them blind, so that if the person later lifts a mute or ignore, nothing was missed, and so you can answer if they ask what their exclusions or ignored security items are currently hiding.

### Step 4: Organize and output

Render three tables in fixed order: security at the top, standard in the middle, long-duration at the bottom. Each is a scannable table, provider and service in column one and the critical detail in two to three sentences in column two. Show a table's header even when it is empty so the person can see the skill checked and found nothing rather than wondering whether it looked: for the standard table an empty service row reads "no updates available"; for the security and long-duration tables an empty table reads "none current". The standard table follows the window and watermark; the security and long-duration tables show the vendor's current full set regardless of date. For the exact table rules and the optional engineering HTML publication mode, read `references/output-formats.md`. Use the three-table layout unless the person explicitly asks to publish or format for engineering.

After the last table, before asking the person anything or adding any other content, print the disclaimer line exactly: "Not liable for inaccurate vendor data or missed items; verify critical items at the vendor source. @ungatedbluemarble." This line appears on every digest without exception.

### Step 5: Offer calendar events for long-duration deadlines

Because this version stores nothing, a far-out deadline surfaced in the long-duration table has no standing place to live between now and the date. The stopgap is to offload it onto the person's calendar, which does persist, at the moment it surfaces. This is the one persistence affordance the chat version offers, and it is offered only when there is a real deadline to act on.

When the long-duration table contains one or more dated deadlines, after the disclaimer, offer to create calendar reminders for them. The calendar is the person's choice, not the skill's: ask which calendar or email service they want the reminders in (Google Calendar, Outlook, Apple Calendar, or whatever they use) rather than assuming one. Then check whether that service connects to Claude natively. If it does and is already connected, proceed. If it is compatible but not yet connected, guide them to connect it before creating events, in plain steps for their named service. If the service they name does not connect to Claude natively, tell them so directly and do not pretend a path exists: name the limitation, and offer the alternatives that do work (a compatible connector they may also have, or creating the events in a connected calendar and syncing from there). Never create events autonomously; this version acts only on the person's explicit yes in the moment, into a connector they chose and confirmed.

For each deadline the person accepts, create reminder events ahead of the date on a 90, 60, and 30 day schedule. When the deadline is six months or more out, create all three. When it is closer, create only the lead times that still fit: under six months might be 60 and 30, under about two months just 30, and for a deadline closer than the shortest lead time, offer a single reminder at a sensible point before it. Title each event so it is self-explanatory months later: the vendor, the service, what is being deprecated or removed, and the hard date. Put the source URL in the event body so the person can re-verify at the vendor when the reminder fires. Confirm back what was created.

This is a per-digest action, not a standing arrangement. The events, once created, live in the person's calendar independent of any chat history, which is the point: even if the conversation is later deleted, the deadline reminders survive. The skill itself still remembers nothing.

## Writing rules (all output)

Prose, not bullet lists, unless the person asks otherwise. No em dashes; use a colon or plain connective language. No emojis or icons. Lead with the date and the most recent confirmed version where relevant. Flag operationally significant items: deprecations with deadlines, breaking changes, anything requiring explicit enablement, anything affecting the person's specific components. Close with the source URL(s) used so the reader can drill in. Never reproduce large verbatim blocks from a vendor's notes; summarize in your own words and link out.

## Notes and known limitations

- Some vendors abandon or under-maintain their RSS feeds. Validate every feed by fetching it; do not trust that a conventional path exists.
- JavaScript-rendered changelog pages (Zoom's support KB, some developer changelog indexes) return blank on direct fetch. Use the search/scrape fallback documented in the references.
- This version is stateless. It stores no registry file, no profile file, nothing in a repo, and nothing in the person's account instructions. Setup persists only through the person's own conversation history, recovered by past-chat search at the start of each run. Deleting the conversations where vendors were onboarded erases the setup and forces a fresh onboarding. There is no backup. The person who needs guaranteed persistence should use the Projects version (registry as a Project file) or the Cowork version (working file plus scheduled delivery).
- This version does not run on a schedule. Recurring or unattended digests are the Cowork version's job, because scheduled runs are self-contained and do not reliably carry this version's recovered setup. Here the person runs a digest by asking for one.
- The skill keeps no item history for deduplication and re-derives all three tables from the vendor's current surfaces every run. Continuity between runs (which vendors, which interests, which ignores, the watermark) comes from past-chat recovery, not from stored state.
- The only persistence affordance is the calendar-event offer for long-duration deadlines: with the person's agreement and a connected calendar, the skill creates 90/60/30 day reminders so a far-out deadline survives on the calendar even though the skill remembers nothing.
- The security and long-duration tables are only as complete as the vendor's published surfaces. A vendor with a proper security advisory page and a standing deprecation list gives reliable top and bottom tables. A vendor that only mentions security fixes or sunset dates once in a general changelog post gives best-effort tables, because once such an item ages out of the standard window there is no standing source to re-find it on. This limitation is recorded per vendor at onboarding and is a property of the vendor's publishing, not the skill.
- The registry ships with two complete example entries (Zoom and Anthropic) so the format is self-documenting and the feedless search/scrape case is demonstrated. They are examples only, never an active monitoring list; everything real is added through onboarding.
