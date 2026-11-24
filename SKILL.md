---
name: job-opportunity-share
description: Turns a job/graduate-programme/bursary posting link into a ready-to-send WhatsApp message for the Dell Young Leaders group, AND checks the team's shared log so nobody shares the same opportunity twice. Use this whenever the user shares a job posting, internship, graduate programme, or bursary link (or a LinkedIn job URL, a company careers-page URL, or a PDF opportunity overview) and asks for a message, blurb, or something to "send out", "share with the group", or "post to WhatsApp" about it, or says "job opportunity share". Also trigger if the user says "write this up for the Dell Young Leaders" or pastes a link with no other instruction alongside a mention of Dell Young Leaders or the WhatsApp group. Use this skill instead of job-opportunity-message:job-opportunity-message, which does not check what teammates already shared. Do not use this for the weekly faculty emails (jobmailer) or general email drafting.
---

# Job Opportunity Share (WhatsApp message + already-shared check)

Write a friendly, semi-formal WhatsApp message announcing a job, graduate programme, internship, or bursary opportunity to the Dell Young Leaders group, based on a link the user provides. Before writing, check what the careers team has already shared, and flag it. After writing, log what was shared so teammates see it too.

## Why this matters

Three people on the careers team share opportunities from their own computers. They only avoid sending the same opportunity twice if everyone reads and writes the same list. That list is "Shared opportunities.csv" in the shared OneDrive folder **DYL Careers Hub**, and the script that reads it belongs to the jobmailer skill (`~/.claude/skills/jobmailer/scripts/shared_log.py`).

There are two routes, and they are treated differently:
- **WhatsApp** (this skill): if the opportunity was already shared on WhatsApp, that is a **problem**: flag it clearly.
- **Weekly mailer** (jobmailer): if it only went out in a mailer, just **mention** it (FYI). It is not a problem to also post it on WhatsApp.

Flag, never refuse: still write the message even when it is flagged, so the user can decide.

## Step 0: Get the latest scripts and check the log is connected

```bash
git -C "$HOME/.claude/skills/jobmailer" pull --ff-only
python "$HOME/.claude/skills/jobmailer/scripts/shared_log.py" status
```

- If `~/.claude/skills/jobmailer` does not exist, this PC does not have the jobmailer skill yet. Tell the user it is needed for the shared log and point them to the jobmailer user manual (Part 1) to install it. Offer to write the messages without the check only if they insist, and say clearly nothing was checked.
- `CONNECTED`: carry on. If it says the name is **(not set)**, ask the user for their first name once and run `python "$HOME/.claude/skills/jobmailer/scripts/shared_log.py" whoami --set "<first name>"`.
- `NOT CONNECTED`: stop and walk the user through `~/.claude/skills/jobmailer/references/onedrive_setup.md`, ONE step at a time, waiting for them after each step. Run `status` again afterwards. Do not write the messages as if everything was new.

## Step 1: Check the links first

Before opening anything:

```bash
python "$HOME/.claude/skills/jobmailer/scripts/shared_log.py" check-links --route whatsapp URL1 URL2 ...
```

## Step 2: Fetch each link and pull out the details

