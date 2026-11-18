# Muse + Hermes: the file-drop workflow guide

Link Muse (in the cloud, at muse.ai) to Hermes Agent (on your desktop) so they pass jobs back and forth like one calm team. Muse chats, plans, and checks the web and mail. Hermes does the hands-on work where your files live. Jobs travel as plain files over Tailscale file sharing. Setup takes about 20 minutes.

One rule: you approve every outward step. Nothing sends, posts, or pays without you.

## How it works

```
Muse (cloud) -- task-001.md --file sharing--> Hermes (desktop)
     ^                                              |
     |           report-001.md                      |  does the work
     +--------------file sharing--------------------+
```

1. Muse writes a task file and sends it to the desktop.
2. Hermes watches the inbox folder, does the job, writes a report file, and sends it back.
3. Muse checks for new reports every 5 minutes and summarizes them for you.

## What you need

- A Muse account at muse.ai, signed in and chatting.
- Hermes Agent running on your desktop. The command line version is fine.
- Tailscale on both sides, same account, same tailnet.
- Tailscale file sharing turned on: in the Tailscale admin console, Settings, General, Send Files. Both devices must be online at the moment a file is sent. File sharing does not hold files for an offline device.
- Desktop on, online, and not asleep.
- About 20 minutes.

## Step 1: Connect with Tailscale and turn on file sharing

Install Tailscale and sign in with the same account on both sides. Turn on Send Files in the admin console. Note each device's MagicDNS name. Keep those names private. Never publish them.

Check: both devices show as online in Tailscale, and `tailscale file cp --targets` lists the other device.

## Step 2: Agree the file convention

Jobs are files. Use one naming rule on both sides so the watcher can tell what is new:

```
task-001.md    -> the job, written by Muse
report-001.md  -> the result, written by Hermes
```

One job at a time per number. Never reuse a number. The task file holds the handoff template: task, files or paths, what done means, and what to report back.

## Step 3: Send the first task file

Muse writes the task file, then sends it. The colon after the target name is required.

```
tailscale file cp task-001.md YOUR-DESKTOP:
```

On the desktop, the file lands in the file sharing inbox. Confirm once where that is on your setup (on Windows it arrives through the Tailscale app; on Linux it waits in the inbox until fetched). Tell Hermes the inbox folder once, and Hermes watches it from then on.

## Step 4: Hermes does the job and sends the report back

Hermes picks up the task file, does the work, and writes a report file with the same number:

```
report-001.md
- what changed
- file paths
- how it was checked
- anything that still needs the human
```

Then Hermes sends it back the same way:

```
tailscale file cp report-001.md YOUR-CLOUD-VM:
```

No secrets in task or report files. No passwords, API keys, or tokens, in either direction.

## Step 5: Add the watcher

On the Muse side, run a check every 5 minutes. It fetches newly arrived files and reports only reports it has not seen before.

```
# fetch anything waiting in the inbox into a reports folder
tailscale file get ./reports

# then compare report file names against a seen list, and summarize only the new ones
```

The dedup rule: the seen list merges, never overwrites. If you overwrite it, old reports look new again.

## Step 6: Make it a habit

```
morning  - Muse checks overnight reports
midday   - one handoff at a time
evening  - Hermes reports, Muse summarises
```

One job at a time per task number. One clear owner per task.

## What to expect

- Reports arrive on the 5-minute check, not instantly. A job that finishes between checks shows up at the next one.
- Approvals happen in chat with Muse, before the task file is sent. Nothing outward goes out on a file alone.
- Both devices must be online when a file is sent. If the desktop is off, the send waits or fails, and you resend when it is back.
- File discipline matters. A misnamed file or a reused number confuses the watcher.
- Everything is plain files. Every job and every report is searchable, copyable, and easy to audit later.

## House rules

1. You outrank both agents. If you say stop, both stop.
2. Outward steps need approval. Sends, posts, payments, and publishes need you plus an approval first.
3. Never send secrets in chat or in task files.
4. Research needs receipts. Every claim needs a URL, a quote, and a timestamp.
5. Context travels as a file. Share a readable file path with provenance, never a paraphrased fact as if it were the source.
6. Unsure means ask. A short pause beats a wrong action.

## When it breaks

| Problem | Fix |
| --- | --- |
| Send fails, target not listed | Check both devices are online in Tailscale and Send Files is on. Check the target name ends with a colon. |
| File sent, nothing arrives | On Linux, files wait in the inbox until fetched: run `tailscale file get` on the receiving side. On the desktop, confirm where file sharing delivers files. |
| Watcher repeats old reports | Restore the full seen list and switch to merge, not overwrite. |
| No report back | Check Hermes is running and watching the inbox folder, then resend the task file once and wait. |
| Desktop slept mid job | Wake it, confirm Hermes is running, ask for a status before resending anything. |

## Option B: use a private git repo

If you want no Tailscale at all: use a private git repo. Muse commits task files, Hermes pulls, works, and commits report files, Muse reads them back. Slower than file sharing, and both sides need git access, but it works when devices are offline at different times, and the full history is kept by git. Same file convention, same house rules.

## Notes

Every hostname in this guide is a placeholder. There are no real hostnames and no secrets here. This version has not been run live yet, so expect to adjust the folder paths once for your own setup. File sharing details follow Tailscale's own behavior; if a command or inbox location differs on your setup, trust what your Tailscale install shows.

Built with Muse, November 2026.
