# CLAUDE.md — Setup Instructions for Claude Code

This file tells Claude Code how to set up this repository for deployment to GitHub Pages with Supabase as the backend.

## Project overview

This is a multi-project Software Project SOP workspace. Two HTML files:

- `index.html` — landing page listing all projects, with create/rename/delete actions
- `project.html` — the SOP editor (reads `?slug=…` from URL to know which project to load)

Both files are static. No build step. They use Supabase (loaded from CDN) for cloud persistence. They will be hosted on GitHub Pages.

**Data model:**
- Supabase table `sop_projects` — index of projects (slug, name, timestamps)
- Supabase table `sop_documents` — full SOP content stored as JSONB per slug
- Browser localStorage holds Supabase credentials (URL + anon key) per device

## What Claude Code should do

Work through the tasks below **in order**. After each task, briefly confirm what you did. If a step requires me (the user) to do something manual (create a GitHub account, log into Supabase, paste a key), pause and tell me clearly what to do, then wait for me to confirm before continuing.

### Task 1 — Verify the repository state

1. Run `ls -la` and confirm that `index.html` and `project.html` exist in the repo root.
2. Run `cat index.html | head -5` and `cat project.html | head -5` to confirm they're the expected HTML files (look for `<!DOCTYPE html>` and the page titles).
3. If files are missing or misnamed, stop and tell me.

### Task 2 — Initialize Git if not already done

1. Check whether this is already a git repo: `git status`
2. If it is not a git repo:
   - Run `git init`
   - Run `git branch -M main` (GitHub Pages defaults work best with `main`)
3. If it is already a git repo, run `git log --oneline -5` to show me the current state, then continue.

### Task 3 — Create a .gitignore

Create a `.gitignore` file in the repo root with the following contents:

```
# OS
.DS_Store
Thumbs.db

# Editors
.vscode/
.idea/
*.swp

# Logs
*.log

# Local environment / notes that should never be committed
.env
.env.local
secrets.txt
credentials.txt
*.local
notes-private.md
```

The point of `.gitignore` here is to make sure I never accidentally commit Supabase keys or local notes. **Never write credentials or secrets into any committed file.**

### Task 4 — Create a README.md

Create a `README.md` file in the repo root with the following structure (adapt the content to be clear and concise):

- Title: "Project SOP Workspace"
- One-paragraph description of what this is (single-page SOP template, Supabase-backed, GitHub Pages hosted)
- Section: "Live site" — placeholder for the GitHub Pages URL (I'll fill this in after deployment)
- Section: "How to use" — open the landing page, paste Supabase credentials once, create projects
- Section: "Setup" — link to CLAUDE.md
- Do NOT include any credentials, URLs from my Supabase project, or anon keys in the README.

### Task 5 — First commit

1. Run `git add .`
2. Run `git status` and show me what's about to be committed.
3. Run `git commit -m "Initial commit: SOP workspace HTML files + setup"`

### Task 6 — Pause and ask me for the GitHub repository URL

At this point you need the remote repository URL from me. Stop and ask:

> "Please create a new **public** GitHub repository (don't initialize it with a README, since we already have one). Then paste the SSH or HTTPS URL here (looks like `git@github.com:username/repo.git` or `https://github.com/username/repo.git`)."

Wait for me to provide the URL before proceeding.

### Task 7 — Connect to remote and push

Once I've given you the URL:

1. Run `git remote add origin <URL>`
2. Run `git remote -v` to confirm.
3. Run `git push -u origin main`
4. If the push fails because of authentication, tell me clearly what the error was (HTTPS needs a Personal Access Token; SSH needs an SSH key set up on GitHub). Don't guess — surface the error and let me fix it.

### Task 8 — Final report

Once the push succeeds, give me a short summary including:

1. The GitHub repo URL
2. The expected GitHub Pages URL (which will be `https://<username>.github.io/<repo-name>/` once Pages is enabled)
3. A reminder list of the manual steps I still need to do myself (enabling GitHub Pages in repo settings, running the Supabase SQL, pasting credentials in the live site). Reference the section below.

### Task 9 — Do NOT do these things

You cannot do these for me, and you should not try:

- Create my GitHub account or repository (requires browser/web UI)
- Enable GitHub Pages in repo settings (requires browser/web UI)
- Create my Supabase project (requires browser/web UI)
- Run SQL in the Supabase SQL editor (requires browser/web UI)
- Paste credentials into the deployed landing page (requires the live site loaded in a browser)
- Touch the HTML files unless I explicitly ask — they are the deliverable, not your scratch space

If I ask you to modify `index.html` or `project.html`, ask me what specific change I want before editing.

## Reference: the manual steps I need to do (for your awareness)

For context, here is what I will do outside Claude Code, in order:

1. Create a new public GitHub repository (no README, no .gitignore — empty)
2. Give you the repo URL so you can push
3. After you push, enable GitHub Pages in repo Settings → Pages → Source: `main` / root
4. Create a Supabase project at supabase.com
5. Run the SQL schema (provided below) in the Supabase SQL editor to create the tables
6. Copy my Supabase project URL and anon key from Project Settings → API
7. Open the live GitHub Pages URL and paste the credentials in the setup panel

## Reference: SQL schema for Supabase

This is the schema I will run in Supabase. Including it here for traceability — **do not run it from Claude Code, it has to be pasted in the Supabase web SQL editor**:

```sql
create table if not exists sop_documents (
  slug         text primary key,
  data         jsonb not null,
  updated_at   timestamptz not null default now()
);

create table if not exists sop_projects (
  slug         text primary key,
  name         text not null,
  created_at   timestamptz not null default now(),
  updated_at   timestamptz not null default now()
);

alter table sop_documents enable row level security;
alter table sop_projects  enable row level security;

create policy "anon can read"   on sop_documents for select using (true);
create policy "anon can insert" on sop_documents for insert with check (true);
create policy "anon can update" on sop_documents for update using (true);
create policy "anon can delete" on sop_documents for delete using (true);

create policy "anon can read projects"   on sop_projects for select using (true);
create policy "anon can insert projects" on sop_projects for insert with check (true);
create policy "anon can update projects" on sop_projects for update using (true);
create policy "anon can delete projects" on sop_projects for delete using (true);
```

## Security note

These RLS policies are intentionally permissive (anonymous access for all CRUD). This is fine for a personal project on a private/unguessable Pages URL with unguessable slugs, but **not** for sensitive data. If I later want proper auth, I'll ask you to help me wire up Supabase Auth with email login and tighten the policies to filter by `auth.uid()`.

## When in doubt

If anything is ambiguous or you'd be guessing, ask me. Don't fabricate URLs, usernames, or keys. Don't invent file changes. Don't run destructive commands (`rm -rf`, `git push --force`, etc.) without explicit confirmation.
