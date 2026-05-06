You are executing the `/github-push` command. The user has provided a GitHub repository URL:

**Input:** $ARGUMENTS

---

## Step 0 — Parse the URL

From `$ARGUMENTS`, extract:
- `REPO_URL` = the full URL as-is (e.g. `https://github.com/mseow0001/lumiere-bridal.git`)
- `OWNER` = the username/org after `github.com/` (e.g. `mseow0001`)
- `REPO` = the repository name, stripping any `.git` suffix (e.g. `lumiere-bridal`)
- `PAGES_URL` = `https://OWNER.github.io/REPO`

**Self-verify before continuing:** State the four parsed values out loud so the user can confirm they are correct before you proceed.

---

## Phase 1 — Push to GitHub

### 1.1 Check for Git
Run `git --version`.

- If it succeeds, continue.
- If it fails (not recognized), run:
  ```
  winget install --id Git.Git -e --source winget
  ```
  After installation, use the full path `"C:\Program Files\Git\bin\git.exe"` for all git commands in this session since PATH won't refresh in the current shell.

### 1.2 Initialize and Push
Check if a `.git` folder already exists in the project directory. If it does, skip `git init`.

Run the following in `c:\Users\mseow\claudeclass`:
```
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin REPO_URL
git push -u origin main
```

- If `git remote add origin` fails because the remote already exists, run `git remote set-url origin REPO_URL` instead.
- If `git push` fails with a non-fast-forward error, ask the user before using `git push --force-with-lease`.
- If `git push` fails with an authentication error, guide the user to generate a GitHub Personal Access Token (PAT) at GitHub → Settings → Developer settings → Personal access tokens, then use it as the password when Git prompts for credentials.

---

## Phase 2 — Generate README.md

### 2.1 Inspect the project
Read `index.html` to identify the tech stack. For this project it is: HTML5, CSS3 (custom properties, Grid, Flexbox), Vanilla JavaScript (ES6+, IntersectionObserver, Fetch API), Google Fonts (Playfair Display + Lato), FormSubmit.co.

### 2.2 Build the file tree
Run `Get-ChildItem -Recurse c:\Users\mseow\claudeclass | Select-Object FullName` and format the output as a tree structure for the README. Include `.github/workflows/pages.yml` in the tree (it will exist after Phase 3).

### 2.3 Take a screenshot
**Attempt with Playwright MCP:**
1. Call `mcp__playwright__playwright_navigate` with `url` = `PAGES_URL`
2. Wait 3 seconds for the hero background image (loaded from Unsplash) to render
3. Call `mcp__playwright__playwright_screenshot` with `name` = `"preview"` and `fullPage` = `false` to capture the above-the-fold hero at 1280×720
4. Save the result to `c:\Users\mseow\claudeclass\screenshot.png`

**Fallback — Pages not yet live:** If `PAGES_URL` returns a 404 (GitHub Pages takes ~90 seconds after first push), use `url` = `"file:///C:/Users/mseow/claudeclass/index.html"` instead.

**Fallback — Playwright MCP unavailable:** Insert `![Lumière Preview](screenshot.png)` in the README with a `<!-- Replace screenshot.png with an actual screenshot of the live site -->` comment, and skip saving a file.

### 2.4 Write README.md
Write the following to `c:\Users\mseow\claudeclass\README.md`:

---

```markdown
# Lumière — Bridal Photography

> Timeless, romantic portraits that carry the warmth of your most treasured moments — forever.

![HTML5](https://img.shields.io/badge/HTML5-E34F26?style=flat&logo=html5&logoColor=white)
![CSS3](https://img.shields.io/badge/CSS3-1572B6?style=flat&logo=css3&logoColor=white)
![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?style=flat&logo=javascript&logoColor=black)
![Google Fonts](https://img.shields.io/badge/Google_Fonts-4285F4?style=flat&logo=google&logoColor=white)
![GitHub Pages](https://img.shields.io/badge/GitHub_Pages-222222?style=flat&logo=github&logoColor=white)
![Deploy to GitHub Pages](https://github.com/OWNER/REPO/actions/workflows/pages.yml/badge.svg)

![Lumière Preview](screenshot.png)

## Live Demo
[View Live Site](PAGES_URL)

## About the Project
Lumière is a single-page bridal photography website for Sofia Lumière — a wedding photographer based in Singapore. The site features a full-viewport hero, about section, three-tier pricing packages, a masonry-style photo gallery, a booking form with live validation (powered by FormSubmit.co), and a dark-themed contact footer. The design uses a blush and gold color palette with Playfair Display serif headings and Lato body text. No frameworks, no build tools — pure HTML, CSS, and JavaScript in a single file.

## File Structure
\`\`\`
[insert Get-ChildItem tree output here]
\`\`\`

## How to Use

### View Locally
Open `index.html` in any modern browser — no server or build step required.

### Deploy
The repository includes a GitHub Actions workflow (`.github/workflows/pages.yml`) that automatically deploys the site to GitHub Pages on every push to `main`. Enable GitHub Pages in your repo settings under **Settings → Pages → Source → GitHub Actions**.

### Customize
All CSS is in the `<style>` block and all JavaScript is in the `<script>` block at the bottom of `index.html`. Edit directly — no compilation needed.

## License
MIT
```

