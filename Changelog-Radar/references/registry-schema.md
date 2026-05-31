# Registry and profile schema

Two files. Both live in the person's repo for distribution and inspection. The skill reads them and writes updated versions for the person to commit; the skill does not push.

- `registry.yaml` — the person's own working registry: the vendors they have onboarded and actively monitor. This starts empty on a fresh install and is built up through onboarding. It is distinct from the bundled `assets/registry.example.yaml`, which is an inert example for documenting the format only. Never write the person's real onboarded vendors into the example file, and never treat the example file as the active monitoring list. The example demonstrates structure; the working registry is what the person builds.
- `profile.yaml` — per person. Which vendors they track and which areas/components matter. Optional: normally elicited fresh in conversation. Saved only as a convenience.

YAML is shown here for readability. JSON with the same structure is equally valid; match whatever the person's repo already uses.

## registry.yaml

Top-level key `vendors`, a list. Each vendor entry:

```yaml
vendors:
  - id: zoom                      # short, lowercase, no spaces. used as the key.
    name: Zoom                    # display name
    channel: search_scrape        # one of: rss | email | github_releases | search_scrape
    classification: semantic      # one of: feed_categories | semantic
    # --- channel-specific source block: include only the one matching `channel` ---
    rss:
      feed_url:                   # validated, fetched, parses to dated entries
    github_releases:
      repo:                       # owner/repo
    email:
      sender:                     # identity to match (e.g. no-reply@vendor.com or display name)
      subject_pattern:            # how to recognize a release notice vs other mail
      identifier_rule:            # how to extract the version/KB from the message
    search_scrape:
      queries:                    # list of search queries that surface dated content
      fetch_targets:              # URL patterns for individual entries that render on fetch
    # --- product taxonomy: what the profile selects from ---
    product_areas:
      - SMS
      - Voice
      # ...
    lookback_window: 30d           # how far back digests reach: 7d | 30d | 90d | <n>d. set at onboarding.
    notes:                        # freeform: known quirks, deprecation deadlines to flag, etc.
```

Rules:
- `id` must be lowercase with no spaces or capitals. Spaces and capitals in name-like fields break packaging and lookups; keep `id` clean and put the pretty name in `name`.
- Include exactly one channel-specific block, matching `channel`.
- Never write an entry whose source was not validated by fetching (feeds) or confirming receipt (email) or confirming search reach (search_scrape).
- `classification: feed_categories` only if the feed actually carries category tags; otherwise `semantic`.
- `lookback_window` is the stable per-vendor floor: a digest never reaches further back than this, and never further than the source provides. Set it from the onboarding question. The person can override it for a single run by asking ("show me the last 90 days"), which does not change the stored window.

## profile.yaml

Top-level key `interests`, keyed by vendor `id`:

```yaml
interests:
  zoom:
    areas:                        # subset of the vendor's product_areas the person cares about
      - Phone
    components:                   # finer-grained things the person actually builds/depends on
      - call log APIs
      - phone number provisioning
    exclude:                      # standing user-set mutes; never skill-inferred. two levels:
      services:                   # whole services the person does not want surfaced
        - Video
      components:                 # components muted within a service that stays in scope
        - service: Phone
          match: SDK version updates
          reason: pinned to a specific SDK version, do not need new SDK release notes
    notes:                        # context that sharpens scope judgments
      - "integrates Zoom Phone with Twilio; flag anything touching that boundary"
```

The `components` under interests are what the person tracks. The `exclude` block is what they have explicitly told the skill to stop surfacing, and it has two levels: whole `services` they do not want at all, and `components` muted within a service they otherwise keep (the Video-but-not-the-new-SDK case). Both are always user-initiated standing instructions. The skill records them verbatim from what the person said; it never adds an exclusion on its own judgment that something is minor. Each component exclusion carries the `service` it applies within, a `match` describing what to suppress, and the `reason` in the person's words so the mute stays auditable.

## The last-checked watermark

To avoid showing the same updates run after run, the skill needs to know when it last produced a digest for a vendor. This watermark is deliberately NOT stored in a file or in the skill, because that would force the person to carry state between sessions or reinstall the skill on every run. Instead, at the start of a digest the skill recovers the watermark from the conversation history using past-chat search: look up the last time a digest ran for this vendor, and treat that moment as the "since last time" line.

How it combines with the lookback window: the window is the floor (never reach further back than it), the watermark is the "new since you last looked" line within that window. On a normal run, show updates dated after the watermark but within the window. If past-chat search finds no prior digest for this vendor, there is no watermark, so fall back to the full lookback window and show everything in it. That is the safe direction: when unsure, show more, never hide.

Overrides are explicit and the person's call: "show me everything" ignores the watermark and shows the full window; "only since yesterday" or "last 90 days" sets a one-off range. Overrides never change the stored lookback window.

This watermark is personal to the person's own conversation history, so it does not travel to someone else who installs the skill. That is correct: each person's own history serves their own watermark.

## The seed entry

`registry.yaml` ships with one complete, real entry, Zoom, in `assets/registry.example.yaml`. It demonstrates the `search_scrape` channel (the hard, feedless case) and `semantic` classification, so the format is self-documenting and the toughest path is shown working. Every other vendor is added through onboarding.
