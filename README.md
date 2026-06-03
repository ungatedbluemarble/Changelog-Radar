# Changelog Radar

A Claude skill that puts vendor change awareness directly in your hands. It learns the software you actually build on, watches each vendor for changes, and shows you only what touches your services, sorted by how urgently it matters: security now, routine updates, and deadlines coming down the line.

This is the **standard chat version**. It is stateless: it stores nothing, installs nothing beyond the skill, and asks you to save nothing. You install it, talk to it, and it works.

**The problem it solves.** When a vendor deprecates an API, ends support for a runtime, or ships a security fix, that information is public. It is also scattered. Security bulletins go to a security inbox. Vendor account emails go to whoever signed the contract. Deprecation notices sit on a docs page nobody has open. Runtime monitoring tells you what is happening now, not what a vendor announced will break in ninety days. The signal lands in a dozen different places, and rarely in front of the engineer who will actually have to deal with it. By the time it reaches the person who owns the dependency, the deadline is often closer than it should be, or already past.

Changelog Radar closes that gap. The engineer with their hands on the systems, who knows exactly which dependencies matter, gets the relevant vendor change directly, with lead time to plan and raise the flag early, rather than finding out when something breaks. You do not need a centralized changelog process, a security team that forwards alerts, or anyone else to loop you in. You watch your own stack, on your own terms.

It is offered freely and open source. It is not a product for sale.

---

## Disclaimer

Changelog Radar reads and presents information published by third-party vendors. It does not generate that information and cannot guarantee its accuracy, completeness, or timeliness. Every result depends on what each vendor publishes and maintains on its own changelog, security, and deprecation pages.

This skill and its author, @ungatedbluemarble, accept no responsibility or liability for missed security patches, missed deprecation or end-of-life deadlines, delayed notices, or any consequence arising from a vendor publishing incomplete, inaccurate, outdated, or altered information, or from a vendor not maintaining a standing source for it. The skill surfaces what a vendor makes available at the time of each check and nothing more.

Always treat the vendor's official page as authoritative. Use the source links in every digest to verify anything critical directly with the vendor before acting on it. Do not rely on this skill as your sole or system-of-record source for security, compliance, or deadline-critical decisions. The skill is provided as is, without warranty of any kind. See the LICENSE file for full terms.

---

## How this version remembers (and what that costs you)

This version is stateless by design. It writes no file, commits nothing to any repository, and saves nothing to your account. That is what lets you install it and use it with zero setup beyond the skill itself.

Continuity between sessions comes from one mechanism: when you ask for a digest on a vendor, the skill quietly searches your own past conversations to recover how you set that vendor up. If it finds your earlier onboarding, the digest proceeds against it. If it does not, it onboards the vendor fresh.

This has one hard consequence, stated plainly because you should know it before you rely on the skill: the only record of your tracked vendors lives in the conversations where you set them up. If you delete those conversations, the setup is gone. There is no backup file and no stored copy. You would have to onboard again from scratch.

If you want your setup to persist guaranteed, independent of your chat history, that is exactly what the Projects and Cowork versions of this skill are for: they store the registry as a real file. This chat version trades that durability for needing nothing installed or saved.

---

## Just using it

Once the skill is installed, the whole experience is conversational. You never touch a file or this repository.

