# Repo and distribution

What the repo is, what it is not, and how to talk about it to two very different people.

## The repo is for distribution, not runtime

The skill runs in Claude. The interview, fetching, filtering, and formatting all happen there. The repo does not get reached into while the skill runs. The repo exists so the skill can be handed to other people, so the validated registry is public and inspectable, and so the whole thing reads as a credible showcase. It is the box the skill ships in, not a part the machine calls.

Concretely the repo holds: the skill itself (SKILL.md and its references and assets), a seed `registry.yaml` with the one worked Zoom example, the schema documentation, an illustrative `profile.yaml`, and a README. Once a person installs the skill, they live in Claude; they touch the repo again only if they choose to share an updated registry back.

## Two audiences, routed immediately

The README must split the reader on the first screen, before either hits instructions meant for the other.

**The hands-on person (engineer or comfortable with git):** clone or download, install the skill into their Claude environment, done. When onboarding adds a vendor, the skill hands them updated registry content and they commit it. Short list, assumes git literacy.

**The person who just wants to watch their tools' updates:** tell them plainly they do not need the repo at all for daily use. They talk to the skill, it interviews them, it does the work. The repo exists for transparency and sharing. The worst outcome is this person believing they must understand a repo to track their updates and bouncing. Give them explicit permission not to clone.

## The skill cannot push

When onboarding changes the registry, the skill outputs the updated file and hands it back. Committing is the person's one manual step. State this without apology: it is a deliberate consequence of the repo being a distribution vehicle rather than a live datastore, and it keeps the registry version-controlled and inspectable.

## README shape

Lead with what this is and a one-line statement of the problem it solves (email and RSS both fail at relevance: they deliver everything on the vendor's schedule, leaving the reader to find what matters). Then immediately route the two audiences. Then, for the hands-on path only, give install and contribute steps. Keep the non-technical path to a few sentences that end in "just talk to the skill." A short contributor note for anyone wanting to submit a verified vendor entry back fits the public-showcase intent but is not required for launch.
