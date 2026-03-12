---
name: cobo-agentic-wallet
description: |
  Use this skill for any crypto wallet operation: sending tokens, calling contracts, checking balances, querying transactions, or handling policy denials.
---

# cobo-agentic-wallet

Operate wallets through the `caw` CLI with policy enforcement. Owners configure limits and approve transactions; agents execute within those guardrails.

## Install

```bash
pip install cobo-agentic-wallet
caw --help
```

## Auth Setup

Credentials are auto-stored by `caw onboard provision`. Set API URL before running any command:

```bash
export AGENT_WALLET_API_URL=https://api-agent-wallet-core.sandbox.cobo.com
export AGENT_WALLET_API_KEY=<your_api_key>  # obtained from caw onboard provision output
```

## Getting Started

### Onboarding

1. Owner opens the Web Console or Human App and gets a setup token.
2. Agent runs:
   ```bash
   caw --format table onboard provision --token <TOKEN>
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

### Execution guidance for AI agents

- For any long-running command: run in background, poll output every 10–15 seconds, and report progress to the user as each step completes — don't wait to be asked.
- Currently long-running commands: `caw wallet create` (60–180 seconds). Progress steps are printed to stdout in the format `[n/total] Step description... done`.
- Any non-zero exit code indicates failure — check output for error details.

## Common Commands

Use `--format json` for programmatic output (transfers, policy checks, audit queries) and `--format table` for human-readable display (onboarding, status checks).

### Onboarding (`caw onboard`)

| Command | Description |
|---------|-------------|
| `caw onboard provision --token <TOKEN>` | Provision with owner-issued setup token |
| `caw onboard self-test --wallet <wallet_uuid>` | Run blocked + allowed transfer probes to validate policy |
| `caw demo` | Run the quickstart demo workflow |

### Wallet (`caw wallet`)

| Command | Description |
|---------|-------------|
| `caw wallet list` | List wallets accessible to the caller |
| `caw wallet create --name <name> --main-node-id <id>` | Create a new MPC wallet |
| `caw wallet get <wallet_uuid>` | Get wallet details |
| `caw wallet balance <wallet_uuid>` | Get wallet token balances |
| `caw wallet status <wallet_uuid>` | Inspect spend summary and freeze state |
| `caw wallet update <wallet_uuid> --name <name>` | Update wallet properties |


### Address (`caw address`)

| Command | Description |
|---------|-------------|
| `caw address list <wallet_uuid>` | List addresses for a wallet |
| `caw address create <wallet_uuid> --chain <id>` | Create new address on chain |
| `caw address recent <wallet_uuid>` | List recently used addresses |

### Transactions (`caw tx`)

| Command | Description |
|---------|-------------|
| `caw tx transfer <wallet_uuid> --to <addr> --token <id> --amount <n> --chain <id>` | Transfer tokens with policy enforcement |
| `caw tx list <wallet_uuid>` | List recent transactions with status |
| `caw tx get <tx_id>` | Get transaction details |
| `caw tx call <wallet_uuid> --contract <addr> --calldata <hex> --chain <id>` | Execute contract call |
| `caw tx estimate-transfer-fee <wallet_uuid> --to <addr> --token <id> --amount <n> --chain <id>` | Estimate transfer fee |
| `caw tx estimate-call-fee <wallet_uuid> --contract <addr> --calldata <hex> --chain <id>` | Estimate contract call fee |

### Delegation (`caw delegation`)

| Command | Description |
|---------|-------------|
| `caw delegation get <delegation_id>` | Get delegation details |
| `caw delegation received` | List delegations received as operator |


### Policy (`caw policy`)

| Command | Description |
|---------|-------------|
| `caw policy list` | List policies |
| `caw policy get <policy_id>` | Get policy details |
| `caw policy dry-run <policy_id> --action <action> --params <json>` | Test policy evaluation |

### Pending Operations (`caw pending`)

| Command | Description |
|---------|-------------|
| `caw pending list` | List pending operations |
| `caw pending get <operation_id>` | Get pending operation details |


## Usage Examples

### Self-test after provisioning
```bash
caw --format json onboard self-test --wallet <wallet_uuid>
```

### Transfer tokens
```bash
caw --format json tx transfer <wallet_uuid> \
  --to 0x1234...abcd \
  --token USDC \
  --amount 10 \
  --chain BASE \
  --request-id pay-invoice-1001
```

### List recent transactions
```bash
caw --format json tx list <wallet_uuid> --limit 20
```

### Check wallet balance
```bash
caw --format json wallet balance <wallet_uuid>
```

## Error Handling

- Policy denials return structured JSON in `--format json` mode.
- Use denial `suggestion` field to retry with adjusted parameters.

