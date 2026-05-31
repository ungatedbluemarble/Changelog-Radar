# Changelog Radar

A Claude skill that tells you what changed in the software you use, filtered down to only the services you care about, in a plain table you can scan in seconds.

**The proposition:** Changelog Radar exists to give proactive teams early recognition of change in their environment, for the specific applications they are responsible for. Whoever owns an integration, a support queue, or an application relationship needs to know about the vendor changes that affect it before those changes become incidents, surprise tickets, or missed deprecation deadlines. This skill turns that from a chore nobody gets around to into something that surfaces on its own, scoped to exactly what each person owns.

Email digests and RSS feeds have the same problem: they deliver everything on the vendor's schedule and leave you to dig out what matters. Changelog Radar flips that. It interviews you about what you actually build and depend on, then reads a vendor's full changelog and shows you only the updates that touch your services, one row each. Run it on a schedule (see below) and that recognition arrives without anyone having to remember to look.

This is offered freely and open source, built from career experience about which tools actually earn their place on a team. It is not a product for sale.

---

## Which are you?

**I just want to track my tools' updates.** You do not need this repository. You do not need to clone anything or understand any of the files here. Install the skill once (see below), then just talk to Claude: tell it what product you use and what you do with it, and it does the rest. The repository exists only so the skill can be shared and inspected. Skip to [Just using it](#just-using-it).

**I want to install, inspect, or contribute to the skill.** You are in the right place. See [Installing](#installing) and [How it works](#how-it-works).

---

## Just using it

Once the skill is installed in your Claude, you never touch this repository again. The whole experience is conversational:

- **Start tracking a vendor.** Say something like "I use Twilio for SMS and want to track their changelog." Claude interviews you about what you build, finds and confirms where the vendor publishes updates, and remembers it.
- **Get a digest.** Ask "what changed in Twilio?" or "give me my updates." Claude pulls the latest changes, keeps only the ones touching services you monitor, and shows you a table: each row is one update, with the provider and service on the left and the critical detail on the right.
- **Refine over time.** React to a digest. "I don't need Video." "Stop showing me SDK version updates, I'm pinned to a version." Claude records that and future digests respect it. Ask "what am I currently muting?" any time, and lift a mute whenever you want.

That is the whole loop. You are the expert on what you use; the skill's job is to ask good questions, fetch reliably, and never bury the things you said you care about.

### Recommended: run it as a scheduled task for a morning summary

The most valuable way to use this skill is to not have to remember it at all. Set it up as a scheduled task in Claude so a filtered summary of changes to the software you are responsible for arrives on its own, for example each morning, rather than waiting for you to think to ask.

This matters because changelog awareness is a prevention job, and prevention is the thing busy people never get around to. An engineer, a support lead, or an application owner will not check vendor changelogs proactively day to day; they go looking only after something has already broken or a ticket has already landed. A scheduled summary flips that: the relevant change is in front of you before it becomes an incident, with no initiative required on your part.

To set it up, ask Claude to schedule a recurring task that runs your Changelog Radar digest for the vendors you track and delivers the table on your chosen cadence. One honest limitation: a scheduled task runs while your Claude app is open and your machine is awake, so a "9am summary" lands when those conditions are met rather than firing on a fixed clock the way a server cron would. For a daily morning check that is usually fine; for guaranteed unattended delivery you would need a hosted runner, which is outside what this skill does.

---

## Installing

The skill is a single file, `changelog-radar.skill`, in this repository. To install it in Claude:

1. Download `changelog-radar.skill`.
2. In Claude, go to your skills settings and add the skill file. (In claude.ai: Settings, then Capabilities or Skills, depending on your plan. Code execution must be enabled for skills to run.)
3. That is it. The skill triggers on its own when you ask about tracking or digesting vendor updates.

If you prefer to inspect before installing, the unpacked skill is in the [`changelog-radar/`](changelog-radar/) folder. A `.skill` file is just a zip of that folder. Reading the contents before enabling a skill from anyone, including a colleague, is good practice.

---

## How it works

The skill keeps two kinds of state, both as plain files you control. Neither is a live database the skill reaches into while running; they exist for portability and inspection.

- **The registry** (`changelog-radar/assets/registry.example.yaml` is the seed) records, per vendor, where its updates come from and what its product areas are. It is shared and reusable. It ships with one real worked entry, Zoom, which demonstrates the hardest case: a vendor with no usable feed, reached by search and scrape instead.
- **Your profile** records which services you track and anything you have chosen to mute. It is normally drawn out of you in conversation each time, because you are the source of truth about your own needs. You can keep it as a file for convenience, but the skill never depends on a stored copy.

When the skill adds a vendor or records a mute, it hands you the updated file. Committing it back is your one manual step. This is deliberate: the skill cannot and does not push to your repository, which keeps your registry version-controlled and inspectable rather than silently mutated.

Two rules sit at the core of the design:

- **It validates before it records.** A vendor is never added until the skill has confirmed a working source by fetching and parsing it, or by you supplying the feed URL when discovery cannot. It does not guess URLs or fabricate sources. If it cannot confirm one, it tells you where the vendor's feed or subscription setting lives so you can supply it.
- **It does not decide what matters to you.** The only filter the skill applies on its own is scope: is this update about a service you monitor. Within a service you track, every update appears. Dropping updates because the skill judged them minor is exactly the bias it avoids. The only things suppressed are the ones you explicitly asked to mute, and those are always visible and reversible on request.

---

## Output

Every digest is a single table. Two columns: provider and service on the left, the update in two or three sentences on the right. One row per update. A service you monitor that had nothing this cycle shows a row reading "no updates available," so you can always tell the difference between "nothing changed" and "nothing was checked."

An optional engineering-publication mode renders a structured HTML page instead, for when you want to share a developer-focused digest with a team. Ask for it explicitly; the table is the default.

---

## Ingestion channels

The skill supports several ways to receive a vendor's updates, and it asks you how you receive them today and how you want to going forward, rather than choosing for you:

- **RSS or Atom feed**, the cleanest source when a vendor offers one.
- **Email notice**, when you would rather updates arrive in a connected inbox; the vendor's own notification becomes the trigger.
- **GitHub releases**, for tools hosted on GitHub, via their releases feed.
- **Search and scrape**, the fallback for vendors with no usable feed, such as Zoom.

---

## Contributing

If you validate a feed or source for a vendor not yet in the seed registry, a registry entry contributed back helps everyone. Keep entries to confirmed, fetched, parsing sources only; the validation bar is the point of the project.

---

## What this is not

It is not a hosted service, a background daemon, or an MCP server. It is a skill: instructions Claude follows, using tools it already has. It does not run unattended on its own; if you want scheduled delivery, pair it with a Claude scheduled task. It does not share your filtered results publicly; the skill is the shareable thing, your digests are yours.
