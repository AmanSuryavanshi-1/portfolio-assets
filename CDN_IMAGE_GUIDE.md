# Content Delivery Network (CDN) & Asset Management Architecture

## Architecture Overview

This project utilizes a high-availability serverless image hosting architecture combining **GitHub Repository** for storage and **jsDelivr** for global content delivery. This setup replaces traditional paid asset management services (e.g., Cloudinary, S3) with a zero-cost, high-performance edge caching solution.

### System Components

1.  **Storage Layer (Origin):**
    *   **Repository:** `https://github.com/AmanSuryavanshi-1/portfolio-assets`
    *   **Role:** Acts as the single source of truth for all static assets.
    *   **Access:** Public read access required for CDN origin fetching.

2.  **Delivery Layer (Edge):**
    *   **Provider:** jsDelivr (Multi-CDN strategy using Cloudflare, Fastly, etc.)
    *   **Mechanism:** Automatic caching and mirroring of GitHub releases/branches.
    *   **Endpoint Format:** `https://cdn.jsdelivr.net/gh/{user}/{repo}@{version}/{path}`

## Cost & Performance Analysis

| Metric | Enterprise/Paid Services (Cloudinary/AWS) | Current Architecture (GitHub + jsDelivr) |
| :--- | :--- | :--- |
| **Operational Cost** | Variable (Bandwidth/Storage dependent) | **Zero (Free Tier / Open Source)** |
| **Bandwidth Limit** | Hard caps or overage fees | **Unlimited** (for valid FOSS usage) |
| **Integration** | SDKs, API Keys, Authentication | **Standard Git Workflow** |
| **Performance** | Tier-dependent | **Global Edge Network (Enterprise Grade)** |

---

## Deployment Workflow

### Adding New Assets

1.  Place the properly optimized image (WebP preferred) in the local repository directory.
2.  Commit and push to the `main` branch:
    ```bash
    git add .
    git commit -m "feat(assets): add new project screenshots"
    git push origin main
    ```
3.  Reference the asset in code using the CDN URL pattern:
    ```typescript
    `https://cdn.jsdelivr.net/gh/AmanSuryavanshi-1/portfolio-assets@main/${DIRECTORY}/${FILENAME}.webp`
    ```

### Updating Existing Assets

When modifying an existing asset without changing the filename:
1.  Overwrite the specific file locally.
2.  Push changes to GitHub.
3.  **Note:** CDN propagation may delay visibility due to edge caching policies. See *Cache Management* below.

---

## Cache Management & Troubleshooting

> [!IMPORTANT]
> There are **TWO separate caching layers** when using CDN images in GitHub documentation. Both must be addressed for updates to appear.

### TL;DR — When Do I Need Cache-Busting?

| Where Image Is Viewed | After Purging jsDelivr | Action Needed |
|-----------------------|------------------------|---------------|
| **Your Website** (via GitHub API) | ✅ Updates automatically | Just purge jsDelivr |
| **Any external site** | ✅ Updates automatically | Just purge jsDelivr |
| **GitHub.com** (README/docs) | ❌ Still cached by Camo | Add `?v=2` to URL |

> [!TIP]
> **Only GitHub documentation requires the `?v=X` version bump.** Your portfolio website and all other consumers will see updated images immediately after jsDelivr purge.

### Understanding the Caching Layers

| Layer | URL Pattern | Cache Owner |
|-------|-------------|-------------|
| **Layer 1** | `cdn.jsdelivr.net/...` | jsDelivr CDN |
| **Layer 2** | `camo.githubusercontent.com/...` | GitHub Camo Proxy |

**Why this matters:** GitHub proxies ALL external images through its Camo service for security. Even after jsDelivr cache is purged, GitHub's Camo may still serve the old cached version.

---

### Step 1: Purge jsDelivr CDN Cache

1. **Construct the purge URL:**
   ```
   https://purge.jsdelivr.net/gh/AmanSuryavanshi-1/portfolio-assets@main/{PATH_TO_FILE}
   ```
   
2. **Execute:** Open the URL in browser (GET request).

3. **Verify:** Response should show `{"status": "finished"}`.

**Example:**
```
https://purge.jsdelivr.net/gh/AmanSuryavanshi-1/portfolio-assets@main/N8N-GithubBackup/v5_canvas_overview.webp
```

---

### Step 2: Bust GitHub Camo Cache

> [!WARNING]
> GitHub Camo has its own cache independent of jsDelivr. A simple commit to documentation will NOT refresh it.

**Solution: Add a version query parameter to the image URL**

In the documentation file where you reference the image:

```diff
- ![Image](https://cdn.jsdelivr.net/gh/AmanSuryavanshi-1/portfolio-assets@main/N8N-GithubBackup/image.webp)
+ ![Image](https://cdn.jsdelivr.net/gh/AmanSuryavanshi-1/portfolio-assets@main/N8N-GithubBackup/image.webp?v=2)
```

**Rules:**
- Increment `?v=2` → `?v=3` → `?v=4` each time you update the same image
- This forces GitHub to treat it as a completely new URL
- Commit and push the documentation change

---

### Quick Reference: Complete Update Workflow

When updating an existing image:

```bash
# 1. Replace the image file locally and push
git add path/to/image.webp
git commit -m "Update image"
git push origin main

# 2. Purge jsDelivr cache (open in browser)
# https://purge.jsdelivr.net/gh/AmanSuryavanshi-1/portfolio-assets@main/path/to/image.webp

# 3. Update image URL in documentation with version bump
# Change: image.webp → image.webp?v=2
# Commit and push the documentation
```

---

## Best Practices

*   **File Naming:** Use kebab-case or snake_case (e.g., `project-dashboard.webp`). Avoid spaces.
*   **Optimization:** Convert PNG/JPG to WebP for smaller payloads and better Core Web Vitals.
*   **Version Tracking:** Maintain a comment in documentation noting current cache-bust version if frequently updated.
