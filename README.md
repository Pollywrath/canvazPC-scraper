# CanvazPC Full Catalogue Scraper

This repository runs the v5.0 exporter on GitHub-hosted runners. The repository is public, so GitHub Actions can use standard Linux runners without consuming a private-repository minutes allowance.

## Run the full catalogue from your phone

1. Open the **Actions** tab.
2. Select **Full CanvazPC Catalogue**.
3. Select **Run workflow** and keep the defaults (2 workers, 2 seconds between request starts).
4. Open the completed run and download the `canvazpc-full-catalogue` artifact.

The workflow scrapes **all discovered CanvazPC categories and product pages**. It deliberately does not set `--category`, `--limit`, or `--product-filter`. It uses `--offers` to collect product details and CanvazPC's aggregated retailer offer listings.

This first workflow does **not** visit every retailer independently. Retailer verification is a separate, much larger operation and is intentionally disabled for the full-catalogue job.

## Files

- `canvazpc_export.py`: CanvazPC exporter v5.0.
- `requirements.txt`: HTTP scraper dependencies.
- `.github/workflows/full-catalogue.yml`: manually triggered full-catalogue cloud job.
- `.github/payload/exporter-00N.b64`: compressed source payload fragments used by the workflow to restore the Python script before running.

The exporter payload is gzip-compressed and base64-encoded in seven ordered fragments. The workflow reconstructs the original Python source before installing dependencies and running it.

## Free usage

Because this repository is public, standard GitHub-hosted runners are free for this workflow under GitHub's public-repository Actions policy. This job is set to a maximum of six hours. Runtime depends on CanvazPC response times and the site's current catalogue size.

## Request behavior

Defaults are conservative: 2 workers with a shared 2-second minimum interval between request starts. The job is serialized so you don't accidentally run multiple full-catalogue crawls simultaneously.

The scraper only requests public CanvazPC pages. It does not bypass access controls, solve CAPTCHAs, use proxies to evade blocks, or log in.

## Resume behavior

The workflow restores a GitHub Actions cache containing the product-detail checkpoint when available. It saves the updated checkpoint after the run and uploads it with the exports, including on ordinary failures. A hard runner timeout can prevent the final artifact-upload step from running, but the cache action is configured to save the latest checkpoint during normal job cleanup.
