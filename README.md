# Terra Tour

Landing page and lightweight Node.js backend for Terra Tour Lampung. Requires Node.js 18 or newer; there are no external npm dependencies.

## Run locally

```powershell
npm start
```

Open `http://localhost:3000`. The WhatsApp number provided by Terra Tour is configured by default. To override it, or to connect a separately hosted GitHub Pages frontend, set these values before starting the server:

```powershell
$env:WHATSAPP_NUMBER="628127289020"
npm start
```

The public news form requires a title, content, and exactly 3 different comma-separated keywords. There is no admin password: anyone who can visit the published form can submit a news article. News is stored in `data/articles.jsonl`; new inquiries are appended to `data/inquiries.jsonl`. These local data files are ignored by Git. Set `DATA_DIR` to a persistent storage path when deploying the backend.

The photos shared through Google Drive are included in `assets/` and loaded from the website itself, so visitors do not depend on Google Drive links. Their destination mapping is documented in [assets/README.md](assets/README.md). Package cards intentionally do not display prices.

## GitHub

Create an empty repository on GitHub, then from this folder run `git init`, `git add .`, `git commit -m "Build Terra Tour landing page"`, `git branch -M main`, `git remote add origin <repository-url>`, then `git push -u origin main`. GitHub Actions runs backend tests on every push and pull request.

GitHub Pages can host the static website, but cannot run `server.js` or save news posts. The `render.yaml` file configures the included Node.js backend for Render and provisions persistent storage. Deploy the backend separately:

1. Upload all changed files, including `render.yaml` and the `assets/` folder, to your GitHub repository.
2. Sign in to Render, choose **New → Blueprint**, connect the GitHub repository, and deploy the service described in `render.yaml`. The blueprint requests a paid Starter web service and a 1 GB persistent disk so published articles are not lost when the service restarts. Review Render's current pricing before confirming deployment.
3. During deployment, enter `FRONTEND_ORIGIN` as the website's exact origin only—for example `https://username.github.io` (not a page path such as `/terra-tour/`). After deployment, copy the backend URL Render gives you, for example `https://terra-tour-backend.onrender.com`.
4. Open `<backend-url>/api/health`. It must show `{"status":"ok"}`. If it does not, the backend deployment is not ready yet.
5. In this repository's `index.html`, set the `api-base` meta tag to that backend URL, for example `<meta name="api-base" content="https://terra-tour-backend.onrender.com" />`. Do not add `/api` at the end.
6. Commit and push this `index.html` change. Wait for GitHub Pages to finish deploying, then reload the website and publish a test article. The article list and `https://<backend-url>/api/articles` should show the saved article.

The publish endpoint is intentionally public with no password, so anyone can post and the endpoint may be abused for spam. Do not use it for private/internal news.

### Upload changes to the GitHub repository

From this project folder, open a terminal and run:

```powershell
git status
git add index.html app.js styles.css server.js test/server.test.js README.md assets/README.md render.yaml assets
git commit -m "Connect news publishing backend and destination photos"
git push
```

If `git push` says there is no upstream branch, use `git push -u origin main` (replace `main` with your branch name if it is different). You can do the same in VS Code by opening **Source Control**, reviewing the changed files, entering a commit message, selecting **Commit**, then **Sync Changes** or **Push**. After pushing, open the repository's **Actions** tab and wait for the checks to finish. If GitHub Pages is already enabled, its deployment will update the live site automatically.

If the form displays “mengirim halaman, bukan JSON,” the frontend is reaching a website page instead of the API. Check that `api-base` is the backend's Render URL, that `/api/health` returns JSON, and that the backend's `FRONTEND_ORIGIN` matches the published website origin exactly.

## API

- `GET /api/health` returns server health.
- `GET /api/config` returns the configured public WhatsApp number.
- `POST /api/contact` validates and saves an inquiry.
- `GET /api/articles` returns published news.
- `POST /api/articles` publishes news without authentication; send `title`, `content`, and exactly three comma-separated `keywords` as JSON.
