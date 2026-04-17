# Private fork notes

This is closient's private fork of
[`lukasaron/terraform-provider-stripe`](https://github.com/lukasaron/terraform-provider-stripe),
which was archived upstream 2026-04-17. Forked at `v3.4.1`.

## Why fork

`stripe_price`'s delete path was a no-op: removing the resource from Terraform
config reported "destruction complete" but the price stayed `active=true` in
Stripe. We hit this in C-2314 (Closient billing pivot — 16 legacy Business
prices stayed active across both envs after Terraform apply). With upstream
archived, the fix has to live here.

## What this fork changes

| Commit | Change |
|--------|--------|
| C-2336 | `stripe/resource_stripe_price.go::resourceStripePriceDelete` now calls `POST /v1/prices/{id} {active: false}` instead of silently no-oping. Also fixes the mis-worded "promotion code" log that was copy-pasted from the promotion-code delete handler. |

## Publishing

Tags matching `v*` trigger `.github/workflows/release.yml`:

1. GoReleaser builds per-platform zips + `SHA256SUMS` + `manifest.json`
   (config: `.github/gorelease.yml`; no GPG signing — the Terraform Network
   Mirror protocol authenticates via SHA256 hashes, not signatures)
2. Upload to GitHub Releases (source of truth)
3. Build a Terraform Network Mirror JSON manifest (`index.json` +
   `<version>.json`) keyed by platform with `h1:` hashes
4. Sync manifests + zips to `s3://${MIRROR_BUCKET}/closient/stripe/`

## Consuming

Atlantis + local dev read via a Terraform Network Mirror configured in
`.terraformrc` pointing at `${MIRROR_BASE_URL}`. Atlantis config lives in
`closient/closient` under `terraform/accounts/developer-resources/atlantis.tf`.

To bump to a new version:

1. Cut a tag here (`git tag v3.4.2 && git push origin v3.4.2`)
2. Watch the release workflow publish to S3
3. Bump `required_providers.stripe.version` in `closient/closient`

## Staying close to upstream

When upstream eventually gets replaced by a maintained alternative, we want to
be able to migrate back with minimal diff. So:

* Don't add features speculatively. Only diffs we can point at a ticket.
* Keep changes to the delete/archive path and obvious bugs.
* If we accumulate enough changes, publish via `lukasaron/stripe` fork URL in
  GitHub + revisit whether a community-maintained fork has emerged.
