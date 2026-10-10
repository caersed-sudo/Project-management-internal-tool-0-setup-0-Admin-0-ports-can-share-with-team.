<div align="center">

# PM Tool — Portable Team Project Management

**Kanban boards, workday reminders, success plans and conflict-safe team sync — in a single folder that runs on your own machine.**

No server · No accounts · No admin rights · Nothing leaves your machine except through the shared folder you choose

![release](https://img.shields.io/badge/release-1.4.0-1d4fd7)
![platforms](https://img.shields.io/badge/Windows%20·%20macOS%20·%20Linux-x64%20&%20arm64-0a1322)
![stack](https://img.shields.io/badge/Next.js%2015%20·%20React%2019%20·%20Tailwind%20·%20Prisma%20·%20SQLite-blueviolet?color=2f80ff)
![tests](https://img.shields.io/badge/unit%20tests-82%20passing-0fa968)
![sync protocol](https://img.shields.io/badge/sync%20protocol-pm--sync%2F1-7a5af8)

![PM Tool dashboard — projects with stat chips, reminders, sync pill](docs/img/readme-dashboard.png)

</div>

## What is this?

PM Tool is a complete project-management app that runs **entirely on your own machine**.
One SQLite file holds everything; team sharing happens through a folder you already have —
a network drive, an SMB share, or a Dropbox-style synced folder. Each project syncs as one
plain JSON snapshot where the newest edit always wins, and the app keeps working while the
share is unreachable (it catches up automatically when it returns).

It's for small teams that want Trello-style boards **without** a SaaS subscription: legal or
compliance constraints, air-gapped or low-connectivity offices, consultants carrying work on
a USB stick, or anyone who prefers their project data to stay theirs.

## Features at a glance

- 🗂 **Kanban board** — five columns (Backlog → Done) with drag & drop, four priorities shown as colored card edges, due dates, subtasks and assignee avatars; completed tasks are tracked as *done on time* vs *done overdue*.
- 🎯 **Success plan** — a two-minute kickoff form (Deliverable · Outcome · Experience · Handover · Value, each with a delivery level 🟢🔵🟣) that stays pinned on the project so "winning" is defined once and never argued later. Syncs with the project.
- ⏰ **Workday reminders** — the sidebar groups everything due within two working days into *My Tasks* / *Team Tasks*, correctly skipping weekends and flagging overdue items.
- 👥 **One machine, many people** — everyone just types their own name at the welcome prompt; no passwords on a shared office PC, but each person's tasks, activity and identity stay separate.
- 🔄 **Shared-folder sync in ~1 second** — green pill = synced, red = offline (keep working, it retries), yellow *Solo mode* = sync off for travel.
- 🛡 **Conflict-safe by design** — field-level editing locks ("Alice is editing…"), newest-wins snapshots, a conflict toast with *apply theirs / keep yours* when both sides edited while offline, and your previous version saved to `.prev.json` before anything is overwritten.
- 💾 **Automatic backups** — hourly snapshots (last 10 kept) plus one at startup; one-click restore; a red integrity banner walks you through disaster recovery if the database is ever corrupted.
- 📤 **Export / Import JSON** — move a whole workspace through e-mail, chat or USB even when the shared folder is unreachable. *Merge* never deletes and never rolls back newer edits; *Replace* asks twice and refuses while teammates are mid-edit.
- 🕵️ **Local activity journal** — your own actions on this machine, newest first, capped at the last 100 per person. Nothing leaves your disk.
- 🎨 **Appearance** — light / dark / system theme, five accent colors, compact cards and reduce-motion; saved per browser, applied instantly (dark mode included).
- 📖 **Built-in interactive guide** — every screen and action documented inside the app: open `/guide/index.html` while running.

## Screens

| Kanban board | Dark mode |
|:---:|:---:|
| ![Kanban board with tinted columns and priority-edged cards](docs/img/readme-board.png) | ![Board in dark mode](docs/img/readme-board-dark.png) |

---

## Install & run (colleague, non-developer)

You received a folder called **pm-tool-dist** (or the zip that contains it). Copy it anywhere — USB stick, Documents, a synced drive. Nothing is installed and nothing is written outside the folder; a portable Node runtime ships inside it.

### Windows
1. Double-click **Launch-PM-Tool.bat**. A console window opens; your browser opens at
   **http://127.0.0.1:3000/** (the app is only reachable from your own machine).
2. To stop, close the console window.

### macOS / Linux (from the same folder)
```sh
cd pm-tool-dist
./start.sh          # portable Node is bundled — nothing to install
```
(If copying from a USB stick dropped the executable bit, run `sh start.sh`.)
Then open http://127.0.0.1:3000/

### First launch
* The app asks **"who are you?"** — type *your own name*. You always act as yourself;
  your tasks appear under **My Tasks**, everyone else's under **Team Tasks**.
  (Change it anytime in Settings → Profile.)
* Your data lives in `app/data/pm.db` inside the folder.

## Connect to the team shared folder

The setup modal appears on first run: paste the shared folder path
(e.g. `\\fileserv\team\pm-sync` or `/mnt/team/pm-sync`) and hit **Test**, then **Connect**.
Later you can change it in **Settings → Sync**.

* Green pill **Synced** — up to date; edits appear on teammates' machines within ~1 second.
* Red pill **⚠ Offline — retrying** — the folder can't be read/written right now. Keep
  working; nothing is lost, it retries automatically and catches up when the share returns.
* Yellow **Solo mode** (Settings → Sync) — sync fully off; good for travel. Turn it off to rejoin.

If the folder doesn't exist yet, any machine can create it — first connect wins nothing,
they just start sharing from then on.

## Backups & recovery

* **Automatic:** every hour (configurable in Settings → Data) a snapshot is written to
  `app/data/backups/` (last 10 kept). One also runs at startup.
* **Restore:** Settings → Data → Restore puts the newest backup back.
  ⚠ Restore is *disaster recovery, not time travel*: after restoring, any **newer**
  version of a project sitting in the shared folder will flow back to you on the next
  sync (the protocol always keeps the newest edit). Tell teammates before restoring.
* **Corrupt database:** if the app refuses to start with a red integrity banner, your DB
  failed its health check. The screen shows your most recent backup path — copy it over
  `app/data/pm.db` (with the app closed) and start again. Worst case, teammates' copies
  + the shared folder still hold every project's newest snapshot.

## Export & import (hand-off files)

Settings → Data lets you move the whole workspace through **any** channel — e-mail, chat,
USB stick — even when the shared folder is unreachable or offline:

* **Export all data (JSON):** downloads one file `pm-tool-export-<date>.json` containing
  every project (tasks included), every team member, and your settings. It never contains
  in-progress editing locks.
* **Import:** pick such a file and choose a mode. A backup is taken automatically first,
  and a malformed file is rejected whole — nothing is ever partially applied.
  * **Merge** – adds new projects and updates existing ones by the newest-wins rule.
    It never deletes anything and never rolls back newer edits; if it would clobber an
    edit you haven't synced yet, you get the normal conflict toast instead.
  * **Replace** – restores the full exported state (two confirmations). Refused while a
    teammate is actively editing anything; try again in a few seconds.

## How sync works (60-second version)

* Every project lives in the shared folder as **one JSON snapshot** stamped with a
  comparison tuple (edit counter + wall-clock tiebreakers). Pushes and pulls run on a
  ~1-second scheduler.
* **Newest wins**, ties break deterministically, and a malformed snapshot file is skipped
  and logged — one broken file never stalls the team. `npm run validate-sync -- /path/to/folder` audits it.
* Editing locks are tiny lease files with ~5-second TTLs, so a crashed machine can't hold
  a field hostage.
* Full specification: **[docs/sync-protocol.md](docs/sync-protocol.md)** · Explorable architecture: **[docs/architecture.html](docs/architecture.html)**

## Data & privacy

* The app binds **127.0.0.1** — it never opens a port other machines (or the internet) can reach; the shared folder is the only channel between copies.
* All state lives in one portable SQLite file plus your chosen sync folder. No telemetry, no cloud services, no accounts.
* Uninstall = delete the folder.

## For developers

```sh
npm install          # Node ≥ 20 recommended (portable bundles ship Node 22)
npm run dev          # http://127.0.0.1:3000 — classic dev server with hot reload
npm test             # 82 unit tests (node:test): sync, merge, locks, conflicts, backups…
npm run build        # classic production build (next start works)
PM_STANDALONE=1 npm run build && bash scripts/make-dist.sh   # portable bundle → release/pm-tool-dist
npm run validate-sync -- /path/to/shared-folder            # audit snapshot files
node scripts/soak.mjs --hours 24                           # soak harness (see docs/e2e-report.md)
```

| Path | What lives there |
|---|---|
| `src/app/` | Next.js 15 App Router pages (dashboard, project board, activity, settings) + API routes |
| `src/components/` | React 19 UI — kanban (dnd-kit), modals, reminders sidebar, success-plan card |
| `src/lib/` | sync engine, scheduler, merge/lock/conflict logic, dates, export/import |
| `prisma/` | SQLite schema (+ seed) |
| `scripts/` | `make-dist.sh` portable bundler, `backup.ts`, `validate-sync.ts`, soak harness |
| `docs/` | sync protocol spec, e2e validation report, architecture diagram, screenshots |

**Stack:** Next.js 15 · React 19 · TypeScript (strict) · Tailwind CSS · Prisma + SQLite · @dnd-kit.
The UI is fully keyboard-friendly and respects *reduce motion*; theme/appearance never touches the database, so it can't sync-conflict.

## Troubleshooting (most common first)

| Symptom | What it means / what to do |
|---|---|
| **⚠ Offline — retrying** pill | Shared folder unreachable (drive disconnected, VPN off, permissions). Fix access; sync self-heals within seconds. Your local work is safe meanwhile. |
| Field says **"Alice is editing…"** | A teammate holds that field lock while typing. Locks expire by themselves after ~5 s of inactivity — try again shortly or coordinate. You can take over deliberately from the dialog. |
| Same project edited on two machines → **conflict toast** | Both changed it while offline from each other. Choose *apply theirs* / keep yours / view changes; your version is saved to a `.prev.json` before anything is overwritten. Nothing is destroyed silently while you're watching. |
| App won't start: **integrity banner** | Database corrupt. Restore from `app/data/backups/` (see Backups & recovery). Never delete pm.db without copying it away first. |
| Port 3000 already in use | Close the other program, or edit the PORT=3000 line in Launch-PM-Tool.bat (`PORT=3010 ./start.sh` on macOS/Linux) and open that port in your browser. |
| A `*.json` file in the shared folder looks broken | Harmless to running apps — bad files are skipped and logged. A developer can audit with `npm run validate-sync`; delete the offending file after everyone has synced. |

## Version strings

App version: Settings → About (also `package.json`). Sync protocol: **pm-sync/1** — the
snapshot shape in [docs/sync-protocol.md](docs/sync-protocol.md); files are not self-stamped, so a breaking change
means bumping both the About string and the validator.

---

*Guide inside the app at `http://127.0.0.1:3000/guide/index.html` — every screen, every button, with screenshots.*
