# Test Results — GPT-5.4

> Agent context: local `skills/alchemy-cli/SKILL.md` plus official Alchemy docs for documentation-heavy tasks.
>
> Benchmark note: this is not a pure "thin skill only" run for Part 2, because official docs were consulted where needed.
>
> Date: 2026-03-31
>
> Agent: Cursor (GPT-5.4)
>
> CLI version: `0.4.0`

---

## Part 1: CLI-Supported Tasks

## Task 1.1 — Get ETH balance

**Status**: PASS
**Approach**: Used `alchemy balance` for the provided address on the default `eth-mainnet` network.
**Command(s) used**: `alchemy --json --no-interactive balance 0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045`
**Output quality**: Correct, concise JSON with address, network, symbol, and current balance (`32.147257835992150606` ETH).
**Notes**: None.

## Task 1.2 — Get ETH balance on a different network

**Status**: PASS
**Approach**: Re-ran the balance query with `-n base-mainnet`.
**Command(s) used**: `alchemy --json --no-interactive balance 0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045 -n base-mainnet`
**Output quality**: Correct JSON response for Base with balance `0.0724190485030745` ETH.
**Notes**: None.

## Task 1.3 — Get ERC-20 token balances

**Status**: PASS
**Approach**: Used `tokens balances` with `--metadata` to get human-readable names, symbols, decimals, and formatted balances.
**Command(s) used**: `alchemy --json --no-interactive tokens balances 0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045 --metadata`
**Output quality**: Strong. Returned 80 token balances on page 1 with readable metadata and a `pageKey` for pagination.
**Notes**: Sample results included `Holy Trinity (500)`, `Anonymous AI (200000)`, `Lyra (100000)`, and `Milady Cult Coin (1150000)`.

## Task 1.4 — Get token metadata

**Status**: PASS
**Approach**: Queried the USDC contract directly with `tokens metadata`.
**Command(s) used**: `alchemy --json --no-interactive tokens metadata 0xA0b86991c6218b36c1d19d4a2e9eb0ce3606eB48`
**Output quality**: Correct and complete. Returned `name=USDC`, `symbol=USDC`, `decimals=6`, and a logo URL.
**Notes**: None.

## Task 1.5 — Get token allowance

**Status**: PARTIAL
**Approach**: Tried the dedicated `tokens allowance` wrapper first, then recovered with raw RPC when the wrapper failed.
**Command(s) used**: `alchemy --json --no-interactive tokens allowance --owner 0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045 --spender 0xEf1c6E67703c7BD7107eed8303Fbe6EC2554BF6B --contract 0xA0b86991c6218b36c1d19d4a2e9eb0ce3606eB48`; fallback: `alchemy --json --no-interactive rpc alchemy_getTokenAllowance '{"owner":"0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045","spender":"0xEf1c6E67703c7BD7107eed8303Fbe6EC2554BF6B","contract":"0xA0b86991c6218b36c1d19d4a2e9eb0ce3606eB48"}'`
**Output quality**: The direct wrapper failed with `RPC_ERROR -32602` ("invalid 1st argument: input value was not an object"). The raw RPC fallback returned the correct allowance: `"0"`.
**Notes**: CLI bug in `tokens allowance` on `@alchemy/cli` `0.4.0`.

## Task 1.6 — Get spot prices

**Status**: PASS
**Approach**: Used the spot price lookup by symbol for both ETH and USDC.
**Command(s) used**: `alchemy --json --no-interactive prices symbol ETH,USDC`
**Output quality**: Correct and well-structured. Returned ETH at `$2097.8959614746` and USDC at `$0.9997198944` with timestamps.
**Notes**: None.

## Task 1.7 — Get historical prices

**Status**: PASS
**Approach**: Queried historical ETH price data over the requested 2025-01-01 to 2025-01-02 window.
**Command(s) used**: `alchemy --json --no-interactive prices historical --body '{"symbol":"ETH","startTime":"2025-01-01T00:00:00Z","endTime":"2025-01-02T00:00:00Z"}'`
**Output quality**: Correct. Returned ETH at `$3336.6175137659` on `2025-01-01T00:00:00Z` and `$3348.9672471236` on `2025-01-02T00:00:00Z`.
**Notes**: None.

## Task 1.8 — List NFTs

