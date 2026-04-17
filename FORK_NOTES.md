# Closient's fork of terraform-provider-stripe

Fork of [`lukasaron/terraform-provider-stripe`](https://github.com/lukasaron/terraform-provider-stripe),
which was archived upstream 2026-04-17. Baselined at `v3.4.1`. **Published to
the public Terraform Registry at [`closient/stripe`](https://registry.terraform.io/providers/closient/stripe).**

Issues, Discussions, Wiki, and Projects are intentionally disabled on this
repo — this fork exists to unblock closient's internal usage, not to host a
community. External PRs are welcome but won't be prioritized.

## Why fork

`stripe_price`'s destroy path was a no-op: removing the resource from
Terraform config reported "destruction complete" but the price stayed
`active=true` in Stripe. We hit this in [C-2314](https://linear.app/closient/issue/C-2314)
(billing pivot — 16 legacy prices stayed active across both envs after
Terraform apply). Upstream is archived so the fix has to live here.

## What this fork changes

| Commit | Change |
|--------|--------|
| [#2](https://github.com/closient/terraform-provider-stripe/pull/2) | `stripe/resource_stripe_price.go::resourceStripePriceDelete` now calls `POST /v1/prices/{id} {active: false}` instead of silently no-oping. Also fixes the mis-worded "promotion code" log that was copy-pasted from the promotion-code delete handler. |

## Release

Tags matching `v*` on `main` trigger `.github/workflows/release.yml`:

1. GoReleaser builds per-platform zips + SHA256SUMS + GPG-signed checksum
2. Uploads to GitHub Releases (picked up by Terraform Registry webhook)

Release-signing key: registered with the Registry under the `closient`
namespace. Private key + passphrase live in the repo's Actions secrets
(`GPG_PRIVATE_KEY`, `PASSPHRASE`). Public key is `.github/gpg-public-key.asc`
in this repo for reference.

## Consuming

Standard registry syntax, no mirror needed:

```hcl
terraform {
  required_providers {
    stripe = {
      source  = "closient/stripe"
      version = "~> 3.4"
    }
  }
}
```

## Bumping to a new upstream release

When upstream gets a maintained successor, prefer switching back. Until then,
periodically:

1. `git fetch upstream && git log upstream/main --oneline` to see new commits
   (upstream is archived so this is usually a no-op, but check anyway).
2. Cherry-pick relevant changes, keep our one-line delete fix.
3. Bump version, tag, push.
