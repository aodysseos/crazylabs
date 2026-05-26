# CrazyLabs — The Terminal

A multi-device escape-room web app that simulates the HTTP request → API → database communication chain. Three groups of players (Client, API, Database) exchange pseudo-code messages in real time through a Matrix-themed terminal interface. Built with vanilla JS + Supabase Realtime, hosted on Netlify — no build step required.

---

## 1. Supabase Setup

1. Go to [https://supabase.com](https://supabase.com) and create a free account.
2. Click **New project**. Choose a name, a strong database password, and select the **EU (West) — Ireland** region (or whichever is closest to your players). Wait ~2 minutes for provisioning.
3. Open the **SQL Editor** (left sidebar) and run the following schema:

```sql
create table messages (
  id uuid default gen_random_uuid() primary key,
  from_group integer not null check (from_group in (1,2,3)),
  to_group integer not null check (to_group in (1,2,3)),
  content text not null,
  created_at timestamptz default now()
);

-- Enable Realtime on this table
alter publication supabase_realtime add table messages;

-- Row Level Security: this is a trusted in-room game, allow anon read/write
alter table messages enable row level security;

create policy "anyone can read"  on messages for select using (true);
create policy "anyone can insert" on messages for insert with check (true);

-- Optional: a reset helper for the facilitator
-- delete from messages;
```

4. Go to **Project Settings ▸ API** (left sidebar, gear icon → API).
5. Copy your **Project URL** (looks like `https://xxxxxxxxxxxx.supabase.co`) and your **anon public** key (a long `eyJ...` JWT). Keep these handy for the next step.

---

## 2. Local Config

```bash
cp config.sample.js config.js
```

Open `config.js` and paste your credentials:

```js
const SUPABASE_URL     = "https://xxxxxxxxxxxx.supabase.co";
const SUPABASE_ANON_KEY = "eyJ...your-anon-key...";
```

> **Note:** `config.js` is listed in `.gitignore` so it won't be committed. The anon key is intentionally safe to expose to browsers (it is restricted by Row Level Security), but keeping it out of the repo is cleaner.

---

## 3. GitHub

```bash
git init          # if you haven't already
git add .
git commit -m "Initial CrazyLabs terminal"
```

Create a new repository on GitHub (public or private), then:

```bash
git remote add origin https://github.com/YOUR_USERNAME/crazylabs-terminal.git
git push -u origin main
```

---

## 4. Netlify Deploy

1. Go to [https://netlify.com](https://netlify.com), create a free account, and click **Add new site → Import an existing project**.
2. Connect to GitHub and select your repository.
3. Build settings:
   - **Build command:** *(leave empty)*
   - **Publish directory:** `.` (the repo root)
4. Click **Deploy site**. Netlify will publish `index.html` directly.

### Making `config.js` available on Netlify

Because `config.js` is gitignored, Netlify won't have it. You have two options:

**Option A — Commit `config.js` (recommended for simplicity)**
The anon key is browser-safe. Simply remove `config.js` from `.gitignore`, commit it, and push. Netlify will pick it up automatically on every deploy.

```bash
# Remove from .gitignore (delete the line), then:
git add config.js
git commit -m "Add Supabase config"
git push
```

**Option B — Inline credentials in `index.html`**
Open `index.html`, find the line:
```html
<script src="config.js" onerror="window.__configMissing=true"></script>
```
Replace it with:
```html
<script>
  const SUPABASE_URL      = "https://xxxxxxxxxxxx.supabase.co";
  const SUPABASE_ANON_KEY = "eyJ...";
</script>
```
This works but means re-editing `index.html` if you ever rotate credentials.

---

## 5. Running the Room

1. Open the Netlify URL on **three separate computers** (or browsers).
2. On each device, click the group button matching that device's role:
   - **Computer 1 → CLIENT (Group 1)**
   - **Computer 2 → API (Group 2)**
   - **Computer 3 → DATABASE (Group 3)**
3. Players construct pseudo-code messages and transmit them down and up the chain. Group 2 (API) is the only node that sees both halves of the chain.

**To reset between sessions**, run this in the Supabase SQL editor:

```sql
delete from messages;
```

---

## Gameplay Reference

| Direction | Visible to |
|-----------|-----------|
| Group 1 → Group 2 | Groups 1 and 2 |
| Group 2 → Group 1 | Groups 1 and 2 |
| Group 2 → Group 3 | Groups 2 and 3 |
| Group 3 → Group 2 | Groups 2 and 3 |

Groups 1 and 3 **never** see each other's messages. Only the API (Group 2) sees the full picture.

**Tip — Ctrl+Enter** sends a message from any text field.