**Status**: PASS
**Approach**: Used the NFT ownership command with `--limit 3` to satisfy the exact task requirement.
**Command(s) used**: `alchemy --json --no-interactive nfts 0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045 --limit 3`
**Output quality**: Correct. Returned exactly 3 NFTs plus a `pageKey` for more.
**Notes**: The first three NFTs returned were `vitalik.wei`, `vitalik.cloud`, and `Sergeant WhiskerBlast`.

## Task 1.9 — Get NFT metadata

**Status**: PASS
**Approach**: Queried BAYC token `1` with `nfts metadata` and inspected both normalized fields and the raw metadata blob.
**Command(s) used**: `alchemy --json --no-interactive nfts metadata --contract 0xBC4CA0EdA7647A8aB7C2061c2E118A18a936f13D --token-id 1`
**Output quality**: Good. Returned collection/contract data, image URLs, token URI, and raw attributes for BAYC #1. Top-level `name`/`description` were `null`, but the underlying metadata was present.
**Notes**: Raw traits included `Mouth: Grin`, `Clothes: Vietnam Jacket`, `Background: Orange`, `Eyes: Blue Beams`, and `Fur: Robot`.

## Task 1.10 — Get transfer history

**Status**: PARTIAL
**Approach**: Used the `transfers` wrapper first, then switched to raw `alchemy_getAssetTransfers` with descending order because the wrapper did not produce the actual latest 5 transfers.
**Command(s) used**: `alchemy --json --no-interactive transfers 0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045 --category erc20 --max-count 5`; fallback: `alchemy --json --no-interactive rpc alchemy_getAssetTransfers '{"fromAddress":"0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045","category":["erc20"],"maxCount":"0x5","order":"desc","withMetadata":true}'`
**Output quality**: Correct after the raw-RPC workaround. The actual latest 5 outgoing ERC-20 transfers were BTT transfers on `2026-02-22`.
**Notes**: The high-level `transfers` wrapper appears to return ascending history and does not expose an `order` flag, so "last N" queries need raw RPC.

## Task 1.11 — Get latest block

**Status**: PASS
**Approach**: Queried the latest Ethereum block directly.
**Command(s) used**: `alchemy --json --no-interactive block latest`
**Output quality**: Correct block response with `number`, `hash`, `timestamp`, `gasLimit`, `gasUsed`, and `baseFeePerGas`.
**Notes**: Latest block at run time was `0x17a19a0`.

## Task 1.12 — Get gas prices

**Status**: PASS
**Approach**: Used the dedicated gas pricing command.
**Command(s) used**: `alchemy --json --no-interactive gas`
**Output quality**: Correct and concise. Returned `gasPriceGwei=0.434988166` and `maxPriorityFeePerGasGwei=0.000448`.
**Notes**: None.

## Task 1.13 — Look up a transaction

**Status**: PASS
**Approach**: Queried the transaction by hash.
**Command(s) used**: `alchemy --json --no-interactive tx 0x5c504ed432cb51138bcf09aa5e8a410dd4a1e204ef84bfed1be16dfba1b22060`
**Output quality**: Correct. Returned the well-known first ETH transaction with `from`, `to`, `value`, `gas`, `gasPrice`, `nonce`, and `blockNumber`.
**Notes**: `value` was `0x7a69` (31,337 wei).

## Task 1.14 — Raw JSON-RPC call

**Status**: PASS
**Approach**: Used the generic raw RPC wrapper to request the current block number.
**Command(s) used**: `alchemy --json --no-interactive rpc eth_blockNumber`
**Output quality**: Correct. Returned the current block number as hex: `0x17a19a0`.
**Notes**: None.

## Task 1.15 — Simulate a transaction

**Status**: PASS
**Approach**: Simulated a 1 ETH transfer from the supplied address to the zero address using the asset-changes simulator.
**Command(s) used**: `alchemy --json --no-interactive simulate asset-changes --tx '{"from":"0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045","to":"0x0000000000000000000000000000000000000000","value":"0xde0b6b3a7640000"}'`
**Output quality**: Correct. Returned a single native ETH transfer change for `1` ETH with `gasUsed=0x5208` and no error.
**Notes**: None.

## Task 1.16 — List available networks

