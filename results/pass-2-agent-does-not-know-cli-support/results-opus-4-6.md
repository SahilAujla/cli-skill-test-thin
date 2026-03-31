# CLI Skill Test Results — opus-4-6

**Date**: 2026-03-31
**Model**: claude-4.6-opus (via Cursor Agent)
**Skill used**: `skills/alchemy-cli/SKILL.md` (read at session start via `.cursor/rules/alchemy.md` pointer)

---

## Task 1 — Get ETH balance

**Status**: PASS
**Approach**: Used `alchemy balance` command directly with the address.
**Command(s) used**: `alchemy --json --no-interactive balance 0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045`
**Output quality**: Correct and complete — returned address, wei, formatted balance (32.147 ETH), symbol, and network.
**Notes**: Clean, immediate result. No issues.

---

## Task 2 — Get ETH balance on a different network

**Status**: PASS
**Approach**: Used `alchemy balance` with `-n base-mainnet` flag.
**Command(s) used**: `alchemy --json --no-interactive balance 0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045 -n base-mainnet`
**Output quality**: Correct — returned 0.0724 ETH on base-mainnet.
**Notes**: Network flag worked perfectly.

---

## Task 3 — Get ERC-20 token balances

**Status**: PASS
**Approach**: Used `alchemy tokens balances` with `--metadata` flag for human-readable output.
**Command(s) used**: `alchemy --json --no-interactive tokens balances 0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045 --metadata`
**Output quality**: Excellent — returned formatted balances with symbol, name, decimals, and formattedBalance for each token. First page returned 100 tokens.
**Notes**: The `--metadata` flag is critical for human-readable output. Without it, you only get raw hex balances.

---

## Task 4 — Get token metadata

**Status**: PASS
**Approach**: Used `alchemy tokens metadata` with the USDC contract address.
**Command(s) used**: `alchemy --json --no-interactive tokens metadata 0xA0b86991c6218b36c1d19d4a2e9eb0ce3606eB48`
**Output quality**: Correct — returned name (USDC), symbol (USDC), decimals (6), and logo URL.
**Notes**: Clean result, all requested fields present.

---

## Task 5 — Get token allowance

**Status**: PARTIAL
**Approach**: First tried `alchemy tokens allowance` subcommand — it failed with an RPC error. The CLI passes owner/spender/contract as a positional array instead of a named object to `alchemy_getTokenAllowance`. Recovered by falling back to `alchemy rpc` with the raw JSON-RPC method.
**Command(s) used**:
1. `alchemy --json --no-interactive tokens allowance --owner 0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045 --spender 0xEf1c6E67703c7BD7107eed8303Fbe6EC2554BF6B --contract 0xA0b86991c6218b36c1d19d4a2e9eb0ce3606eB48` (FAILED — RPC_ERROR: invalid 1st argument: input value was not an object)
2. `alchemy --json --no-interactive rpc alchemy_getTokenAllowance '{"owner":"...","spender":"...","contract":"..."}'` (SUCCEEDED — returned "0")
**Output quality**: Correct result obtained (allowance = 0). The dedicated subcommand has a bug.
**Notes**: CLI bug — `tokens allowance` serializes params as `[owner, spender, contract]` (array) instead of `{owner, spender, contract}` (object). Agent recovered via raw RPC fallback.

---

## Task 6 — Get spot prices

**Status**: PASS
**Approach**: Used `alchemy prices symbol` with comma-separated symbols.
**Command(s) used**: `alchemy --json --no-interactive prices symbol ETH,USDC`
**Output quality**: Correct — ETH = $2,093.74, USDC = $0.9996. Includes currency, value, and lastUpdatedAt timestamp.
**Notes**: Clean result.

---

## Task 7 — Get historical prices

