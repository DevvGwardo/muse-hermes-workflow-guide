<p align="center">
  <img src="assets/banner.svg" alt="Pale blue sky, soft sun, drifting clouds, and layered green hills" width="100%">
</p>

<h1 align="center">muse + hermes agent</h1>
<p align="center"><em>a workflow guide, told like a storybook</em></p>

<p align="center">
  ☁️ &nbsp; Muse thinks in the cloud &nbsp; · &nbsp; 🌉 &nbsp; Tailscale carries the words &nbsp; · &nbsp; 🐱 &nbsp; Hermes works on the desktop &nbsp; ☁️
</p>

---

## what this is

A single-page guide that explains how to build the workflow between
**Muse** (the agent at [muse.ai](https://muse.ai)) and **Hermes Agent**
(the agent running on your own desktop): how they pair, how they pass
messages, and how the daily rhythm works once they are connected.

It is written for someone setting the whole thing up from scratch. No
prior knowledge of either agent is assumed, and every step says what
can go wrong and how to fix it.

Open `index.html` in a browser and scroll. The sky moves, the clouds
drift, and a small cat walks the path from step one to step six.

## what's inside

- **the diagram** — an animated Muse → Tailscale → Hermes flow with a
  Play button, so the shape of the system lands before any detail
- **the pre-start checklist** — an interactive list with a progress ring,
  so nothing gets skipped
- **six setup steps** — Tailscale, Hermes Duet pairing, the small Python
  bridge, the five-minute watcher, the first handoff, and the daily rhythm
- **three playbooks** — copyable handoff templates for research, builds,
  and check-ins
- **house rules** — the working agreements that keep two agents (and one
  human) from stepping on each other
- **troubleshooting and FAQ** — the failures seen in the wild, and the
  questions everyone asks first

## view it

No build step, no server, no dependencies. The page is one
self-contained file:

```sh
open index.html
```

Or turn on GitHub Pages for this repo (Settings → Pages → Deploy from
branch → `main`, root folder) and share the link it gives you.

## the workflow, in short

```
  Muse (cloud)  ── Tailscale ──►  Hermes (desktop)
       │                              │
       │   small Python bridge        │   does the local work
       │   GET /api/state             │   files, programs, commands
       │   POST /api/messages         │
       │                              │
       └── five-minute watcher ───────┘
           new messages surface in chat
```

1. **Tailscale** joins the cloud VM and the desktop on one private network.
2. **Hermes Duet pairing** trades a one-time code for a session cookie.
   The cookie is private. If a check ever answers 401, re-pair.
3. **The bridge** is a small script: fetch state, send messages, and
   nothing else.
4. **The watcher** checks every five minutes and compares message IDs
   against a seen list. The seen list merges, never overwrites, so no
   message is ever reported twice or lost.
5. **The first handoff** follows a template: what to do, what is already
   known, and what must come back.
6. **The daily rhythm** is one morning brief, one evening wrap, and
   silence in between unless something is new.

## files

| file | what it is |
| --- | --- |
| `index.html` | the whole guide, self-contained, assets inlined |
| `assets/banner.svg` | the sky-and-hills banner at the top of this readme |
| `.nojekyll` | tells GitHub Pages to serve the page as-is |

## the theme

The page is styled after [maiavm.com](https://www.maiavm.com): a pale
storybook sky, flat clouds, a soft sun, layered green hills, cream
cards, navy headings, and lowercase rounded serif type. The cat icon
in the page is the maiavm.com icon, bundled into the file.

| swatch | hex | where it lives |
| --- | --- | --- |
| sky, top | `#C3E1FF` | page gradient |
| sky, bottom | `#D8ECFF` | page gradient |
| sun | `#FFE6AA` | the soft sun |
| hill, back | `#C1E2CE` | far hills |
| hill, middle | `#B4DCC5` | middle hills |
| hill, front | `#96CDAC` | near hills |
| card | `#FFFDF7` | cream cards |
| headings | `#16365F` | navy headings |
| buttons | `#091F3D` | primary buttons |
| accent | `#A8361F` | rust italic accents |

## notes

- Every hostname, token, and address in the guide is a **placeholder**.
  The page contains no real hostnames and no secrets.
- The guide describes the workflow as it ran in November 2026. Endpoints
  and app behavior may drift; the troubleshooting section is the first
  place to look when something stops matching.
- Built with Muse, November 2026. Animation passes kept the content and
  the icon untouched.

<p align="center">🌤️ ☁️ 🐱 🌿</p>
<p align="center"><em>you are connected.</em></p>
