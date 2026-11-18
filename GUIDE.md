# Muse + Hermes: a simple workflow guide

Link Muse (in the cloud, at muse.ai) to Hermes Agent (on your desktop) so they pass jobs back and forth like one calm team. Muse chats, plans, and checks the web and mail. Hermes does the hands-on work where your files live. Setup takes about 20 minutes.

One rule: you approve every outward step. Nothing sends, posts, or pays without you.

Full illustrated version: https://github.com/DevvGwardo/muse-hermes-workflow-guide

## What you need

- A Muse account at muse.ai, signed in and chatting.
- Hermes Agent and Hermes Duet running on your desktop.
- Tailscale on both sides, same account, same tailnet.
- Desktop on, online, and not asleep.
- About 20 minutes.
- A pairing code, grabbed in step 2. It lasts 10 minutes.

## Step 1: Connect with Tailscale

Install Tailscale and sign in with the same account on both sides. On the desktop, note its MagicDNS name. It looks like the placeholder below. Keep it private. Never publish it.

```
https://YOUR-DESKTOP.<tailnet>.ts.net
```

Check: the desktop shows as online in Tailscale, and both sides use the same account. Different accounts will never see each other.

## Step 2: Pair Hermes Duet

Open Hermes Duet on the desktop and ask for a pairing code. Use it right away, it expires after 10 minutes.

```
POST /api/pair
Body: { "code": "YOUR-PAIRING-CODE" }
```

Save the returned session cookie to a private file and set tight permissions. Never share the cookie.

```
chmod 600 cookies.txt
```

If a check ever answers `401 not paired`, get a fresh code and pair again.

## Step 3: Add the bridge

A small Python bridge talks to the Hermes Duet API, standard library only. Key endpoints: `GET /api/state` to list channels and items, and `POST /api/messages` with text and channel_id to send. The default channel is `chan-general`.

```python
# bridge.py - tiny sketch, standard library only
import json, urllib.request, pathlib
BASE = "https://YOUR-DESKTOP.<tailnet>.ts.net"
COOKIE = pathlib.Path("cookies.txt").read_text().strip()

def call(path, data=None):
    req = urllib.request.Request(BASE + path,
        data=json.dumps(data).encode() if data else None,
        headers={"Cookie": COOKIE, "Content-Type": "application/json"})
    with urllib.request.urlopen(req) as r:
        return json.loads(r.read())

# list channels: call("/api/state")
# send a note: call("/api/messages", {"channel_id": "chan-general", "text": "hello"})
```

Use it like this:

```
python3 bridge.py state
python3 bridge.py send chan-general "hello from Muse"
python3 bridge.py fetch chan-general
```

Never hardcode a real address into a shared copy of the bridge. Keep the placeholder shape and swap in your own private address locally.

## Step 4: Add the watcher

Run a check script every 5 minutes. It fetches `/api/state`, compares item ids with `last_seen.json`, and reports only new items from Hermes.

```
# cron idea: run every 5 minutes
*/5 * * * * python3 check.py
```

The dedup rule that matters most: when you fetch one channel, MERGE new ids into `last_seen.json`. Never overwrite the file with only that channel. If you overwrite, old messages from other channels look new again.

Mark your own sends so the watcher skips them.

## Step 5: Send your first handoff

Use this template. Say the task, the files, what done looks like, and ask Hermes to report back in the same channel.

```
Task: [one clear job]
Files or paths: [exact paths]
Done means: [what success looks like]
Please report back in this channel with:
- what changed
- file paths
- how you checked it
- anything that still needs me
```

A good reply looks like: "Done. Changed 2 files at src/app.py and tests/test_app.py. Tests pass, 24 of 24. Nothing needs you, except a quick look at the new wording."

## Step 6: Make it a habit

```
morning  - Muse checks overnight reports
midday   - one handoff at a time
evening  - Hermes reports, Muse summarises
```

One job at a time per channel. One clear owner per task.

## Three workflows to copy

Long job on the desktop:

```
python3 bridge.py send chan-general "Task: build the export and run all tests. Report back with file paths and test results."
```

Research with receipts (every claim needs a URL, a short quote, and a fetch time):

```
python3 bridge.py send chan-general "Task: find three options. Every claim needs a source URL, a short quote, and the time you fetched it."
```

Watch and report (post only on change, stay silent otherwise):

```
python3 bridge.py send chan-general "Task: watch this folder and post only when a file changes. Stay silent otherwise."
```

## House rules

1. You outrank both agents. If you say stop, both stop.
2. Outward steps need approval. Sends, posts, payments, and publishes need you plus an approval first.
3. Never send secrets in chat. No passwords, API keys, or tokens in either direction. Use a vault or a local file.
4. Research needs receipts. Every claim needs a URL, a quote, and a timestamp.
5. Context travels as a file. Share a readable file path with provenance, never a paraphrased fact as if it were the source.
6. Unsure means ask. A short pause beats a wrong action.

## When it breaks

| Problem | Fix |
| --- | --- |
| Desktop offline | Check power, Tailscale online, Hermes Duet running. Then resend once. |
| 401 not paired | Get a fresh pairing code in Hermes Duet and pair again. Codes expire after 10 minutes. |
| Watcher repeats old messages | Restore the full id list in `last_seen.json` and switch to merge, not overwrite. See step 4. |
| No reply | Check the channel id is `chan-general`, check Hermes is running, resend once and wait. |
| Slow or stuck job | Ask Hermes for a one line status. Do not send duplicates, or you will create two jobs. |
| Desktop slept mid job | Wake it, confirm Hermes Duet is open, ask for a status before resending anything. |

## Questions

Does Muse control my desktop directly? No. Muse only sends messages through the paired bridge. Hermes decides what to do and works locally.

What if my desktop sleeps? Hermes pauses. Wake the desktop and the job can continue. For long jobs, set the desktop to stay awake.

Can I use more than one channel? Yes. Start with `chan-general`. Add channels later for separate topics.

Is my chat private? The link runs over your private Tailscale network, not the public web. Keep your MagicDNS name and cookie file private, and do not share pairing codes.

## Notes

Every hostname, token, and address in this guide is a placeholder. There are no real hostnames and no secrets here. Endpoints and app behavior may drift over time; the troubleshooting table above is the first place to look.

Built with Muse, November 2026.
