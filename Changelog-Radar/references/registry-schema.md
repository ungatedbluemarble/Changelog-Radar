# Registry and profile schema

Two structures: the registry and the profile. In this chat version neither is a file. Both are captured in conversation during onboarding and recovered on later runs by past-chat search. The shapes below define what to capture so the in-conversation record is complete and consistent, and so a recovery run knows what fields to look for. YAML is shown for readability; the structure is what matters, not the format.

- `registry` — the person's onboarded vendors: how to reach each vendor's updates and what its product areas are. Built up through onboarding, recovered by past-chat search, never stored to a file.
- `profile` — per person: which vendors they track, which areas and components matter, what they have excluded, and which security items they have ignored. Normally elicited in conversation and recovered the same way.

The bundled `assets/registry.example.yaml` is an inert example that documents this format and demonstrates the feedless search/scrape case. It is never an active monitoring list and the skill never treats it as one.

## registry structure

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
    # --- additional surfaces feeding the security and long-duration tables ---
    security_surface:             # where the vendor publishes security advisories
      url:                        # advisory page or security-labeled feed, or null if none
      present: true               # false if the vendor has no standing security surface
                                  # (security table is then best-effort for this vendor)
    deprecation_surface:          # the vendor's standing list of pending deadlines/EOL
      url:                        # deprecation/end-of-life page, or null if none
      present: true               # false if the vendor has no standing deprecation surface
                                  # (long-duration table is then best-effort for this vendor)
    # --- product taxonomy: what the profile selects from ---
    product_areas:
      - SMS
      - Voice
      # ...
    lookback_window: 30d           # applies to the STANDARD table only: 7d | 30d | 90d | <n>d.
    notes:                        # freeform: known quirks, deprecation deadlines to flag, etc.
```

Rules:
- `id` must be lowercase with no spaces or capitals. Spaces and capitals in name-like fields break packaging and lookups; keep `id` clean and put the pretty name in `name`.
- Include exactly one channel-specific block, matching `channel`.
- Never write an entry whose source was not validated by fetching (feeds) or confirming receipt (email) or confirming search reach (search_scrape).
- `classification: feed_categories` only if the feed actually carries category tags; otherwise `semantic`.
- `lookback_window` is the stable per-vendor floor for the standard table only: the standard table never reaches further back than this, and never further than the source provides. The security and long-duration tables are not window-bound; they reflect the vendor's current security and deprecation surfaces regardless of date. Set it from the onboarding question. The person can override it for a single run by asking ("show me the last 90 days"), which affects only the standard table and does not change the stored window.
- `security_surface` and `deprecation_surface` record whether the vendor maintains a standing source for each. Set `present: false` honestly when one does not exist; that flag is what tells the digest the corresponding table is best-effort for this vendor rather than reliable.

## profile structure

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
    ignored_security:             # security items the person resolved and told the skill to ignore by name
      - id: 2026-005-AWS          # the vendor's own identifier for the item (bulletin ID, CVE, advisory slug)
        match: AWS-LC             # short human label so the person recognizes it when listing ignores
        ignored_on: 2026-06-02    # date the person ignored it, for auditability
    notes:                        # context that sharpens scope judgments
      - "integrates Zoom Phone with Twilio; flag anything touching that boundary"
```

The `components` under interests are what the person tracks. The `exclude` block is what they have explicitly told the skill to stop surfacing, and it has two levels: whole `services` they do not want at all, and `components` muted within a service they otherwise keep (the Video-but-not-the-new-SDK case). Both are always user-initiated standing instructions. The skill records them verbatim from what the person said; it never adds an exclusion on its own judgment that something is minor. Each component exclusion carries the `service` it applies within, a `match` describing what to suppress, and the `reason` in the person's words so the mute stays auditable.

The `ignored_security` block holds security items the person has resolved on their side and told the skill to ignore by name. Each entry records the vendor's own `id` for the item (the bulletin ID, CVE, or advisory slug the skill matches against), a short `match` label so the person recognizes it when they ask what they are ignoring, and the `ignored_on` date for auditability. Formal advisories carry a clean identifier; a best-effort security item that surfaced only through a general changelog may not, in which case set `id` to null and rely on `match` (a distinctive title fragment or the CVE if one is named) to suppress it. An ignore is always user-initiated and scoped to the single named item: the skill suppresses that item from the security table by its `id` when present and otherwise by `match`, and never expands an ignore to the whole vendor or the whole table. Like exclusions, ignores are visible and reversible on request. In this chat version the ignore is held in conversation and recovered on later runs by past-chat search, the same as the rest of the setup, so it holds as long as the conversation that records it is not deleted.

## The last-checked watermark (standard table only)

The watermark governs the standard (middle) table only. The security and long-duration tables are not deduplicated by a watermark: they show the vendor's current full set every run, because their value is in always reflecting what is currently pending, not in showing only what changed since last time. What follows applies to the standard table.

To avoid showing the same standard updates run after run, the skill needs to know when it last produced a digest for a vendor. This watermark is deliberately NOT stored in a file or in the skill, because that would force the person to carry state between sessions or reinstall the skill on every run. Instead, at the start of a digest the skill recovers the watermark from the conversation history using past-chat search: look up the last time a digest ran for this vendor, and treat that moment as the "since last time" line.

How it combines with the lookback window: the window is the floor (never reach further back than it), the watermark is the "new since you last looked" line within that window. On a normal run, show updates dated after the watermark but within the window. If past-chat search finds no prior digest for this vendor, there is no watermark, so fall back to the full lookback window and show everything in it. That is the safe direction: when unsure, show more, never hide.

Overrides are explicit and the person's call: "show me everything" ignores the watermark and shows the full window; "only since yesterday" or "last 90 days" sets a one-off range. Overrides never change the stored lookback window.

This watermark is personal to the person's own conversation history, so it does not travel to someone else who installs the skill. That is correct: each person's own history serves their own watermark.

## The example entries

`assets/registry.example.yaml` ships with two complete, real entries, Zoom and Anthropic. Both demonstrate the `search_scrape` channel (the hard, feedless case) and `semantic` classification, and they show the `security_surface` and `deprecation_surface` fields populated and, where a vendor lacks a standing surface, marked absent. The format is self-documenting and the toughest path is shown working. The file is an inert example only, never an active monitoring list; every real vendor is added through onboarding and recovered by past-chat search, never written into this example file or any other file.
