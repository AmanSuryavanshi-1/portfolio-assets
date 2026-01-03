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

jsDelivr implements aggressive caching policies for performance. Updates to existing filenames may not reflect immediately.

### Strategy 1: Cache Purging (Recommended)

Manually trigger a cache invalidation for a specific resource via the jsDelivr Purge API.

1.  **Construct Purge Request:**
    `https://purge.jsdelivr.net/gh/AmanSuryavanshi-1/portfolio-assets@main/{PATH_TO_FILE}`
2.  **Execution:** Send a GET request (browse to URL) to the constructed endpoint.
3.  **Verification:** A JSON response `{"status": "finished"}` confirms the purge.

### Strategy 2: Versioning / Cache Busting

Force immediate retrieval by appending a unique query parameter to the asset URL in the codebase.

```diff
- imageUrl: "https://.../asset.webp"
+ imageUrl: "https://.../asset.webp?v=2"
```

## Best Practices

*   **File Naming:** Use kebab-case or snake_case (e.g., `project-dashboard.webp`). Avoid spaces to prevent URL encoding issues.
*   **Optimization:** Convert PNG/JPG to WebP format to reduce payload size and improve Core Web Vitals.
