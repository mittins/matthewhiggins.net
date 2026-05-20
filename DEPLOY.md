# Deploying matthewhiggins.net

This is a one-page static site, hosted on **Cloudflare Pages** with the domain and DNS already on Cloudflare. Three things have to happen to get it live at `https://matthewhiggins.net`:

1. **Put the files on GitHub** so Cloudflare Pages has something to deploy.
2. **Connect the GitHub repo to Cloudflare Pages** and attach the custom domain.
3. **Turn on email forwarding** so `hello@matthewhiggins.net` reaches your Gmail.

Total time: ~15-20 minutes. The gig is tomorrow night so you've got plenty of buffer for DNS to settle.

---

## Step 1 — Push the site to GitHub

Even though Cloudflare Pages is doing the hosting, your code still lives on GitHub. Cloudflare reads from there on every push and rebuilds the site automatically.

### 1a. Create a new public repo

Go to https://github.com/new and create a repo under your account (`mittins`).

- **Repository name:** `matthewhiggins.net` (any name works, but matching the domain makes future-you grateful)
- **Visibility:** Public is fine (Cloudflare Pages works with private repos too, but public keeps things simple)
- **Don't** initialize with a README, .gitignore, or license — you want it empty so you can upload your existing files cleanly.

### 1b. Upload the files

Two ways. Pick whichever feels easier; the result is identical.

**Option A — Drag and drop in the browser (no terminal needed).**

1. On the new empty repo page, click **"uploading an existing file"**.
2. Drag `index.html` and `DEPLOY.md` from this folder into the upload area.
3. Scroll down, write a commit message like "initial commit", and click **Commit changes**.

**Option B — git from the command line.**

```bash
cd "/Users/Shared/Agentic_Development/matthewhiggins_net_website/matthewhiggins.net"
git init
git add .
git commit -m "initial commit"
git branch -M main
git remote add origin git@github.com:mittins/matthewhiggins.net.git
git push -u origin main
```

(Swap `git@github.com:...` for the HTTPS URL if you don't have SSH keys set up. GitHub shows both on the empty-repo page.)

---

## Step 2 — Connect Cloudflare Pages to the repo

### 2a. Create the Pages project

1. Log in to Cloudflare and open **Workers & Pages** in the left sidebar.
2. Click **Create application → Pages → Connect to Git**.
3. Authorize Cloudflare to access your GitHub account (one-time prompt the first time you use this).
4. Select the `matthewhiggins.net` repository you just created.
5. Click **Begin setup**.

### 2b. Build settings

This page is plain HTML — no build step needed. Use these settings:

| Field                        | Value                          |
| ---------------------------- | ------------------------------ |
| Project name                 | `matthewhiggins-net` (any name) |
| Production branch            | `main`                         |
| Framework preset             | **None**                       |
| Build command                | *(leave empty)*                |
| Build output directory       | `/`                            |
| Root directory               | *(leave empty)*                |

Click **Save and Deploy**.

Cloudflare will run a deployment in ~30 seconds. When it finishes, your site will be live at a Cloudflare URL like `https://matthewhiggins-net.pages.dev`. Click through and confirm the page loads correctly — this is your safety check before pointing the real domain at it.

### 2c. Attach the custom domain

1. From the Pages project dashboard, go to **Custom domains → Set up a custom domain**.
2. Enter `matthewhiggins.net` and click **Continue**.
3. Cloudflare will detect that you own the domain (since DNS is already on Cloudflare) and offer to add the necessary CNAME record automatically. Click **Activate domain**.
4. Repeat for `www.matthewhiggins.net` if you want `www` to redirect to the apex.

Cloudflare handles TLS automatically — no certificate to provision manually. Within a minute or two, `https://matthewhiggins.net` should resolve to your page.

**What just happened under the hood:** Cloudflare added a CNAME record on the apex of your domain pointing to the `.pages.dev` URL. Because Cloudflare manages your DNS, this happens with a click instead of the four-A-record dance you'd need with GitHub Pages.

---

## Step 3 — Set up `hello@matthewhiggins.net` forwarding

Cloudflare's Email Routing is free and handles this without you needing a full mailbox.

1. In Cloudflare, with `matthewhiggins.net` selected, open **Email → Email Routing**.
2. Click **Get started** (or **Enable Email Routing**).
3. **Destination addresses:** Add `matthew.higgins@gmail.com`. Cloudflare will email you a verification link — click it from Gmail to confirm.
4. **Routing rules:** Create a custom address.
   - Custom address: `hello@matthewhiggins.net`
   - Action: Send to → `matthew.higgins@gmail.com`
   - Save.
5. Cloudflare may ask to automatically add MX and TXT records for you — say yes. (Email Routing's DNS records sit alongside the Pages CNAME — they don't conflict.)
6. Send yourself a test email from another account to `hello@matthewhiggins.net`. It should land in Gmail within a minute.

**Replying as `hello@`.** Forwarding sends mail into Gmail but, by default, replies will come from your Gmail address. If you want to reply *as* `hello@matthewhiggins.net`, you need either Gmail's "Send mail as" feature with an SMTP server, or a real mailbox provider (Google Workspace, Fastmail, etc.). That's a Phase-2 problem — not blocking the gig.

---

## What's in this folder

- **`index.html`** — the whole site, in one file. HTML structure at the top, CSS in a `<style>` block, no JavaScript. Light/dark theme follows the visitor's system preference via `prefers-color-scheme`.
- **`DEPLOY.md`** — this file. Setup instructions. Not served to visitors (Cloudflare Pages won't render `.md` files unless you opt in to a static-site framework, which we haven't).

## Editing later

Any change to `index.html` pushed to the `main` branch triggers Cloudflare to rebuild and redeploy automatically (typically within 30–60 seconds). To preview locally before pushing, open `index.html` directly in a browser — there's no build step.

## What's intentionally missing

- **Instagram link.** You mentioned a new music-only IG handle is still being chosen. Better to launch with no link than a "coming soon" placeholder — true to the anti-slop ethos. Send the handle once locked and we'll add it in a one-line edit.
- **A favicon.** Browsers will show a generic icon in tabs until we add one. Easy follow-up — a single 32×32 PNG saved as `favicon.png` with a `<link rel="icon">` tag.
- **An About / Music / Photography / Work split.** That's the Phase 2 build, after the v0 page is live and the gig is done.

## Why Cloudflare Pages over the alternatives

For posterity / future you, the rationale for this choice:

- **Vs. GitHub Pages:** Cloudflare Pages has unlimited bandwidth (GH Pages has a soft 100 GB/month limit), keeps everything under one Cloudflare dashboard alongside your DNS and email, and attaches custom domains with one click instead of four manual A records.
- **Vs. Hostinger / shared hosting:** Static-on-CDN is fundamentally faster and cheaper for this kind of site. Shared hosting is the right call only when you need WordPress, PHP, or MySQL on the server side.
- **Vs. Railway:** Railway is for running application processes (Node servers, Python apps, databases). Hosting static files there means paying compute for what a CDN serves free.