---

Substitute `OWNER`, `REPO`, and `PAGES_URL` with the actual parsed values. Insert the real file tree in the File Structure section.

### 2.5 Commit README
```
git add README.md screenshot.png
git commit -m "docs: add README with badges, screenshot, and project overview"
git push
```

If no `screenshot.png` exists (Playwright unavailable), omit it from `git add`.

---

## Phase 3 — GitHub Pages via GitHub Actions

### 3.1 Create the workflow file
Create the directory `.github/workflows/` inside `c:\Users\mseow\claudeclass\` if it does not exist.

Write the following to `c:\Users\mseow\claudeclass\.github\workflows\pages.yml`:

```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches:
      - main
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

jobs:
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Pages
        uses: actions/configure-pages@v5

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: '.'

      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

### 3.2 Commit and push
```
git add .github/workflows/pages.yml
git commit -m "ci: add GitHub Actions workflow for GitHub Pages deployment"
git push
```

### 3.3 Inform the user
Tell the user: "The workflow has been pushed. GitHub Pages will be live at **PAGES_URL** in approximately 60–120 seconds. You can monitor the deployment at `https://github.com/OWNER/REPO/actions`."

Also note: GitHub Pages must be enabled in the repo. Go to **Settings → Pages → Source** and select **GitHub Actions**. Phase 4 will attempt to enable it via the API automatically.

---

## Phase 4 — Update GitHub Repo About

### 4.1 Check for GitHub CLI
Run `gh --version`.

**If `gh` is available:**
```
gh repo edit OWNER/REPO --description "Lumière — A timeless single-page bridal photography website. Built with pure HTML, CSS & JavaScript." --homepage "PAGES_URL"
```

**If `gh` is not available — use PowerShell REST API:**

Tell the user: "I need a GitHub Personal Access Token with `repo` scope (or `public_repo` for public repos) to update the repository settings. Generate one at GitHub → Settings → Developer settings → Personal access tokens → Tokens (classic), then paste it below."

Ask the user to provide their PAT, then run:

```powershell
$token = "USER_PROVIDED_PAT"
$headers = @{
  Authorization = "Bearer $token"
  Accept = "application/vnd.github+json"
  "X-GitHub-Api-Version" = "2022-11-28"
}

Invoke-RestMethod `
  -Uri "https://api.github.com/repos/OWNER/REPO" `
  -Method Patch `
  -Headers $headers `
  -ContentType "application/json" `
  -Body (ConvertTo-Json @{
    description = "Lumière — A timeless single-page bridal photography website. Built with pure HTML, CSS & JavaScript."
    homepage    = "PAGES_URL"
  })

Invoke-RestMethod `
  -Uri "https://api.github.com/repos/OWNER/REPO/pages" `
  -Method Post `
  -Headers $headers `
  -ContentType "application/json" `
  -Body (ConvertTo-Json @{ build_type = "workflow" })
```

- If the Pages POST returns HTTP 409, it means Pages is already enabled — ignore that error and continue.

---

## Done

Once all four phases complete, confirm the following to the user:

1. **Code pushed** — `https://github.com/OWNER/REPO`
2. **README live** — visible on the repo homepage with badges and screenshot
3. **GitHub Actions running** — `https://github.com/OWNER/REPO/actions`
4. **Live site** — `PAGES_URL` (active within ~90 seconds of the workflow completing)
5. **Repo About updated** — description and homepage URL set in the repo's About panel
