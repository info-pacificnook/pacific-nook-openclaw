# TOOLS.md - Local Notes

Skills define _how_ tools work. This file is for _your_ specifics — the stuff that's unique to your setup.

## What Goes Here

Things like:

- Camera names and locations
- SSH hosts and aliases
- Preferred voices for TTS
- Speaker/room names
- Device nicknames
- Anything environment-specific

## Examples

```markdown
### Cameras

- living-room → Main area, 180° wide angle
- front-door → Entrance, motion-triggered

### SSH

- home-server → 192.168.1.100, user: admin

### TTS

- Preferred voice: "Nova" (warm, slightly British)
- Default speaker: Kitchen HomePod
```

## Why Separate?

Skills are shared. Your setup is yours. Keeping them apart means you can update skills without losing your notes, and share skills without leaking your infrastructure.

---

## gog (Google OAuth CLI)

`gog` is installed at `~/.npm-global/bin/gog`. Use it to interact with all Google Workspace APIs.

### Authorizing accounts — ALWAYS use `--services=all`

When running `gog auth add` for any account (new user, re-auth, or token refresh), **always** pass `--services=all`:

```bash
GOG_KEYRING_PASSWORD=WCtTxVgPlchI-jD-sIFfPg gog auth add <email> --services=all
```

**Why:** `--services=all` grants every user-accessible Google service (Gmail, Drive, Docs, Sheets, Calendar, Contacts, Tasks, Slides, Forms, Chat, Classroom, AppScript, People). The default (`--services=user`) only grants OIDC identity scopes — users will get permission errors on any API call. Never authorize with a subset; users shouldn't have to worry about which services are available once connected.

If a user reports a permission error on any Google service (Sheets, Drive, etc.), re-authorize with:
```bash
GOG_KEYRING_PASSWORD=WCtTxVgPlchI-jD-sIFfPg gog auth add <email> --services=all --force-consent
```

### Current accounts

- `info@pacificnook.com` — main account, all services authorized

### `gog docs` vs `gog sheet` — How to Tell Which One to Use

Google Drive has two distinct file types that look similar. Use the URL or file ID to determine which command to use:

| Signal | Use |
|---|---|
| URL contains `/spreadsheets/d/` | `gog sheet` |
| URL contains `/document/d/` | `gog docs` |
| File ID from Drive — check MIME type | see below |

**If you only have a file ID** (no URL), check the MIME type first:
```bash
GOG_KEYRING_PASSWORD=WCtTxVgPlchI-jD-sIFfPg gog drive get <fileId> --account info@pacificnook.com --json | python3 -c "import sys,json; d=json.load(sys.stdin); print(d.get('mimeType',''))"
```
- `application/vnd.google-apps.spreadsheet` → `gog sheet`
- `application/vnd.google-apps.document` → `gog docs`

**Quick rule:** If the user says "sheet", "spreadsheet", "cells", or "rows" → `gog sheet`. If they say "doc", "document", or "text" → `gog docs`.

Add whatever helps you do your job. This is your cheat sheet.
