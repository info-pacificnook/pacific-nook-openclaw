# gog Setup Guide

Set up `gog` to access Google Workspace (Gmail, Calendar, Drive, Contacts, Sheets, Docs, etc.) via OAuth.

## Server Setup (Admin, once)

### 1. Install gog CLI

```bash
brew install steipete/tap/gogcli
# or: npm install -g gogcli
```

### 2. Create Google OAuth Credentials

1. Open [Google Cloud Console](https://console.cloud.google.com/)
2. Create a project (or use an existing one)
3. Go to **APIs & Services > Library**, enable:
   - Gmail API, Google Calendar API, Google Drive API, People API,
     Google Sheets API, Google Docs API, Google Chat API, Google Tasks API
4. Go to **APIs & Services > Credentials**
5. Create **OAuth 2.0 Client ID** (type: **Desktop app**)
6. Download the JSON → `client_secret_xxx.json`
7. If prompted for OAuth consent screen: set to **External** (or Internal for Workspace),
   add test users or publish the app

### 3. Register credentials

```bash
gog auth credentials /path/to/client_secret_xxx.json
```

### 4. Configure keyring for headless server

```bash
gog auth keyring file
```

### 5. Set shared environment

In `~/.openclaw/.env`:

```
GOG_KEYRING_PASSWORD=<your-secure-passphrase>
```

This passphrase encrypts all stored tokens. It must be set **before** any `gog auth add` and must be the same value everywhere gog runs (gateway, agents, cron).

> **Do NOT set `GOG_ACCOUNT` globally.** Each user/agent sets their own account.

---

## Per-User Onboarding (each channel user)

Each user authorizes their own Google account. On a headless server, use the **remote** auth flow:

### Step 1: Generate the auth URL

```bash
gog auth add USER@gmail.com --services all --remote --step 1 --timeout 10m
```

This prints an `auth_url`. Send this URL to the user to open in their browser.

Use `--services all` for full access, or pick specific services:

```bash
gog auth add USER@gmail.com --services gmail,calendar,drive,contacts,sheets,docs --remote --step 1
```

### Step 2: User authorizes in browser

The user opens the URL, signs in with their Google account, and grants permissions.
After authorization, the browser redirects to a `127.0.0.1` URL that won't load.
The user copies the **full URL** from the browser address bar.

### Step 3: Exchange the code

```bash
gog auth add USER@gmail.com --services all --remote --step 2 --auth-url '<PASTE_REDIRECT_URL_HERE>'
```

This exchanges the OAuth code for a refresh token, encrypted with `GOG_KEYRING_PASSWORD`.

### Step 4: Verify

```bash
gog auth list
```

Should show the new account with all authorized services.

### Step 5: Configure the agent

For per-agent setup, set `GOG_ACCOUNT` in the agent's environment or config:

```bash
openclaw config set skills.entries.gog.env.GOG_ACCOUNT "USER@gmail.com"
```

Or for multi-agent setups with separate workspaces, set it per-agent.

---

## Multiple Accounts

gog supports multiple accounts in the same keyring. Each user's tokens are stored separately:

```bash
gog auth list
# user1@gmail.com    default    gmail,calendar,drive,...
# user2@company.com  default    gmail,calendar,drive,...
```

Use `--account` to target a specific account:

```bash
gog gmail search 'newer_than:1d' --account user1@gmail.com
```

Or set `GOG_ACCOUNT` per agent/session.

---

## Troubleshooting

- **`aes.KeyUnwrap(): integrity check failed`**: The `GOG_KEYRING_PASSWORD` doesn't match what was used when the token was stored. Re-run `gog auth add` with the correct password set.
- **`no TTY available for keyring file backend password prompt`**: `GOG_KEYRING_PASSWORD` env var is not set. Ensure it's in `~/.openclaw/.env`.
- **"Access blocked" during OAuth**: Add the user's email as a test user in Google Cloud Console > OAuth consent screen.
- **"API not enabled"**: Enable the required API in Google Cloud Console > APIs & Services > Library.
- **Token expired**: Re-run the auth flow (`gog auth add ... --remote --step 1/2`).
- **Check auth state**: `gog auth list` shows all registered accounts and services.
