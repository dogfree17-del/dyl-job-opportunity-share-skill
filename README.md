# job-opportunity-share - Dell Young Leaders WhatsApp opportunity messages

A Claude skill that turns a job / graduate-programme / bursary link into a ready-to-send WhatsApp
message for the Dell Young Leaders group, and checks the team's shared log first
(`Shared opportunities.csv` in the shared OneDrive folder **DYL Careers Hub**) so nobody shares the
same opportunity twice. Already shared on WhatsApp = flagged with a warning. Only in a weekly
mailer = just mentioned. It then logs what you share, with your name.

## Install (once per computer)

You need the **jobmailer** skill too (it holds the shared-log script and the OneDrive setup steps):
see https://github.com/dogfree17-del/dyl-jobmailer-skill (MANUAL.md, Part 1). Then:

```powershell
git clone https://github.com/dogfree17-del/dyl-job-opportunity-share-skill.git "$HOME\.claude\skills\job-opportunity-share"
```

Restart the Claude app. In the old Claude plugin list, switch off `job-opportunity-message`
(it does not check the shared log), so Claude uses this one.

## Use

In the Claude app's Code tab: `/job-opportunity-share` followed by the job links.
The first time, Claude helps you connect the shared OneDrive folder and asks for your first name.

## Updates

```powershell
git -C "$HOME\.claude\skills\job-opportunity-share" pull
```
