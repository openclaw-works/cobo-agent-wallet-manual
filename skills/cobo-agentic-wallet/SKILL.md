---
name: cobo-agentic-wallet
description: |
  Use this skill whenever a task involves a Cobo Agentic Wallet — including wallet setup and onboarding, token transfers, contract calls, balance checks, transaction queries, and policy denial handling. Also use it for DeFi strategy execution: Aave V3 borrow/repay, DEX token swaps (Uniswap V3), DCA (dollar cost averaging), grid trading, and prediction market positions — on EVM chains (Ethereum, Base, Arbitrum, Optimism, Polygon) or Solana (mainnet via Jupiter, devnet). Use it whenever the user mentions `caw`, `cobo-agentic-wallet`, MPC wallet agent operations, or needs to call a smart contract or DeFi protocol through a policy-enforced wallet. Use it even if they don't say "Cobo" — if they're building or operating an AI agent that needs to spend crypto with guardrails, this skill applies.
---

# cobo-agentic-wallet

Operate wallets through the `caw` CLI with policy enforcement. Owners configure limits and approve transactions; agents execute within those guardrails.

## Install

```bash
pip install cobo-agentic-wallet
caw --help
```

## Environments

| Environment | API URL | Web Console | TSS env |
|-------------|---------|-------------|---------|
| Sandbox | `https://api-agent-wallet-core.sandbox.cobo.com` | https://agenticwallet.sandbox.cobo.com/ | `sandbox` |
| Dev | `https://api-agent-wallet-core.dev.cobo.com` | https://agenticwallet.dev.cobo.com/ | `dev` |

## Auth Setup

Credentials are auto-stored by `caw onboard provision`. Set API URL before running any command:

```bash
export AGENT_WALLET_API_URL=https://api-agent-wallet-core.sandbox.cobo.com
export AGENT_WALLET_API_KEY=<your_api_key>  # obtained from caw onboard provision output
```

## Quick Start

### Onboarding

**Web Console:** Use the console matching your environment (see Environments table above) to get a setup token, set policies, and delegate wallets.

1. Owner opens the Web Console (see Environments table) or Human App and gets a setup token.
2. Agent runs (must match the environment of the setup token):
   ```bash
   # Sandbox
   caw --api-url https://api-agent-wallet-core.sandbox.cobo.com \
     onboard provision --token <TOKEN> --tss-env sandbox

   # Dev
   caw --api-url https://api-agent-wallet-core.dev.cobo.com \
     onboard provision --token <TOKEN> --tss-env dev
   ```
3. An API key is created and bound to the owner's account. Write it to your environment:
   ```bash
   export AGENT_WALLET_API_KEY=<api_key_from_output>
   ```
4. If instructed, create a wallet and address:
   ```bash
   caw wallet create --name <name> --main-node-id <id>
   caw address create <wallet_uuid> --chain <id>
   ```
5. Print a summary table with `agent_id`, `wallet_uuid`, `address` (if created), and config paths. If `wallet_uuid` is empty, remind the user to go to the Human App to delegate a wallet to this agent, then return to start using it.
6. Validate setup:
   ```bash
   caw --format json onboard self-test --wallet <wallet_uuid>
   ```
7. First transfer:
   ```bash
   caw --format json tx transfer <wallet_uuid> \
     --to 0x1234...abcd --token USDC --amount 1 --chain BASE
   ```

### Reset / Cleanup

When onboarding fails and you need to start over, run the reset script:

```bash
bash scripts/reset-env.sh
```

This will stop the TSS Node, back up existing state, and remove state files to allow re-provisioning. See [scripts/reset-env.sh](scripts/reset-env.sh) for details.

### Execution guidance for AI agents

- For any long-running command: run in background, poll output every 10-15 seconds, and report progress to the user as each step completes.
- Currently long-running commands: `caw wallet create` (60-180 seconds). Progress steps are printed to stdout in the format `[n/total] Step description... done`.
- Any non-zero exit code indicates failure -- check output for error details.

## Reference

Read the file that matches the user's task. Do not load files that aren't relevant.

- [CLI Command Reference](./commands.md) — Read when you need the full `caw` command syntax
- [Recipes](./recipes.md) — Read for quick one-liner examples of common operations

**DeFi recipes** — read the matching file when the user asks about a specific strategy:

| User asks about… | Read |
|---|---|
| Aave, borrow, repay, supply, collateral | [evm-defi-aave.md](./recipes/evm-defi-aave.md) |
| DEX swap, Uniswap, token exchange (EVM) | [evm-defi-dex-swap.md](./recipes/evm-defi-dex-swap.md) |
| DCA, dollar cost average, recurring buy (EVM) | [evm-defi-dca.md](./recipes/evm-defi-dca.md) |
| Grid trading, ladder orders (EVM) | [evm-defi-grid-trading.md](./recipes/evm-defi-grid-trading.md) |
| Solana DEX swap, Jupiter, SOL/USDC | [solana-defi-dex-swap.md](./recipes/solana-defi-dex-swap.md) |
| Solana DCA, recurring SOL purchase | [solana-defi-dca.md](./recipes/solana-defi-dca.md) |
| Solana grid trading | [solana-defi-grid-trading.md](./recipes/solana-defi-grid-trading.md) |
| Prediction market, Drift, Polymarket, long/short | [solana-defi-prediction-market.md](./recipes/solana-defi-prediction-market.md) |
| Policy denial, 403 error, TRANSFER_LIMIT_EXCEEDED | [error-handling.md](./recipes/error-handling.md) |
| Policy setup, dry-run, delegation | [policy-management.md](./recipes/policy-management.md) |