- **Start tracking a vendor.** Say something like "I use Twilio for SMS and want to track their changelog." The skill interviews you about what you build, finds and confirms where the vendor publishes its updates, security advisories, and deprecation notices, and records it for the session.
- **Get a digest.** Ask "what changed in Twilio?" or "give me my updates." The skill pulls each vendor live, keeps only what touches services you monitor, and shows you three tables (see [Output](#output)).
- **Refine over time.** React to a digest. "I don't need Video." "Stop showing me SDK version updates, I'm pinned to a version." The skill records it for the session and future digests respect it. Ask "what am I currently muting?" any time, and lift a mute whenever you want.

You are the expert on what you use. The skill's job is to ask good questions, fetch reliably, and never bury the things you said you care about.

### Running a digest

This version runs on demand, when you ask. It does not run on a schedule; recurring or automatic morning digests are what the Cowork version is for. Here, you get a digest by asking for one.

---

## Output

Every digest is three tables, in order of how urgently each item needs your attention:

- **Security** (top): items the vendor labels as security or publishes to its advisory page. These appear first because they are what you scan for before anything else. A vendor's advisory usually stays published long after you have patched, so once you have resolved an item on your side, name it and tell the skill to ignore it, for example "ignore the AWS-LC item" or "ignore bulletin 2026-005." The skill drops that item and does not bring it up again. It removes that one named item only: it does not stop tracking the vendor, does not turn off the security table, and does not affect any other item. Ask "what security items am I ignoring?" any time, and lift any of them.
- **Standard** (middle): the normal update feed for services you track, scoped to the last 30 days (or whatever lookback you set per vendor) and to what is new since your last check.
- **Long-duration deadlines** (bottom): future-dated changes that will break something you depend on, like API deprecations, runtime end-of-life, and forced migrations, drawn from the vendor's standing deprecation page. Each shows from the day it is announced until its date passes, so a deadline a year out is in front of you with a year of lead time, not discovered the week it lands.

Each table shows its header even when empty, so you can always tell the difference between "nothing changed" and "nothing was checked." Every item links to the vendor source it came from. The accuracy and completeness of the security and deadline tables depend on the vendor maintaining a standing page for each; where a vendor only mentions these once in a general changelog, the skill tells you at onboarding that those tables are best-effort for that vendor.

An optional engineering-publication mode renders a structured HTML page instead, for sharing a developer-focused digest with a team. Ask for it explicitly; the tables are the default.

### Calendar reminders for deadlines

Because this version stores nothing, a far-out deadline has no place to live between now and its date. So when a digest surfaces a long-duration deadline, the skill offers to put reminders on your calendar, which does persist. With your agreement and a connected calendar, it creates reminders ahead of the date (90, 60, and 30 days out for deadlines six months or more away; the lead times that still fit for closer ones). The events live in your calendar independent of any chat history, so even if you later delete the conversation, the deadline reminders survive. The skill never creates events on its own; it acts only on your explicit yes, into a calendar you choose and confirm.

---

## Installing

The skill is a single file, `changelog-radar.skill`, in this repository. To install it in Claude:

1. Download `changelog-radar.skill`.
2. In Claude, go to your skills settings and add the skill file. (Code execution must be enabled for skills to run.)
3. That is it. The skill triggers on its own when you ask about tracking or digesting vendor updates.

If you prefer to inspect before installing, the unpacked skill is in the [`changelog-radar/`](changelog-radar/) folder. A `.skill` file is just a zip of that folder. Reading the contents before enabling a skill from anyone is good practice.

---

## How it works

The skill holds two things in the conversation, never in a file: a registry of the vendors you track (where each one publishes its updates, security advisories, and deprecation notices) and a profile of which services matter to you and anything you have muted or ignored. On a later run it recovers them by searching your past conversations.

Two rules sit at the core of the design:

- **It validates before it records.** A vendor is never added until the skill has confirmed a working source by fetching it, or by you supplying the URL when discovery cannot. It does not guess URLs or invent sources. If it cannot confirm one, it tells you where the vendor's feed or subscription setting lives so you can supply it.
- **It does not decide what matters to you.** The only filter the skill applies on its own is scope: is this about a service you monitor. Within a service you track, every item appears. Dropping items because the skill judged them minor is exactly the bias it avoids. The only things suppressed are the ones you explicitly muted or ignored, and those are always visible and reversible on request.

The skill re-reads each vendor's current pages on every run and rebuilds the tables from scratch. A deadline shows because it is still listed on the vendor's deprecation page and drops off once the date passes. A security item shows because the vendor is still surfacing it; since advisories often stay published after you have patched, you clear those by ignoring them by name.

---

## Ingestion channels

The skill asks how you receive a vendor's updates today and how you want to going forward, rather than choosing for you:

- **RSS or Atom feed**, the cleanest source when a vendor offers one.
- **Email notice**, when you would rather updates arrive in a connected inbox.
- **GitHub releases**, for tools hosted on GitHub.
- **Search and scrape**, the fallback for vendors with no usable feed (such as JavaScript-rendered changelog pages).

Alongside the standard channel, onboarding also locates each vendor's security advisory page and standing deprecation page where they exist, since those feed the security and long-duration tables.

---

## What this is not

It is not a hosted service, a background daemon, or an MCP server. It is a skill: instructions Claude follows, using tools it already has. It does not run on a schedule and does not share your results anywhere. The skill is the shareable thing; your digests are yours.

The bundled `changelog-radar/assets/registry.example.yaml` is an example only, included to document the format. The skill never monitors those example vendors; it tracks only what you onboard.
