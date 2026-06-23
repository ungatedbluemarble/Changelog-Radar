# Changelog Radar

A Claude skill that interviews you about the products and services you actually use, fetches each vendor's release notes live, and returns only the changes relevant to your stack, explained in plain language.

## What makes it different

Most changelog tools dump an entire vendor feed at the reader, or match crude keywords. Changelog Radar does something keyword matching cannot: it interviews you as the expert on your own stack, learns what you actually depend on, and reads a vendor's full changelog before returning only the items that apply to you. You are the source of truth about your own needs. The skill's job is to interview well, source updates reliably, and filter by genuine relevance.

## Default output

Every digest renders as an inline chat widget by default. The widget shows three sections in fixed order: security items (shown until resolved at source), standard updates (governed by your lookback window), and long-duration deadlines (deprecations and end-of-life dates, shown until the date passes). Each row shows the vendor and service on the left and the actionable detail on the right, with a direct link to the source.

![Changelog Radar digest widget](Changelog-Radar/assets/digest-preview.png)

## Output modes

Three modes are available.

**Widget (default):** The inline chat widget shown above. Renders automatically on every digest run. Covers all in-scope items regardless of API impact.

**Plain text:** Three markdown tables in the same three-section order. Request this explicitly if you prefer text output or are working in an environment where the widget does not render well.

**Engineering publication:** A self-contained HTML artifact with severity classifications (Action Required, Verify, Info), a table of contents, and callout blocks per item. Filtered to items with direct API or SDK impact. Request this explicitly when you want something to share with an engineering team or publish as a changelog diff.

## How to install

Changelog Radar installs into Claude as a custom skill. No code required. The process is download, zip, upload.

### Step 1: Download the skill files

1. At the top of this repository page, click the green **Code** button, then **Download ZIP**.
2. Save it somewhere easy to find, like your Downloads folder.
3. Unzip the downloaded file. On Windows, right-click and choose **Extract All**. On Mac, double-click it.

### Step 2: Make the skill zip

Claude installs a skill from a single zip whose top level is the skill folder itself (the folder that contains SKILL.md). You zip one inner folder, not the whole download.

1. Open the unzipped folder. Inside, find the folder named **Changelog-Radar**. It contains `SKILL.md`, an `assets` folder, and a `references` folder. This is the skill.
2. Zip that **Changelog-Radar** folder by itself. On Windows, right-click it, choose **Send to**, then **Compressed (zipped) folder**. On Mac, right-click it and choose **Compress "Changelog-Radar"**.
3. You now have **Changelog-Radar.zip**. This is what you upload. Do not unzip it again.

If you open the zip to check, the **Changelog-Radar** folder sits at the very top, with `SKILL.md` directly inside it. That is correct.

### Step 3: Turn on code execution

Skills need this to run. In Claude, go to **Settings** and make sure **Code execution and file creation** is turned on.

### Step 4: Install the zip

1. In Claude, go to **Customize**, then **Skills**.
2. Click the **+** button, then **+ Create skill**.
3. Upload your **Changelog-Radar.zip**.
4. The skill appears in your skills list. Toggle it on.

### Step 5: Use it

Start a new chat and tell Claude what you want to track, for example "set up Changelog Radar for my Zoom and Twilio stack." The skill interviews you about what you use, then returns only the changes that matter. To get a digest later, just ask for one.

Note: this is the stateless chat version. Your tracked vendors live in the conversations where you set them up. Delete that conversation and you will need to set those vendors up again.

## How memory works

This is the stateless chat version of the skill. It writes nothing to a file, a repo, or your account instructions. Your tracked vendors and interest profile live only in the conversations where you set them up. When you ask for a digest, the skill silently searches those past conversations to recover your setup and proceeds from there.

One hard consequence: if you delete the conversation where you onboarded a vendor, that vendor's setup is gone. There is no backup. You would need to onboard from scratch. If you want your setup to survive independent of chat history, the Projects version of this skill stores the registry as a Project file and is the correct solution.

This version does not run on a schedule. You run a digest by asking for one.

## Repo structure

```
changelog-radar/                          # repository root
├── LICENSE                               # CC BY-NC-ND 4.0 license text
├── NOTICE                                # Copyright and attribution notice
├── README.md                             # This file
├── profile.example.yaml                  # Example interest profile
└── Changelog-Radar/                      # The skill folder (this is what you zip and upload)
    ├── SKILL.md                          # Skill instructions loaded by Claude
    ├── assets/
    │   ├── digest-preview.png            # Screenshot of the default widget output
    │   ├── engineering-template.html     # Base stylesheet for the HTML publication mode
    │   └── registry.example.yaml         # Example registry entries (Zoom, Anthropic)
    └── references/
        ├── output-formats.md             # Defines the three output modes and their rules
        ├── ingestion-channels.md         # Per-channel sourcing workflows (RSS, email, search/scrape)
        └── registry-schema.md            # Schema for vendor registry entries and interest profiles
```

## License

Licensed under [CC BY-NC-ND 4.0](https://creativecommons.org/licenses/by-nc-nd/4.0/) (Attribution-NonCommercial-NoDerivatives). You may share this work for noncommercial purposes with attribution. You may not use it commercially or distribute derivative versions. All rights not expressly granted are reserved; see the LICENSE and NOTICE files. For a commercial license, contact [@ungatedbluemarble](https://github.com/ungatedbluemarble).
