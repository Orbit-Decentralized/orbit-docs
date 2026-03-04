# Technical Appendix

This page captures the implementation-oriented mechanics described in the whitepaper. It is not a substitute for audited code or integration docs, but it explains the protocol's intended architecture.

## Core Infrastructure

Orbit is described as being built on two foundational components:

- Meteora DLMM for concentrated AMM liquidity
- A conditional token mechanism for minting YES and NO outcome tokens

## Conditional Token Mechanism

Each market deploys a dedicated conditional token contract. Users deposit collateral to mint paired YES and NO tokens. At resolution:

- the winning token redeems 1:1 against collateral
- the losing token has zero redemption value

This keeps settlement deterministic and collateralized.

## DLMM Integration

Rather than pairing an outcome token with a stablecoin, Orbit pairs YES directly against NO. That lets prices map more naturally to probabilities.

Example:

- a YES:NO ratio of `3:1` corresponds to an implied YES probability of roughly `75%`

The AMM organizes liquidity into discrete bins. The active bin is the one currently earning fees and serving live execution.

## Probability Boundaries

The whitepaper proposes constraining tradable probabilities to a bounded range such as `1%` to `99%`. The purpose is to avoid pathological conditions near absolute certainty, where price movement would otherwise require impractical capital and could damage market interpretability.

## Order Execution

### Market Orders

Market orders execute immediately through the AMM's standard swap path, subject to available liquidity and slippage.

### Limit Orders

Limit-order-like behavior is implemented through conditional single-sided liquidity positions:

1. the user deposits funds into a hook contract
2. the hook mints YES/NO tokens through the conditional token mechanism
3. one side is deployed as single-sided liquidity at a target probability range
4. the other side is held in escrow until the trigger condition is met

This preserves AMM-based execution while approximating familiar limit order behavior.

## Belief-Based Opening Price

The whitepaper defines the opening price as the capital-weighted average of LP probability expectations:

`Opening Price = sum(LP capital * LP probability) / sum(LP capital)`

That opening state is the protocol's main alternative to default-priced instant-launch markets.

## Time-Aware Liquidity Adjustment

The technical appendix ties liquidity decay to time remaining until expiry. The design goal is to spread LP risk across the market lifecycle instead of concentrating it near settlement.

Operationally, the whitepaper describes:

- scheduled LP withdrawals
- custodial handling of withdrawn LP inventory
- periodic rebalancing toward each LP's target orientation

## Custody and Settlement

- LP inventory withdrawn during decay remains tracked in protocol custody until settlement
- Trader YES/NO tokens remain in user wallets
- Final settlement is executed on-chain after the outcome is reported

## Current Status

This appendix reflects the current design direction from the Orbit whitepaper. Precise contract interfaces, oracle rules, and implementation details should be treated as subject to change until they are finalized in code and audits.
