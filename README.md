# terraform-provider-stripe (closient fork) — retired

**Do not use this provider.** It is retired as of 2026-04-17.

Use Stripe's official provider instead:

```hcl
terraform {
  required_providers {
    stripe = {
      source  = "stripe/stripe"
      version = "~> 0.2"
    }
  }
}
```

Registry: https://registry.terraform.io/providers/stripe/stripe

## Why this fork existed

[lukasaron/terraform-provider-stripe](https://github.com/lukasaron/terraform-provider-stripe)
was the de-facto community provider, but was archived on 2026-04-17 and had a
`stripe_price` destroy bug: removing a `stripe_price` resource from Terraform
config was a no-op at the Stripe API level, leaving `active=true` prices
orphaned. We hit that bug during the
[C-2336](https://linear.app/closient/issue/C-2336) billing migration.

This fork baselined at `lukasaron/stripe v3.4.1` and shipped a single
one-line fix: `resourceStripePriceDelete` now issues
`POST /v1/prices/{id} {active: false}` instead of silently clearing state.
See [`CHANGELOG.md`](./CHANGELOG.md) §3.4.3.

## Why it was retired

Partway through preparing to maintain this fork long-term, we discovered
that Stripe themselves publish an officially-supported provider at
[stripe/stripe](https://registry.terraform.io/providers/stripe/stripe).
Its V1 resource coverage matched everything we used, with an additional
V2 Billing family (pricing plans, rate cards, service actions) worth
exploring for future work.

Maintaining a fork when the vendor maintains an official provider was
not a good trade, so we migrated state to `stripe/stripe` and retired
this repository.

## History

- **Source**: Fork of
  [`lukasaron/terraform-provider-stripe`](https://github.com/lukasaron/terraform-provider-stripe)
  at `v3.4.1`.
- **Published**: `registry.terraform.io/providers/closient/stripe` `v3.4.3`.
- **Retired**: 2026-04-17 after migration to `stripe/stripe`.

The registry entry for `closient/stripe` has been unpublished and this
GitHub repository has been archived.
