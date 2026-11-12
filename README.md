# luislopezcreates.com: GitHub + Cloudflare setup

## Your repo must look exactly like this
`wrangler.toml` has to sit at the **top level** of the GitHub repo, with `public/` right next to it.
Most setup errors come from an extra folder wrapped around these files.

```
your-repo/
├── wrangler.toml        <- Cloudflare config (must be at the top level)
├── worker.js            <- the small backend that makes editing possible
├── package.json
├── package-lock.json
├── .gitignore
├── README.md
└── public/              <- the website itself
    ├── index.html
    ├── styles.css
    ├── app.js
    ├── editor.js
    ├── robots.txt
    ├── sitemap.xml
    ├── _headers
    └── assets/          <- images and animations (31 files)
```

## Step 1: Put the files on GitHub
1. Unzip `luislopezcreates-repo.zip` on your computer. Open the unzipped folder. You should see `wrangler.toml` and `public` right there.
2. GitHub, **New repository** (name it anything, for example `luislopezcreates`), then **uploading an existing file**.
3. Drag **the contents** of the unzipped folder in (select everything inside it and drag, not the folder itself, and not the .zip).
   Make sure the `public` folder uploads with its `assets` subfolder. Commit.
4. Check on GitHub: when you open the repo you should see `wrangler.toml` on the first page, not inside another folder.
   (Dragging folders into GitHub's web uploader works in Chrome, Edge and Safari. If a folder won't upload, use GitHub Desktop.)

## Step 2: Connect the repo to Cloudflare
1. Cloudflare dashboard, **Workers & Pages**, **Create**, **Import a repository**. Connect GitHub and pick your repo.
2. **Use your existing Worker `my-dynamic-site`** (your domain already points to it), or import with the project name `my-dynamic-site`. The name must match the `name` line in `wrangler.toml`.
   (If you use a different name, change `name = "my-dynamic-site"` in `wrangler.toml` to match it exactly.)
3. Build settings: **Root directory** `/` (leave it blank or `/`), **Build command** leave empty, **Deploy command** `npx wrangler deploy`.
4. Save and deploy. Every time you push a change to GitHub, Cloudflare redeploys automatically.

**If your domain shows only "Hello world":** your Worker `my-dynamic-site` still has Cloudflare's starter code, which means the GitHub deploy hasn't reached it yet.
Open the Worker, **Deployments** tab, and look at the latest one. In **Settings, Builds**, check that it is connected to your repo and branch (usually `main`),
then push any small change (or click **Retry build**). When the deploy succeeds, the Deployments tab shows a new version and the domain serves your site.

## Step 3: Set your admin password (this makes editing work)
Worker `my-dynamic-site`, **Settings**, **Variables and Secrets**, **Add**, type **Secret**, name `ADMIN_TOKEN`, value = a long random password.
Save it in a password manager. It's needed to publish edits. Secrets stay put when GitHub redeploys.

## Step 4: Attach your domain
First, in Cloudflare, `luislopezcreates.com` must show **Active** (nameservers pointed at Cloudflare) and have no old A, AAAA or CNAME records
for `@` or `www` under DNS, Records.
Then: Worker, **Settings**, **Domains & Routes**, **Add**, **Custom domain**, enter `luislopezcreates.com`. Repeat for `www.luislopezcreates.com`.
Cloudflare creates the DNS records and HTTPS certificate. `www` forwards to the main address. Turn on SSL/TLS, Edge Certificates, **Always Use HTTPS**.

## Step 5: Edit your live site
Go to **https://luislopezcreates.com/?edit**, click **Edit site**, make changes, then open the **Data** tab, paste your admin token, and press
**Publish changes**. Visitors never see the editor buttons. Data tab, **Download data (.json)**, makes a backup.

## Videos (optional)
The editor accepts MP4, WebM, MOV and MKV. Without R2 storage, videos up to about 2.5 MB can be added, or paste a link to a hosted video.
For real video uploads (up to 100 MB each): turn on R2 in the Cloudflare dashboard (needs a payment method; the free 10 GB is plenty), create a bucket named
`portfolio-media`, remove the `# ` from the three `r2_buckets` lines at the bottom of `wrangler.toml`, and push to GitHub.
Best format: MP4 with H.264 video and AAC audio. Most browsers can't play MKV; convert first
(`ffmpeg -i in.mkv -c:v libx264 -c:a aac out.mp4`, or the free HandBrake app).

## If something goes wrong
| What you see | What it means / fix |
|---|---|
| Build log: *can't find wrangler.toml* or *Missing entry-point* | The files are nested inside an extra folder in GitHub. `wrangler.toml` and `worker.js` must be at the top of the repo (see the tree above). |
| *assets directory ./public does not exist* | The `public` folder didn't upload or is nested. It must sit next to `wrangler.toml`. |
| *name in wrangler.toml does not match the Worker name* | Make the project name in Cloudflare and the `name` line in `wrangler.toml` identical. |
| *KV namespace … not found* | The KV id in `wrangler.toml` belongs to a different Cloudflare account. Workers & Pages, KV, create a namespace, paste its id over the one in `wrangler.toml`. |
| Domain shows an error page | The domain isn't Active yet, or an old DNS record is in the way (Step 4). |
| "Could not publish: wrong token" | The token must match the `ADMIN_TOKEN` secret exactly. Edit the secret in the dashboard to change it. |
| Edits don't appear for visitors | You saved a draft but didn't press **Publish changes** in the Data tab. |
| Old content after a deploy | Hard refresh (Cmd/Ctrl + Shift + R). |

## Deploying from your own computer instead (optional)
Install Node.js (nodejs.org), then in this folder run `npm install`, `npm run login`, `npm run secret`, `npm run deploy`.
`npm run check` tests the project without deploying.