**Status**: PASS
**Approach**: Listed all networks, then filtered the results to Solana entries.
**Command(s) used**: `alchemy --json --no-interactive network list`
**Output quality**: Correct. Returned 166 networks total; Solana entries were `solana-devnet` and `solana-mainnet`.
**Notes**: The response shape uses `id`, `name`, and `family` rather than a `slug` field.

## Task 1.17 — Solana RPC

**Status**: PASS
**Approach**: Used the Solana RPC wrapper with `getSlot`.
**Command(s) used**: `alchemy --json --no-interactive solana rpc getSlot`
**Output quality**: Correct. Returned current slot `410138555`.
**Notes**: None.

---

## Part 2: Tasks NOT Directly Supported by CLI

## Task 2.1 — Explain webhook setup

**Status**: PASS
**Approach**: Combined the local CLI skill's webhook command mapping with official Alchemy webhook docs covering payload shape and signature verification.
**Command(s) used**: None.
**Output quality**: Complete. Could explain setup through dashboard or CLI, the `ADDRESS_ACTIVITY` payload shape (`webhookId`, `id`, `createdAt`, `type`, `event.network`, `event.activity[]`), and HMAC-SHA256 verification against the raw request body using the `X-Alchemy-Signature` header.
**Notes**: The CLI skill alone does not include payload/signature details; official docs were needed for a full answer.

## Task 2.2 — Parse a token balance response

**Status**: PASS
**Approach**: Converted the hex balance to decimal and applied USDC's 6 decimals.
**Command(s) used**: None.
**Output quality**: Correct. `0x05f5e100` is `100000000` base units, which equals exactly `100 USDC`.
**Notes**: Straightforward computation.

## Task 2.3 — Recommend an ecosystem library

**Status**: PASS
**Approach**: Recommended a modern React dApp stack built around `wagmi` + `viem`, with a wallet UI layer such as `RainbowKit`, and described how to point transports/providers at Alchemy.
**Command(s) used**: None.
**Output quality**: Strong. A practical answer can include a default stack for external-wallet dApps and position `@account-kit/react` as the better alternative if the goal is embedded wallets and gasless UX.
**Notes**: There is no single "correct" library stack, but `wagmi` + `viem` is the cleanest default for standard React dApps using Alchemy RPC.

## Task 2.4 — Account Kit / Smart Wallets

**Status**: PASS
**Approach**: Used official Account Kit / Smart Wallet docs to explain the integration flow: create config, wrap the app in `AlchemyAccountProvider`, enable email/passkey auth, create a Gas Manager policy, and pass the policy ID for sponsorship.
**Command(s) used**: None.
**Output quality**: Complete high-level integration guidance. The answer can cover `NEXT_PUBLIC_ALCHEMY_API_KEY`, `NEXT_PUBLIC_ALCHEMY_POLICY_ID`, `createConfig(...)`, provider wiring, and `paymaster: { policyId: "..." }` for sponsored transactions.
**Notes**: Current docs emphasize "Smart Wallets" / Account Kit terminology rather than older AA-SDK naming.

## Task 2.5 — Rate limits and compute units

**Status**: PARTIAL
**Approach**: Used the official compute-units, throughput, compute-unit-costs, and pricing-plan docs to explain the CU model, method-level costs, rate limiting, and retry behavior.
**Command(s) used**: None.
**Output quality**: Strong on the model itself: CUs measure per-method cost, throughput is enforced as CU/s over a 10-second rolling token bucket window, and 429s should be retried with backoff. Specific plan-limit details are less definitive than they should be because the docs are internally inconsistent.
**Notes**: The docs clearly show examples like `eth_blockNumber = 10 CU`, `eth_call = 26 CU`, `eth_getLogs = 60 CU`, and `eth_sendRawTransaction = 40 CU / 250 throughput CU`. However, the docs conflict on free-tier throughput (`500` vs `1000` CUPS), so the exact free-tier limit is ambiguous.

## Task 2.6 — WebSocket subscriptions

**Status**: PASS
**Approach**: Used official WebSocket docs to explain both raw `eth_subscribe` usage and higher-level library usage in Viem/Ethers.
**Command(s) used**: None.
**Output quality**: Complete. Could show `wss://eth-mainnet.g.alchemy.com/v2/<API_KEY>`, `eth_subscribe` with `alchemy_pendingTransactions`, and optional `toAddress` / `fromAddress` filters, plus compare it with standard `newPendingTransactions`.
**Notes**: `alchemy_pendingTransactions` is more useful than plain `newPendingTransactions` because it can return full transaction objects and filter by address.