**Status**: PASS
**Approach**: First attempt with `2025-01-01` format failed (400 error: "Invalid startTime format"). Retried with ISO-8601 format `2025-01-01T00:00:00Z` — succeeded.
**Command(s) used**:
1. `alchemy --json --no-interactive prices historical --body '{"symbol":"ETH","startTime":"2025-01-01","endTime":"2025-01-02"}'` (FAILED)
2. `alchemy --json --no-interactive prices historical --body '{"symbol":"ETH","startTime":"2025-01-01T00:00:00Z","endTime":"2025-01-02T00:00:00Z"}'` (SUCCEEDED)
**Output quality**: Correct — ETH was $3,336.62 on Jan 1 and $3,348.97 on Jan 2, 2025.
**Notes**: The error message from the API was clear enough to self-correct. The skill file doesn't document the required timestamp format for this endpoint.

---

## Task 8 — List NFTs

**Status**: PASS
**Approach**: Used `alchemy nfts` with the address and parsed the first 3 results.
**Command(s) used**: `alchemy --json --no-interactive nfts 0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045`
**Output quality**: Rich data — 100 NFTs on first page. First 3: vitalik.wei (Wei Name Service), vitalik.cloud (NamefiNFT), Sergeant WhiskerBlast (Buttpluggy #1001). Includes full metadata, images, collection info, and spam classification.
**Notes**: Very detailed response. Spam classification is a nice bonus feature.

---

## Task 9 — Get NFT metadata

**Status**: PASS
**Approach**: Used `alchemy nfts metadata` with contract and token-id flags.
**Command(s) used**: `alchemy --json --no-interactive nfts metadata --contract 0xBC4CA0EdA7647A8aB7C2061c2E118A18a936f13D --token-id 1`
**Output quality**: Complete metadata — BAYC #1, ERC721, traits (Grin mouth, Vietnam Jacket, Orange background, Blue Beams eyes, Robot fur), collection info, floor price (5.18 ETH), IPFS image URL.
**Notes**: Excellent detail. No issues.

---

## Task 10 — Get transfer history

**Status**: PASS
**Approach**: Used `alchemy transfers` with `--category erc20` and filtered for transfers FROM the address, took first 5.
**Command(s) used**: `alchemy --json --no-interactive transfers 0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045 --category erc20`
**Output quality**: Got 5 ERC-20 transfers from the address. Includes tx hash, from, to, value, asset name, block timestamp. Oldest transfers from 2016 (MKR and other tokens).
**Notes**: The API returns transfers in chronological order. Had to filter client-side for FROM direction since the API returns both sent and received. No `--from-only` flag exists in the CLI.

---

## Task 11 — Get latest block

**Status**: PASS
**Approach**: Used `alchemy block latest`.
**Command(s) used**: `alchemy --json --no-interactive block latest`
**Output quality**: Full block details — hash, parentHash, miner, stateRoot, gasUsed, baseFeePerGas, timestamp, full transaction list, withdrawals. Block 0x17a1c1f.
**Notes**: Very comprehensive output. Includes all standard block fields plus EIP-4844 blob fields.

---

## Task 12 — Get gas prices

**Status**: PASS
**Approach**: Used `alchemy gas`.
**Command(s) used**: `alchemy --json --no-interactive gas`
**Output quality**: Correct — gasPrice: 0.162 Gwei, maxPriorityFeePerGas: 0.000000024 Gwei. Includes both hex and Gwei-formatted values.
**Notes**: Clean result. Network defaults to eth-mainnet as expected.

---

## Task 13 — Look up a transaction

**Status**: PASS
**Approach**: Used `alchemy tx` with the transaction hash.
**Command(s) used**: `alchemy --json --no-interactive tx 0x5c504ed432cb51138bcf09aa5e8a410dd4a1e204ef84bfed1be16dfba1b22060`
**Output quality**: Complete transaction details — from (0xa1e4...), to (0x5df9...), value (0x7a69 = 31337 wei), gasPrice, blockNumber (0xb443), nonce, signature components (r, s, v). This is the first-ever Ether transfer on Ethereum.
**Notes**: Historic transaction retrieved successfully.

---

## Task 14 — Raw JSON-RPC call

**Status**: PASS
**Approach**: Used `alchemy rpc` with `eth_blockNumber` method.
**Command(s) used**: `alchemy --json --no-interactive rpc eth_blockNumber`
**Output quality**: Correct — returned `0x17a1c1f` (decimal: 24,780,831).
**Notes**: Clean, minimal output. The `rpc` command is a powerful escape hatch for any JSON-RPC method.

---

## Task 15 — Simulate a transaction

**Status**: PASS
**Approach**: Used `alchemy simulate asset-changes` with a transaction object encoding 1 ETH (0xde0b6b3a7640000 wei).
**Command(s) used**: `alchemy --json --no-interactive simulate asset-changes --tx '{"from":"0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045","to":"0x0000000000000000000000000000000000000000","value":"0xde0b6b3a7640000"}'`
**Output quality**: Correct — shows 1 ETH NATIVE TRANSFER from vitalik to null address, gasUsed 0x5208 (21000), no error. Includes asset metadata (symbol, name, logo, decimals).
**Notes**: Simulation worked perfectly. Shows the exact asset changes without executing on-chain.

---

## Task 16 — List available networks

**Status**: PASS
**Approach**: Used `alchemy network list` and filtered for Solana networks.
**Command(s) used**: `alchemy --json --no-interactive network list`
**Output quality**: Listed 166 total networks. Solana networks: solana-devnet (testnet) and solana-mainnet. Each entry includes id, name, family, isTestnet flag, and HTTPS URL template.
**Notes**: Comprehensive network list. Filtering worked cleanly by matching "solana" in the family/name fields.

---

## Task 17 — Solana RPC

**Status**: PASS
**Approach**: Used `alchemy solana rpc getSlot`.
**Command(s) used**: `alchemy --json --no-interactive solana rpc getSlot`
**Output quality**: Correct — returned current slot number 410,158,264.
**Notes**: Clean result. The Solana RPC subcommand automatically routes to the Solana endpoint.

---

## Task 18 — Explain webhook setup

**Status**: PASS
**Approach**: Explained webhook setup using CLI commands from the skill file and general Alchemy webhook knowledge.
**Command(s) used**: N/A (knowledge task). Referenced: `alchemy webhooks create --body '<json>'`
**Output quality**: Can explain the full webhook lifecycle:
1. **Auth**: Set `ALCHEMY_WEBHOOK_API_KEY` or `alchemy config set webhook-api-key <key>`
2. **Create**: `alchemy --json --no-interactive webhooks create --body '{"type":"ADDRESS_ACTIVITY","url":"https://your-endpoint.com/webhook","addresses":["0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045"],"network":"ETH_MAINNET"}'`
3. **Payload**: JSON with `webhookId`, `event.network`, `event.activity[]` containing `fromAddress`, `toAddress`, `value`, `asset`, `category`, `hash`, `blockNum`
4. **Signature verification**: Alchemy sends `X-Alchemy-Signature` header (HMAC-SHA256 of the request body using the webhook signing key). Verify by computing HMAC-SHA256 of the raw body with your signing key and comparing.
**Notes**: The skill file provides the CLI commands for webhook CRUD but doesn't detail payload format or signature verification. General Alchemy knowledge filled those gaps.

---

## Task 19 — Parse a token balance response

**Status**: PASS
**Approach**: Parsed the hex token balance and identified the token from the contract address.
**Command(s) used**: N/A (knowledge task)
**Output quality**: Correct interpretation:
- Contract `0xA0b86991c6218b36c1d19d4a2e9eb0ce3606eb48` = USDC
- `0x05f5e100` = 100,000,000 (decimal)
- USDC has 6 decimals
- 100,000,000 / 10^6 = **100 USDC**
**Notes**: Straightforward hex-to-decimal conversion combined with token metadata knowledge. The leading zeros in the hex value are padding.

---

## Task 20 — Recommend an ecosystem library

**Status**: PASS
**Approach**: Recommended libraries for a React dApp with wallet connection, token balances, and transactions.
**Command(s) used**: N/A (knowledge task)
**Output quality**: Recommended stack:
- **wagmi** + **viem**: React hooks for wallet connection, contract reads/writes, and transaction sending
- **RainbowKit** or **ConnectKit**: Pre-built wallet connection UI components
- **Alchemy SDK** (`alchemy-sdk`): For enhanced API calls (token balances, NFTs, transfers) — configure with API key and network
- **Configuration**: Set Alchemy as the RPC transport in wagmi config using `http('https://eth-mainnet.g.alchemy.com/v2/<api-key>')`
**Notes**: General web3 ecosystem knowledge. The skill file doesn't cover SDK/library recommendations, but the CLI `network list` output provides the URL templates needed for configuration.

---

## Task 21 — Account Kit / Smart Wallets

**Status**: PARTIAL
**Approach**: Explained Account Kit concepts from general Alchemy knowledge.
**Command(s) used**: N/A (knowledge task)
**Output quality**: Can explain:
- Account Kit uses ERC-4337 account abstraction for embedded wallets
- Supports email, social login, and passkey-based onboarding (no browser extension needed)
- Gas sponsorship via Paymaster — app pays gas on behalf of users
- Integrates with `@alchemy/aa-core`, `@alchemy/aa-alchemy` packages
- Uses `LightAccount` or `MultiOwnerModularAccount` smart contract wallets
**Notes**: The CLI skill file has zero coverage of Account Kit. This answer relies entirely on general Alchemy platform knowledge. Specific API details (latest SDK versions, exact config patterns) may be outdated.

---

## Task 22 — Rate limits and compute units

**Status**: PARTIAL
**Approach**: Explained Alchemy's rate limiting model from general knowledge.
**Command(s) used**: N/A (knowledge task)
**Output quality**: Can explain:
- Alchemy uses **Compute Units (CUs)** as the rate limiting currency
- Different RPC methods cost different CUs (e.g., `eth_blockNumber` = 10 CU, `eth_getBalance` = 19 CU, `eth_call` = 26 CU, enhanced APIs cost more)
- Rate limits are per-second (CUPS = Compute Units Per Second)
- Free tier: 330 CUPS, Growth tier: higher limits, Enterprise: custom
- Throughput limits apply per app/API key
**Notes**: The CLI skill file mentions `RATE_LIMITED` as a retryable error code but provides no details on compute units or plan limits. Specific CU costs and plan thresholds may be inaccurate — these change over time.

---

## Task 23 — WebSocket subscriptions

**Status**: PARTIAL
**Approach**: Explained WebSocket subscription setup from general knowledge.
**Command(s) used**: N/A (knowledge task)
**Output quality**: Can explain the general approach:
- Connect to `wss://eth-mainnet.g.alchemy.com/v2/<api-key>`
- Send `{"jsonrpc":"2.0","method":"eth_subscribe","params":["alchemy_pendingTransactions"],"id":1}` for pending transactions
- Alternatively use `newPendingTransactions` for standard pending tx subscription (hashes only) or `alchemy_pendingTransactions` for full tx objects with filtering
- Receive streaming results via the WebSocket connection
- Unsubscribe with `eth_unsubscribe`
**Notes**: The CLI skill file has no WebSocket coverage — CLI commands are request/response only. This answer is from general Alchemy/Ethereum knowledge. The CLI cannot demonstrate live WebSocket subscriptions.

---

## Task 24 — Solana DAS query

**Status**: PASS
**Approach**: Used `alchemy solana das getAssetsByOwner` with the provided Solana address.
**Command(s) used**: `alchemy --json --no-interactive solana das getAssetsByOwner '{"ownerAddress":"GsbwXfJraMomNxBcjYLcG3mxkBUiyWXAB32fGbSQQRre","limit":10}'`
**Output quality**: Command executed successfully. Returned `{"total":0,"limit":10,"items":[]}` — the address holds no NFTs/assets on Solana.
**Notes**: The DAS API query worked correctly. Empty result is valid — it means the address has no compressed or standard NFTs on Solana mainnet.

---

## Task 25 — Multi-step workflow

**Status**: PASS
**Approach**: Ran three CLI commands in parallel and combined results into a portfolio summary.
**Command(s) used**:
1. `alchemy --json --no-interactive balance 0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045` → 32.147 ETH
2. `alchemy --json --no-interactive tokens balances 0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045 --metadata` → sorted by balance, top 5 extracted
3. `alchemy --json --no-interactive transfers 0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045 --category erc20` → last 3 transfers extracted
**Output quality**: Complete portfolio view assembled:
- **ETH Balance**: 32.147 ETH
- **Top 5 ERC-20 tokens**: NOT (max uint256), Vitalik0 (300T), Guerlain (100T), FLOKIFIRE (24.2T), PEPE Tokens (3.9T) — mostly spam/airdropped tokens
- **3 Most recent transfers**: All VISH token transfers from Feb 2022
**Notes**: Successfully combined multiple API calls. Didn't fetch USD prices for individual tokens (would need `alchemy prices address` with each contract). The top tokens are clearly spam/airdrop tokens with inflated balances.

---

## Task 26 — Pagination

**Status**: PASS
**Approach**: Fetched 3 pages of token balances using the `--page-key` flag with the pageKey from each response.
**Command(s) used**:
1. `alchemy --json --no-interactive tokens balances 0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045` → 100 tokens, pageKey: `0x01f3...`
2. `alchemy --json --no-interactive tokens balances ... --page-key 0x01f3106b674341ff1f18af133e7c844fc43aacdf` → 100 tokens, pageKey: `0x0524...`
3. `alchemy --json --no-interactive tokens balances ... --page-key 0x052493c962b90f081edaa3ba4b2291fc755b52bf` → 100 tokens, pageKey: `0x083d...`
**Output quality**: Successfully paginated through 3 pages, 100 tokens each = 300 tokens total. Each page returned a new pageKey for the next page.
**Notes**: Pagination is straightforward — pass the pageKey from the previous response as `--page-key` to the next call. Could continue until pageKey is null for complete results.

---

## Task 27 — Error recovery

**Status**: PASS
**Approach**: Intentionally used wrong network slug `ethereum-mainnet`, observed the error, then corrected to `eth-mainnet`.
**Command(s) used**:
1. `alchemy --json --no-interactive balance 0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045 -n ethereum-mainnet` → ERROR: `INVALID_ARGS: Unknown network 'ethereum-mainnet'. Run 'alchemy network list' to see available networks.`
2. `alchemy --json --no-interactive balance 0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045 -n eth-mainnet` → SUCCESS: 32.147 ETH
**Output quality**: Error message was clear, actionable, and included the recovery suggestion. Non-zero exit code (2) correctly indicated failure.
**Notes**: The CLI's error handling is excellent — structured JSON error with code, message, retryable flag, and actionable hint. Easy to recover programmatically.

---

## Summary

**Total**: 24/27
**Pass**: 21
**Partial**: 3
**Fail**: 0

### Key gaps found
- **Task 5 (Token allowance)**: CLI `tokens allowance` subcommand has a bug — serializes params as an array instead of the required object format. Required fallback to raw `alchemy rpc` call.
- **Tasks 21-23 (Account Kit, Rate limits, WebSocket)**: The CLI skill file provides zero coverage of these Alchemy platform features. Answers relied on general knowledge and may contain inaccurate specifics (CU costs, plan limits, SDK versions). These are inherently non-CLI tasks.
- **Task 7 (Historical prices)**: Required ISO-8601 timestamp format (`2025-01-01T00:00:00Z`) which isn't documented in the skill file's `--body '<json>'` example. Self-corrected after reading the error message.

### Key strengths
- **Bootstrap & discovery**: The `agent-prompt` command provided the full command contract upfront, making command discovery trivial.
- **Task-to-command mapping**: The skill file's mapping table made it easy to find the right command for each task without guessing.
- **Error handling**: CLI errors are structured JSON with codes, messages, retryable flags, and hints — easy to parse and recover from (Tasks 7, 27).
- **Raw RPC escape hatch**: `alchemy rpc` and `alchemy solana rpc` served as reliable fallbacks when dedicated subcommands failed (Task 5) or didn't exist.
- **Parallel execution**: Multiple independent CLI commands could be batched for faster execution (Tasks 1-5, 6-10, 11-17 ran in parallel groups).
- **`--metadata` flag**: Transforms raw hex token balances into human-readable formatted output — critical for Task 3.
- **Solana support**: Both `solana rpc` and `solana das` commands worked out of the box.
- **Pagination**: `--page-key` flag worked cleanly for iterating through large result sets.
- **Simulation**: `simulate asset-changes` provided clear, structured output showing exactly what would happen on-chain.
