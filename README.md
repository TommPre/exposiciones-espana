# 🎨 Exposiciones de Arte en España — Auto-updating Website

A website that lists art exhibitions in Spain (museums, galleries, cultural centres, fairs)
and **automatically refreshes itself on the 1st of every month** using AI.

---

## How it works

```
Every 1st of the month
       │
       ▼
GitHub Actions wakes up
       │
       ▼
Runs update_exhibitions.py
  ├─ Removes closed exhibitions from exhibitions.json
  ├─ Asks Claude AI to search the web for new ones
  └─ Rebuilds index.html with fresh data
       │
       ▼
Commits & pushes to GitHub
       │
       ▼
GitHub Pages serves the updated site live 🌍
```

**Cost:** roughly $0.01–0.05 per month (uses the cheapest Claude model).

---

## One-time setup (step by step)

### Step 1 — Create a GitHub account
1. Go to **github.com** and click **Sign up**
2. Choose a free account

### Step 2 — Create a new repository
1. Click the **+** icon (top right) → **New repository**
2. Name it `exposiciones-espana` (or anything you like)
3. Set it to **Public** *(required for free GitHub Pages)*
4. Click **Create repository**

### Step 3 — Upload the files
1. In your new empty repo, click **uploading an existing file**
2. Drag and drop ALL these files at once:
   - `index.html`
   - `exhibitions.json`
   - `template.html`
   - `update_exhibitions.py`
   - The folder `.github/workflows/update.yml`
     *(to upload the workflow, you may need to create the folder structure manually — see tip below)*
3. Click **Commit changes**

> **Tip for the workflow file:** After uploading the other files, click **Add file → Create new file**.
> In the filename box type `.github/workflows/update.yml` (GitHub will create the folders automatically).
> Paste the contents of `update.yml` and commit.

### Step 4 — Get a Claude API key
1. Go to **console.anthropic.com** and sign up / log in
2. Click **API Keys** in the left menu → **Create Key**
3. Copy the key (it starts with `sk-ant-...`) — save it somewhere safe, you'll only see it once

> **Cost note:** Each monthly run costs roughly **$0.01–0.05**.
> Anthropic gives new accounts $5 in free credits — that's years of monthly updates.

### Step 5 — Add the API key as a GitHub Secret
This lets GitHub Actions use the key without it being visible to anyone.

1. In your GitHub repository, click **Settings** (top menu)
2. In the left sidebar → **Secrets and variables** → **Actions**
3. Click **New repository secret**
4. Name: `ANTHROPIC_API_KEY`
5. Value: paste your key (`sk-ant-...`)
6. Click **Add secret**

### Step 6 — Enable GitHub Pages
1. Still in **Settings**, scroll down to **Pages** in the left sidebar
2. Under **Source**, select **Deploy from a branch**
3. Branch: `main` · Folder: `/ (root)`
4. Click **Save**
5. After ~1 minute, GitHub will show you your live URL:
   `https://YOUR-USERNAME.github.io/exposiciones-espana/`

### Step 7 — Test it manually
You don't have to wait until the 1st of the month!

1. In your repo, click the **Actions** tab
2. Click **Monthly Exhibitions Update** in the left list
3. Click **Run workflow** → **Run workflow** (green button)
4. Watch it run — it takes about 30–60 seconds
5. After it finishes, visit your live URL and see the updated site ✅

---

## Monthly maintenance
**None required.** Everything is automatic.

But if you want to add or remove an exhibition manually, just edit `exhibitions.json`
directly on GitHub (click the file → pencil icon → edit → commit).

---

## Files in this project

| File | What it does |
|------|-------------|
| `index.html` | The website (auto-generated, don't edit manually) |
| `template.html` | The HTML skeleton — edit this if you want to change the design |
| `exhibitions.json` | The data — all current exhibitions |
| `update_exhibitions.py` | The script that calls Claude and rebuilds the site |
| `.github/workflows/update.yml` | Tells GitHub *when* and *how* to run the script |

---

## Troubleshooting

**The workflow failed** → Click the failed run in the Actions tab to see the error log.
Most common cause: the `ANTHROPIC_API_KEY` secret is missing or wrong.

**The site shows old data** → Trigger a manual run (Step 7 above).

**I want to update more often** → Edit the `cron` line in `update.yml`.
For example `"0 8 1,15 * *"` runs on the 1st AND 15th of every month.

**I want to change the design** → Edit `template.html` and commit. The next
run of the script will pick up your changes automatically.
