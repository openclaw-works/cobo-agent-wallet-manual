---
name: cobo-agentic-wallet
description: |
  Use this skill for any crypto wallet operation: sending tokens, calling contracts, checking balances, querying transactions, or handling policy denials.
---

# cobo-agentic-wallet

Operate wallets through the `caw` CLI with policy enforcement. Owners configure limits and approve transactions via Telegram; agents execute within those guardrails.

## Install

```bash
pip install cobo-agent-wallet
caw --help
```

## Auth Setup

Credentials are auto-stored by `caw provision` / `caw init`. Set API URL before running any command:

```bash
export AGENT_WALLET_API_URL=https://api.agent-wallet-core.sandbox.cobo.com
export AGENT_WALLET_API_KEY=<your_api_key>  # only needed if not already stored
```

## Getting Started

### Path A: Owner-First (Telegram → Token → Provision)

1. Owner opens Telegram bot and sends `/start`.
2. Owner authenticates via magic link.
3. Owner configures chains, budgets, and limits in the chat.
4. Owner receives a setup token.
5. Agent runs:
   ```bash
   caw --format table provision --token <TOKEN>
   ```
6. Wallet is auto-provisioned with key shares, delegation, and policies.

### Path B: Agent-First (Init → Share Link → Auto-Detect)

1. Agent runs:
   ```bash
   caw --format table init
   ```
2. Agent shares the returned deep link with the owner.
3. Owner opens the link, authenticates, and configures preferences.
4. The command detects activation automatically, then downloads + init + starts TSS-Node, creates vault, and runs KeyGen.
5. The command exits when onboarding is complete.

`caw init` and `caw provision` are long-running blocking commands (typically 60–180 seconds). Do not interrupt them.

### Execution guidance for AI agents

Before running any `caw` command, ensure `AGENT_WALLET_API_URL` is set.

Both `caw init` and `caw provision` run for 60–180 seconds. When executing:

1. **Run in background** and poll output every 10–15 seconds.
2. **Report progress** to the user as each step completes (don't wait for the user to ask).
3. **Share the deep link immediately** — for `caw init` (Path B), the Telegram deep link is printed early in the output. Surface it to the user right away so the owner can start the Telegram flow in parallel.

**Progress steps** (printed to stdout):
```
[1/5] Registering with Cobo backend... done
[2/5] Downloading TSS-Node in background...
      Share this link with your wallet owner:
        https://t.me/<bot>?start=<agent_id>
[3/5] Waiting for owner to connect... done
[4/5] Initializing TSS-Node... done
[5/5] Starting TSS-Node and creating wallet... done
```

**Successful completion** — a table is printed at the end:
```
field                 | value
----------------------+------------------------------------------
agent_id              | caw_agent_<hex>
status                | active
deep_link             | https://t.me/<bot>?start=caw_agent_<hex>
wallet_uuid           | <uuid>
onboarding_session_id | <uuid>
config_path           | ~/.cobo-agent-wallet/config
tss_node_path         | ~/.cobo-agent-wallet/tss-node
```

Exit code 0 = success. Any non-zero exit code indicates failure — check output for error details.

## CLI Commands

All commands use `caw` as the entry point. Use `--format json` when you need to parse the output programmatically (transfers, policy checks, audit queries). Use `--format table` for human-readable display (onboarding, status checks).

### Onboarding & Status

| Command | Description |
|---------|-------------|
| `caw provision --token <TOKEN>` | Path A: provision with owner-issued setup token |
| `caw init` | Path B: agent-first onboarding, returns deep link for owner |
| `caw self-test --wallet <wallet_uuid>` | Run blocked + allowed transfer probes to validate policy |
| `caw status` | Inspect spend summary and freeze state |
| `caw demo` | Run the quickstart demo workflow |

### Wallet Operations

| Command | Description |
|---------|-------------|
| `caw list-wallets` | List wallets accessible to the caller |
| `caw balance <wallet_uuid>` | Get wallet balance |
| `caw transfer <wallet_uuid> --to <addr> --token <id> --amount <n> --chain <id>` | Transfer tokens with policy enforcement |
| `caw list-transactions <wallet_uuid>` | List recent transactions with status |
| `caw estimate-transfer-fee <wallet_uuid> --to <addr> --token <id> --amount <n> --chain <id>` | Estimate transfer fee |

### Contract Calls

| Command | Description |
|---------|-------------|
| `caw contract-call <wallet_uuid> --contract <addr> --calldata <hex> --chain <id>` | Execute contract call with policy enforcement |
| `caw estimate-contract-call-fee <wallet_uuid> --contract <addr> --calldata <hex> --chain <id>` | Estimate contract call fee |

### Audit & Monitoring

| Command | Description |
|---------|-------------|
| `caw audit-logs [--wallet <uuid>] [--result allowed\|denied\|error] [--action <type>]` | Query audit logs with filters |

## Owner-Side NL Assistant

Owners interact via Telegram:

- `/start` — Begin onboarding or resume session
- Natural language commands: set budgets, freeze/unfreeze, approve/reject, view activity
- Push notifications for approvals, policy violations, deposits, and transfers

## Usage Examples

### Path B onboarding (sandbox)
```bash
export AGENT_WALLET_API_URL=https://api.agent-wallet-core.sandbox.cobo.com
caw --format table init --tss-env sandbox
```

### Path A onboarding (sandbox)
```bash
export AGENT_WALLET_API_URL=https://api.agent-wallet-core.sandbox.cobo.com
caw --format table provision --token <TOKEN> --tss-env sandbox
```

### Self-test after provisioning
```bash
caw --format json self-test --wallet <wallet_uuid>
```

### Transfer tokens
```bash
caw --format json transfer <wallet_uuid> \
  --to 0x1234...abcd \
  --token USDC \
  --amount 10 \
  --chain BASE \
  --request-id pay-invoice-1001
```

### List recent transactions
```bash
caw --format json list-transactions <wallet_uuid> --limit 20
```

## Error Handling

- Policy denials return structured JSON in `--format json` mode.
- Use denial `suggestion` field to retry with adjusted parameters.
- `self-test` validates both blocked and allowed transfers automatically.