Use web fetch (or, for a client-rendered page like a LinkedIn job listing, Claude in Chrome: read `anthropic-skills:chrome-browser` first and use the user's own Chrome). If it's a PDF, fetch and read it directly. Read every link, even flagged ones.

Pull out what's there, and only what's there:
- Organization name
- Role / programme name (e.g. "Graduate Programme", "Food Safety Graduate", "Bursary")
- Location, if mentioned
- Application deadline, if explicitly stated
- Field of study / degree requirement, if it is a bursary or a role tied to a specific qualification
- The link itself (use exactly what the user gave you: don't rewrite or shorten it, even a tracking link like an `lnkd.in` shortlink)

If a detail (deadline, field) isn't on the page, simply don't mention it. Never guess or invent one.

Then check again with the real details, which also catches the same job shared under a different link:

```bash
python "$HOME/.claude/skills/jobmailer/scripts/shared_log.py" check-opps --route whatsapp \
  --opp "Company | Opportunity | City | Link" --opp "Company | Opportunity | City | Link"
```

(City = the town, or "Nationwide"; keep the company and opportunity short, e.g. "Faurecia", "Finance Graduate".)

## Step 3: Write the messages

Write one WhatsApp message per link, even for flagged ones. See "Format and tone" below.

## Step 4: Show the already-shared table, then the messages

Start the reply with the table `check-opps` printed (Opportunity | Status | Shared before), then a line for each flagged item:

- **ALREADY SHARED** (on WhatsApp before): put a clear warning above that message, e.g. "Warning: already shared on WhatsApp on 24 Nov 2026 by Anokunda. The group has seen this one. The message is below in case you still want to send it."
- **FYI: other route** (only in a weekly mailer): one calm line above that message, e.g. "Note: this was in the weekly mailer on 12 Nov 2026 (shared by Anokunda). Fine to post on WhatsApp as well."
- **New**: no note.

If nothing was shared before, say so in one line. Then give the messages, each in its own ```text code block so it can be copied (see Output).

## Step 5: Log it

After giving the messages, record the opportunities that are not already in the log as WhatsApp shares:

```bash
python "$HOME/.claude/skills/jobmailer/scripts/shared_log.py" record-whatsapp \
  --opp "Company | Opportunity | City | Link" --opp "..."
```

Only include the ones that were **New** or **FYI** (the script skips any already logged as WhatsApp anyway). Tell the user in one line: "Logged as shared on WhatsApp by <name>. If you decide not to send one, tell me and I'll take it out." If they say so, run:

```bash
python "$HOME/.claude/skills/jobmailer/scripts/shared_log.py" unrecord --route whatsapp --link "<link>"
```

If a script says the log is open in Excel, ask the user to close it and run the command again.

## Format and tone

Semi-formal and encouraging: the register of someone who respects the reader but is genuinely rooting for them. Not corporate-brochure, not slangy. These messages go to young people the sender wants to encourage, so the tone should feel like a mentor personally flagging something worth their time: warm and specific, never a copy-pasted job alert. The user already filters which links go to which sub-group, so don't judge fit or eligibility. Just represent the opportunity accurately and enthusiastically.

Always address the reader as "Dell Young Leader" (e.g. "Hi Dell Young Leader," or "Hello Dell Young Leaders,"), generic and not tailored to a sub-group.

Vary these elements naturally from message to message so it doesn't read as a template being filled in:

- **Greeting**: e.g. "Hi Dell Young Leader," / "Hello Dell Young Leaders, hope you're well!" / "Hey there, Dell Young Leader!" Occasionally skip a separate greeting line and open straight into the news.
- **Opening the news**: e.g. "I've got some exciting news to share!" / "We're excited to announce that applications are now open for..." / a direct lead with the opportunity itself.
- **The pitch**: 1-3 sentences naming the organization, the role/programme, and (only if genuinely useful) a line encouraging them to apply. Keep it proportional to how much real information the posting gives: a thin posting gets a short message.
- **Deadline callout**: only if the page states one. A friendly heads-up ("just a heads-up, applications close August 15"), not a warning.
- **The link**: always include it plainly, usually introduced with something like "For more details, visit:" or "Check it out here:" or "you can apply here:".
- **Sign-off**: often present but not mandatory, e.g. "Good luck if you apply, you've got this!" or no sign-off at all, just ending on the link.
- **Emoji**: sparingly. Most messages should have zero or one, never more than two.

Length varies with content, from a tight 2-sentence announcement to a fuller 4-5 sentence message. Don't pad a thin posting, and don't truncate a rich one.

Never put the "already shared" warning or notes inside the WhatsApp message itself. They go above it, outside the code block.

## Output

Give the user each finished WhatsApp message as plain text in its own ```text code block, ready to copy, not as a file. If you couldn't find something on the page (e.g. no deadline listed), say so in one short aside after the message, outside the code block.

If the user gives several links, produce one distinct message per link, clearly separated, in the same order as the links.
