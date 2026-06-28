# Proposed documentation improvement — `docs/base.md`

One addition to the troubleshooting section based on friction encountered while
migrating an existing Foundry test suite to base-forge v1.1.0.

---

## `vm.etch` migration guide

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
from being overwritten — but it gives no guidance on what to do instead. The fix
is not obvious: the developer needs to delete the mock entirely rather than adjust it.

### Proposed addition

A new section **"Migrating tests from `vm.etch` mocks to real precompiles"**
(placed before the troubleshooting table) shows the before/after pattern:

**Before:**
```solidity
MockB20Factory mock = new MockB20Factory();
vm.etch(0xB20f000000000000000000000000000000000000, address(mock).code);
```

**After:**
```solidity
// Nothing — base-forge seeds the precompile active in-process.
```

The section also notes the one follow-on adjustment: tests that asserted on a
predicted address derived from the mock's own formula should call the real
factory's `getB20Address(variant, sender, salt)` view instead.

A matching row is added to the troubleshooting table pointing to this section.

---

## Minor: `FeatureNotActivated` — add feature IDs and verification command

The existing troubleshooting row says "use a network where the feature is live"
but doesn't show how to check. Added the `cast call` command against the
ActivationRegistry and the B20 feature IDs inline, so developers can verify
before hitting the error on a live network:

```bash
# B20_ASSET
cast call 0x8453000000000000000000000000000000000001 \
  "isActivated(bytes32)(bool)" \
  0xcdcc772fe4cbdb1029f822861176d09e646db96723d4c1e82ddfdeb8163ef54c \
  --rpc-url https://mainnet.base.org

# B20_STABLECOIN
cast call 0x8453000000000000000000000000000000000001 \
  "isActivated(bytes32)(bool)" \
  0xecfa0def2c10020caaf65e6155aa69c84b24892aaef76eeac52e0e2b3a0b8601 \
  --rpc-url https://mainnet.base.org
```

---

## Diff

Full change in [`docs/base.md`](docs/base.md). This file is for context only
and is not intended to be merged.
