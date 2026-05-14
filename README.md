# Project SOP Workspace

A static, single-page SOP (Standard Operating Procedure) template workspace. Projects are stored in Supabase and the app is hosted on GitHub Pages — no build step, no server required. Open the landing page, enter your Supabase credentials once, and start creating projects with structured SOP documents.

## Live site

> URL will be filled in after GitHub Pages is enabled.
> Expected format: `https://<username>.github.io/<repo-name>/`

## How to use

1. Open the live GitHub Pages URL.
2. On first load, enter your Supabase project URL and anon key in the setup panel. These are saved in your browser's localStorage — they never leave your device.
3. From the landing page (`index.html`) you can create, rename, and delete projects.
4. Click a project to open the SOP editor (`project.html`), where you can fill out and save structured SOP content.

## Setup

See [CLAUDE.md](CLAUDE.md) for full setup instructions, including the Supabase SQL schema and GitHub Pages configuration steps.
