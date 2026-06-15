# Project notes

This file captures the current standalone hackathon state of Agent Commerce Hub.

## Current implementation

- Service discovery uses the Sepolia `ServiceRegistryV2` deployment by default.
- Sellers can register AI service metadata, price, endpoint URI, and payment address.
- Buyers can discover services through the Web UI, CLI, or Agent API.
- Buyer Agent procurement supports natural-language requests, budget filtering, candidate ranking, and optional x402/CAW auto-payment.
- Seller Agent registration converts a service brief into registry metadata and posts it through an admin-gated API.
- Delivery proofs are recorded on-chain after paid x402 requests.
- The Agent Console shows discovery, selection, payment, and proof details for demos.

## Demo narrative

1. Seller Agent publishes a paid AI service.
2. Buyer Agent receives a user task and budget.
3. Buyer Agent discovers matching services from the registry.
4. Buyer Agent selects a candidate and enforces the maximum payment amount.
5. Cobo Agentic Wallet handles Pact approval and transfer execution.
6. Seller endpoint returns the delivery result.
7. The registry stores a proof trail that can be inspected later.

## Safety boundaries

- This repository should not contain private keys, API keys, CAW credentials, or production RPC secrets.
- Live writes are testnet-only by default.
- `DEMO_ADMIN_TOKEN` should be configured for shared deployments.
- Service removal is deactivation, not deletion, so historical proofs remain auditable.

## Future work

- Multi-buyer CAW session support instead of one server-paired demo wallet.
- Signed seller ownership checks for all service-management operations.
- Richer agent reasoning over service quality, seller reputation, latency, and proof history.
- Broker Agent support for composing several paid services into one procurement task.
