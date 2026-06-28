# Proposed documentation improvements — `docs/base.md`

Two additions to the troubleshooting section based on real friction encountered
while migrating a Foundry project to base-forge v1.1.0 shortly after the Beryl
mainnet activation.

---

## 1. `vm.etch` migration guide

### What happens

Developers porting existing test suites to base-forge commonly mock the B20Factory
at its canonical precompile address using `vm.etch`:

```solidity
MockB20Factory mock = new MockB20Factory();
vm.etch(0xB20f000000000000000000000000000000000000, address(mock).code);
```

Under base-forge this aborts immediately with:

```
[FAIL: vm.etch: cannot use precompile 0xB20f000000000000000000000000000000000000 as an argument]
```

The error is correct — base-forge compiles the real precompile in and protects it
from being overwritten — but it gives no guidance on what to do instead.

### Proposed addition

A new section **"Migrating tests from `vm.etch` mocks to real precompiles"**
(placed before the troubleshooting table) shows the before/after pattern:

- **Before**: `vm.etch` + hand-rolled mock  
- **After**: remove the mock entirely; base-forge seeds the precompile active
  with no node and no network access required

The section also covers the one follow-on adjustment: any test that asserted on
a predicted address derived from the mock's own formula should be updated to call
the real factory's `getB20Address(variant, sender, salt)` view instead.

---

## 2. `FeatureNotActivated` — verifying feature state before deploying

### What happens

The existing troubleshooting row reads:

> The precompile's feature is not activated on that chain yet. Local `base-anvil`
> seeds them active; on a network, use one where the feature is live.

This is accurate but leaves developers without a way to check whether the feature
is live on a given network before hitting the error. The natural next step — query
the ActivationRegistry — is not shown.

In practice, a developer targeting Base mainnet shortly after the Beryl hardfork
would send a transaction, receive `FeatureNotActivated(bytes32)` as the revert
data, and have no immediate path to understanding whether the feature was simply
delayed or permanently absent.

### Proposed addition

Expand the troubleshooting row to include the concrete verification command:

```bash
cast call 0x8453000000000000000000000000000000000001 \
  "isActivated(bytes32)(bool)" \
  <featureId> \
  --rpc-url <RPC>
```

With the two B20 feature IDs included inline:

| Feature | ID |
| --- | --- |
| `B20_ASSET` | `0xcdcc772fe4cbdb1029f822861176d09e646db96723d4c1e82ddfdeb8163ef54c` |
| `B20_STABLECOIN` | `0xecfa0def2c10020caaf65e6155aa69c84b24892aaef76eeac52e0e2b3a0b8601` |

---

## Diff

The full change is in [`docs/base.md`](docs/base.md). This file is for context
only and is not intended to be merged.
