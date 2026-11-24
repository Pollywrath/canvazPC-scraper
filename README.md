# CanvazPC Full Catalogue Scraper

This repository runs the v5.0 exporter on GitHub-hosted runners. The repository is public, so standard GitHub-hosted Linux runners are available for these Actions jobs.

## 1. Collect the full catalogue

1. Open the **Actions** tab.
2. Select **Full CanvazPC Catalogue**.
3. Select **Run workflow** and keep the defaults (2 workers, 2 seconds between request starts).
4. Open the completed run and download the artifact named `canvazpc-full-catalogue-run-N`.

This job scrapes all discovered CanvazPC categories and product pages. It collects specifications and CanvazPC's aggregated retailer offer listings. It does not independently visit retailer websites.

## 2. Independently verify retailer prices and stock

1. Open the **Actions** tab.
2. Select **Verify Retailer Offers**.
3. Select **Run workflow**. The defaults point to the successful catalogue run #2, use a 1-second minimum interval, and use 4 HTTP workers.
4. Download the artifact named `canvazpc-retailer-verification-run-N` when it finishes.

The verification job downloads the previous catalogue artifact and reuses its 6,544-product detail checkpoint. It does **not** repeat the 6,544 product-page scrape. It reads current category listings to reconnect products, then visits each unique external retailer URL and evaluates the retrieved page against the expected product and seller. The source catalogue contains 16,686 offer records across 14,145 unique purchase URLs.

The 4 workers allow slow page requests to overlap. The shared rate limiter still enforces at least 1 second between request starts across all workers; this is not four requests per second. The verification job has a six-hour limit and saves a resumable verification checkpoint in its artifact/cache.

A retailer can still block automated requests, require a variant selection, omit a price, or provide no explicit stock information. Those results are recorded as blocked/unverified/ambiguous rather than being treated as confirmed prices. The job does not bypass access controls, solve CAPTCHAs, use proxies to evade blocks, or log in.

## Files

- `canvazpc_export.py`: reconstructed automatically on the runner from the compressed payload; the plaintext script is not stored in the repository.
- `requirements.txt`: HTTP scraper dependencies.
- `.github/workflows/full-catalogue.yml`: full-catalogue cloud job.
- `.github/workflows/verify-retailer-offers.yml`: independent retailer verification job.
- `.github/payload/exporter-00N.b64`: compressed source payload fragments used to restore the Python script.

The exporter payload is gzip-compressed and base64-encoded in ordered fragments. Workflows concatenate and verify the payload, reconstruct the Python source, and run offline self-tests before network requests.

## Request behavior and resume

The full-catalogue workflow uses 2 workers and a shared 2-second request-start interval by default. The verification workflow defaults to 4 HTTP workers and a shared 1-second request-start interval. Each workflow is serialized independently so two large crawls of the same type do not run simultaneously.

The verification workflow restores the most recent GitHub Actions cache containing its product/seller-specific verification checkpoint. Verified results are reused only while fresh; blocked and fetch-error results are retried. The checkpoint is also included in the downloadable artifact when the upload step is reached.
