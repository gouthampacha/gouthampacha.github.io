# Deploying to Cloudflare Pages

Step-by-step guide to deploy this Hugo site on Cloudflare Pages and migrate your domain from GitHub Pages.

---

## Part 1: Deploy on Cloudflare Pages

### 1.1 Create a Cloudflare account (if you don't have one)

1. Go to https://dash.cloudflare.com/sign-up
2. Sign up with your email — the free plan is all you need

### 1.2 Connect your GitHub repo

1. In the Cloudflare dashboard, go to **Workers & Pages** in the left sidebar
2. Click **Create** > **Pages** > **Connect to Git**
3. Authenticate with GitHub if prompted
4. Select the repository: `gouthampacha/gouthampacha.github.io`
5. Click **Begin setup**

### 1.3 Configure build settings

Set the following:

| Setting | Value |
|---|---|
| **Project name** | `gouthampacha` (or whatever you prefer) |
| **Production branch** | `master` |
| **Framework preset** | Hugo |
| **Build command** | `hugo` |
| **Build output directory** | `public` |

### 1.4 Set environment variables

Click **Environment variables** and add:

| Variable | Value |
|---|---|
| `HUGO_VERSION` | `0.156.0` |

This ensures Cloudflare uses the same Hugo version you develop with locally.

### 1.5 Deploy

Click **Save and Deploy**. Cloudflare will:
1. Clone your repo
2. Run `hugo` to build the site
3. Deploy the `public/` directory to their CDN
4. Give you a URL like `https://gouthampacha.pages.dev`

Visit that URL to verify your site works. Every future `git push` to `master` will trigger an automatic redeploy.

---

## Part 2: Migrate your domain from GitHub Pages

You have two options depending on whether you want Cloudflare to manage your DNS or not.

### Option A: Transfer DNS to Cloudflare (Recommended)

This gives you the best performance (Cloudflare CDN, caching, DDoS protection) and simplest setup.

#### Step 1: Add your domain to Cloudflare

1. In the Cloudflare dashboard, click **Add a site** (or go to **Websites** > **Add a site**)
2. Enter `gouthampacha.com`
3. Select the **Free** plan
4. Cloudflare will scan your existing DNS records

#### Step 2: Update nameservers at your domain registrar

1. Cloudflare will give you two nameservers (e.g., `ada.ns.cloudflare.com`, `bob.ns.cloudflare.com`)
2. Go to your domain registrar (wherever you bought `gouthampacha.com` — GoDaddy, Namecheap, Google Domains, etc.)
3. Find the **Nameservers** setting and replace the current nameservers with the two Cloudflare ones
4. Save. Propagation takes up to 24-48 hours (usually much faster)

#### Step 3: Connect domain to Cloudflare Pages

1. Go to **Workers & Pages** > your project > **Custom domains**
2. Click **Set up a custom domain**
3. Enter `gouthampacha.com`
4. Cloudflare will automatically create the DNS record for you
5. Repeat for `www.gouthampacha.com` if desired

#### Step 4: Verify SSL/TLS

1. Go to **SSL/TLS** in the dashboard for your domain
2. Set encryption mode to **Full (strict)**
3. SSL certificates are provisioned automatically — no action needed

### Option B: Keep DNS at your registrar (CNAME approach)

If you don't want to move your nameservers to Cloudflare:

1. Go to your domain registrar's DNS settings
2. Remove the old GitHub Pages records (A records pointing to `185.199.108.153`, etc., or CNAME to `gouthampacha.github.io`)
3. Add a `CNAME` record:
   - **Name:** `www`
   - **Value:** `gouthampacha.pages.dev`
4. For the root domain (`gouthampacha.com`), you'll need your registrar to support CNAME flattening or ALIAS records. If not, Option A is simpler.

---

## Part 3: Clean up GitHub Pages

Once your Cloudflare Pages deployment is working and your domain is pointed there:

1. **Remove the CNAME file** from this repo (it's a GitHub Pages artifact):
   ```bash
   git rm CNAME
   git commit -m "Remove GitHub Pages CNAME — now on Cloudflare Pages"
   git push
   ```

2. **Disable GitHub Pages** (optional):
   - Go to your repo on GitHub > **Settings** > **Pages**
   - Set source to **None** or leave it — it won't conflict since the domain now points to Cloudflare

---

## Part 4: Day-to-day workflow

### Writing a new blog post

```bash
# Create a new post (creates a draft)
hugo new content blog/my-post-title/index.md

# Edit the file in your editor
# Set draft: false when ready to publish

# Preview locally
hugo server -D    # -D includes drafts

# Publish
git add content/blog/my-post-title/
git commit -m "Add blog post: My Post Title"
git push
# Cloudflare Pages auto-deploys within ~30 seconds
```

### Local development

```bash
# Start dev server with drafts and live reload
hugo server -D

# Build the site (output goes to public/)
hugo

# Build with drafts included
hugo -D
```

### Updating the Hugo version

If you upgrade Hugo locally, update the `HUGO_VERSION` environment variable in your Cloudflare Pages project settings to match.

---

## Troubleshooting

| Issue | Fix |
|---|---|
| Build fails on Cloudflare | Check `HUGO_VERSION` env var matches your local version |
| CSS looks wrong | Ensure git submodule is committed (`git submodule update --init`) |
| Domain not working | Wait for DNS propagation (up to 48h). Check with `dig gouthampacha.com` |
| Old GitHub Pages still serving | Disable Pages in GitHub repo settings, or wait for DNS to propagate |
| "Page not found" on subpages | Make sure `baseURL` in `config/_default/hugo.toml` matches your domain |
