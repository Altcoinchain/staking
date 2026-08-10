# Altcoinchain Validator Staking

Web UI for registering validators, delegating ALT, claiming rewards and managing
withdrawals on Altcoinchain (chainId **2330**).

Live: **https://altcoinchain.github.io/staking/**

Single static page — no build step, no framework, no dependencies. It talks to
the chain over JSON-RPC for reads and through your browser wallet
(Rabby / MetaMask) for writes.

## Contract

`ValidatorStaking` — [`0x2e05FfB10eF99e3c8B2BE1b752D7D3D45E6AC2a7`](https://github.com/Altcoinchain/contracts)

| Parameter | Value |
|---|---|
| Minimum validator self-stake | 32 ALT |
| Max commission | see `MAX_COMMISSION` on-chain |
| Withdrawal delay (delegations) | 7 days |
| Activity threshold | 100 blocks |
| Slashing penalty | 10% of total stake |

## What the UI supports

- `registerValidator(commission)` — become a validator (32 ALT minimum)
- `addSelfStake()` — increase your own stake
- `attest()` — liveness attestation
- `delegate(validator)` / `undelegate(validator, amount)`
- `completeWithdrawals()` — claim after the 7-day delay
- `claimRewards(validator)`
- Live validator table with online / offline / slashed status

## Read this before you stake

**A validator's self-stake cannot be withdrawn.** The deployed contract has no
`unstake`, `unregister`, or exit function. Every write to `selfStake` either adds
to it (`registerValidator`, `addSelfStake`) or removes it via `slash()`.
`undelegate()` operates on the separate `delegations` mapping and does not touch
self-stake, and `_removeValidator()` does not refund.

Practical consequence: **stake the 32 ALT minimum as a validator, and put any
additional capital in through `delegate()` instead** — delegations are
withdrawable via `undelegate` → 7-day delay → `completeWithdrawals()`.

Delegated funds are still exposed to slashing: the 10% penalty is taken from
self-stake first, then from delegations.

## Using your own node

The page defaults to a public RPC. To point it at your own node:

```
https://altcoinchain.github.io/staking/?rpc=https://your-node.example
```

or persistently in the browser console:

```js
localStorage.setItem("alt_rpc", "https://your-node.example");
```

The endpoint must be `https://` — the page is served over HTTPS and browsers
block mixed content.

## Validator status

Validators are registered on-chain now, but rewards and finality duties only
begin once the hybrid PoW/PoS fork activates. Before then, `isOnline` will read
false more than 100 blocks after your last `attest()`, which is expected and
harmless.