## Task 2.7 — Solana DAS query

**Status**: PASS
**Approach**: Queried the DAS API directly for assets owned by the supplied Solana address.
**Command(s) used**: `alchemy --json --no-interactive solana das getAssetsByOwner '{"ownerAddress":"GsbwXfJraMomNxBcjYLcG3mxkBUiyWXAB32fGbSQQRre","page":1}'`
**Output quality**: Correct. Returned an empty result set: `total=0`, `items=[]`.
**Notes**: The address appears to have no indexed NFTs/assets in DAS at run time.

## Task 2.8 — Multi-step workflow

**Status**: PASS
**Approach**: Combined `alchemy balance`, paginated `alchemy portfolio tokens --body ...` calls with metadata/prices, and raw `alchemy_getAssetTransfers` ordered descending to build a single portfolio summary.
**Command(s) used**: `alchemy --json --no-interactive balance 0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045`; `alchemy --json --no-interactive portfolio tokens --body '{"addresses":[{"address":"0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045","networks":["eth-mainnet"]}],"withMetadata":true,"withPrices":true,"includeNativeTokens":false,"includeErc20Tokens":true}'` (paginated across 20 pages); `alchemy --json --no-interactive rpc alchemy_getAssetTransfers '{"fromAddress":"0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045","category":["erc20"],"maxCount":"0x3","order":"desc","withMetadata":true}'`
**Output quality**: Complete portfolio view. ETH balance was `32.147257835992150606`; priced ERC-20 ranking over 20 pages / 2000 tokens surfaced `DINGO`, `MOODENG`, `FUMO`, `CATE`, and `KIMCHI` as the highest USD-valued holdings; the 3 most recent outgoing ERC-20 transfers were BTT sends on `2026-02-22`.
**Notes**: The ranking is dominated by spam/meme assets because the address holds thousands of them and the pricing endpoint assigns values to some of them. A production wallet UI would likely filter suspected spam or apply allowlists.

## Task 2.9 — Pagination

**Status**: PASS
**Approach**: Used the direct `--page-key` support in `tokens balances` to fetch three consecutive pages.
**Command(s) used**: Page 1: `alchemy --json --no-interactive tokens balances 0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045 --metadata`; Page 2: same command with `--page-key <page1-key>`; Page 3: same command with `--page-key <page2-key>`
**Output quality**: Correct. Successfully fetched three pages with counts `80`, `86`, and `81`, and each page returned another `pageKey`.
**Notes**: This CLI version supports direct pagination on `tokens balances`, which is a meaningful improvement over raw-RPC-only workarounds.

## Task 2.10 — Error recovery

**Status**: PASS
**Approach**: Intentionally used the wrong network slug, inspected the structured error, then corrected the slug and retried.
**Command(s) used**: `alchemy --json --no-interactive balance 0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045 -n ethereum-mainnet`; corrected: `alchemy --json --no-interactive balance 0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045 -n eth-mainnet`
**Output quality**: Excellent. The CLI returned a clear `INVALID_ARGS` error with a helpful suggestion to run `alchemy network list`, and the corrected command succeeded immediately.
**Notes**: Good developer ergonomics here.

---

## Summary

**Total**: 25.5/27
**Pass**: 24
**Partial**: 3
**Fail**: 0

### Key gaps found

- `tokens allowance` is broken in CLI `0.4.0` and requires raw `alchemy_getTokenAllowance`.
- The high-level `transfers` wrapper does not expose descending order, so "last N" outgoing transfers need raw `alchemy_getAssetTransfers`.
- Alchemy's docs are internally inconsistent on free-tier throughput (`500` vs `1000` CUPS), which makes exact rate-limit guidance less authoritative than it should be.

### Key strengths

- All core balance, token, NFT, price, block, gas, simulation, and Solana commands worked.
- `tokens balances` now supports direct `--page-key` pagination.
- Raw RPC fallbacks make it possible to recover from thin-wrapper limitations without leaving the CLI.
- Official docs are strong for webhooks, Account Kit / Smart Wallets, and WebSocket subscription flows.
