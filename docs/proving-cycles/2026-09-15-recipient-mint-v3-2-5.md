# Proving cycle: recipient mint on xtrata v3.2.5 (testnet)

**Date:** 2026-09-15 · **Contract:** `ST7KN0NNFMEJVKD8AX54QSZ71GA9Q6HY8T1BFHK3.xtrata-v3-2-5-test1` (Stacks testnet) · **Thread:** #3 · **Findings filed:** #13, #14

This is the record of DeOrganized's first-pass proving run against the recipient-mint functions delivered in v3.2.5 (requested in #1). It exists so the next integrator can run the same cycle — or skip the parts we already paid for. Every number below is a measurement from the run, not arithmetic.

## What the run set out to prove

The recipient-mint functions (`mint-single-tx-to`, `mint-single-tx-recursive-to`, `mint-single-tx-with-relationships-to`) take a recipient as the first argument. The publisher pays; the recipient becomes owner **and** recorded creator. The claims worth proving on-chain, rather than reading from the source:

1. Payer and recipient are distinct principals in every on-chain record.
2. The recipient holds zero STX throughout, signs nothing, and is allowlisted nowhere.
3. Fees are quoted at runtime and bounded by a deny-mode `<=` post-condition; the miner fee is predictable.
4. The token id comes from the confirmed transaction's own `{token-id, existed}` result — never from a hash lookup.
5. A retry (same idempotency key, identical bytes) reconciles to the existing record and broadcasts nothing.
6. An intentional duplicate (new key, identical bytes) mints a new token and pays a second fee; `get-id-by-hash` afterwards names the first token only.

Recursive, relationship, and parent-delegation paths were out of scope for this pass.

## Setup

| Item | Value |
|---|---|
| Publisher (payer) | `STY8JZN46DRC0ZDQV7EKWPJY8644VTE8B4SXTWCD` — faucet-funded, direct calls, standard address |
| Recipient | `ST17DTV5D2TB2V13GA56HJF0X8S027179VN60GJX` — 0 STX before and after; never signed; not allowlisted |
| Contract state | `is-paused` → `(ok false)`; no allowlist entry required while unpaused |
| Payload | 1,442 bytes of markdown, 1 chunk, mime `text/plain` |
| Content sha256 | `1c694acaa5e53a208a7ef908c7ef5f467b407fd8ca4702eeb575d212e990d37c` |
| Chain fold (`expected-hash`) | `d135f6db34111417e338dce7fcadbe8e000879b2d926074d1be7bf5646ee275b` — confirmed fresh (`get-id-by-hash` → none) before the run |
| `token-uri` | `data:,deorganized-proving-run-20260915` (38 chars) |
| Quote | `quote-single-tx-fee(1442, 1)` → `total-fee` 11,000 µSTX, `single-tx-eligible` true |
| Fee schedule read | `single-tx-fee-unit` 10,000 · `upload-chunk-fee-unit` 1,000 · `get-chunk-size` 16,384 |

The quote is chunk-count driven: 1,024 through 16,383 bytes all quoted 11,000 at one chunk.

## Results

| | Token 1005 (MINT-1) | Token 1006 (DUPLICATE) |
|---|---|---|
| txid | `69f08b1a455cb447ea5aa49cdecae108bd091022d253e2c03acce1acfd2e7316` | `44930620cc9457179ec8bc8767ef87625688e01aaf9873c0637b9bf421a1c9d2` |
| block / status | 382372 / success | 382422 / success |
| `tx_result` | `(ok (tuple (existed false) (token-id u1005)))` | `(ok (tuple (existed false) (token-id u1006)))` |
| `get-owner` | recipient | recipient |
| `get-inscription-creator` | recipient | recipient |
| `get-inscription-meta` | `text/plain` / 1442 / 1 chunk / fold `0xd135f6db…275b` / sealed | identical, same fold |
| `get-token-uri` | `data:,deorganized-proving-run-20260915` | same |
| post-condition | deny mode, principal = publisher, `sent_less_than_or_equal_to` 11,000 | same |
| `fee_rate` (miner) | 1,887 µSTX | 1,887 µSTX |
| `stx_asset` event | publisher → contract, 11,000 | publisher → contract, 11,000 |
| `non_fungible_token_asset` | mint → recipient, u1005 | mint → recipient, u1006 |
| `smart_contract_log` | `inscription-sealed`, payer = publisher, owner = recipient, token-id 1005 | same shape, token-id 1006 |

Token identity, in the form requested in #3:

```json
[
  {"network": "testnet", "contractId": "ST7KN0NNFMEJVKD8AX54QSZ71GA9Q6HY8T1BFHK3.xtrata-v3-2-5-test1", "tokenId": "1005"},
  {"network": "testnet", "contractId": "ST7KN0NNFMEJVKD8AX54QSZ71GA9Q6HY8T1BFHK3.xtrata-v3-2-5-test1", "tokenId": "1006"}
]
```

**Retry (same key, identical bytes):** returned the original record (same txid, token 1005) and broadcast nothing — publisher nonce and balance unchanged. This proves reconciliation at the chain level. It does **not** prove node-level rejection of identical signed bytes; our integration didn't retain the signed transaction after broadcast, so there was nothing to re-broadcast. Stated as partial in #3.

**Recipient mismatch (same key, different recipient):** refused by our integration before any broadcast. An idempotency key bound to one recipient cannot be replayed to hand that recipient's token to another caller. This is an integrator-side guard, not a contract feature — but it's one every recipient-mint integration needs.

**Hash lookup after the duplicate:** `get-id-by-hash(0xd135f6db…275b)` → `(some u1005)`. Two tokens carry that fold; the index names the first and is blind to the second. This is the behavior documented in #5, observed on v3.2.5. Consequence for integrators: once recipients can vary, `get-id-by-hash` is unsafe both as a dedup pre-flight (it would absorb a legitimate mint to a second recipient into the first one's token) and as a post-mint token-id recovery (it can return the wrong token). Take the id from `tx_result`.

## Balances

| Point | Publisher (µSTX) | Δ |
|---|---|---|
| Start | 500,000,000 | |
| After aborted attempt (see below) | 499,998,153 | −1,847 (miner fee only) |
| After MINT-1 | 499,985,266 | −12,887 (11,000 + 1,887) |
| After DUPLICATE | 499,972,379 | −12,887 |
| **Total** | | **27,621** |

Recipient: 0 → 0.

## Fee prediction

The miner fee is deterministic from the serialized transaction length: `fee_rate = ceil(serialized_bytes × multiplier)`. With a 1,797-byte serialized transaction and a 1.05 multiplier, the prediction was 1,887 µSTX; both mints charged exactly 1,887. An earlier serialization without a `token-uri` was 1,759 bytes → 1,847, also exact. Worth building into any cost model: the protocol fee is quoted, the miner fee is computable, and the sum is what the absolute ceiling must cover.

## The abort that produced #13

The first MINT-1 attempt sent `token-uri ""` and aborted:

- tx `412807e3703b5772c1c09266d24eda8387e89873e6d0c0124c2b913b7e870424`, block 382248, `abort_by_response`, `(err u107)`, `event_count 0`.
- The contract asserts `(> (len token-uri-string) u0)`; the function signature doesn't indicate the constraint.
- The deny post-condition held: the protocol fee was never transferred. The miner fee (1,847) was paid, as it is on every mined abort.
- The idempotency key was reused for the successful attempt; a terminal failed record must not satisfy replay, and the second attempt broadcast a fresh transaction.

Guidance: validate `token-uri` non-empty (≤ 256) before quoting or signing. Filed as #13.

## The width that would have produced a second abort (#14)

`mint-single-tx-to` accepts `mime` as `(string-ascii 64)`; the stored meta field is `(string-ascii 10)`. `text/markdown` (13 characters) passes the argument boundary and fails inside execution, after the fee. `text/plain` (10) and `text/html` (9) fit. We read this from the source before the run rather than discovering it on-chain. Filed as #14.

## Reproducing the cycle

1. Read `is-paused`, the fee units, and `quote-single-tx-fee(size, chunks)` at runtime. Record the quote.
2. Record the publisher's `possible_next_nonce`, balance, and `get-next-token-id`. Record the recipient's balance.
3. Confirm the payload's fold is fresh (`get-id-by-hash` → none) so `existed` can only be false.
4. Build `mint-single-tx-to(recipient, expected-hash, mime, total-size, chunks, token-uri)` with a **non-empty** `token-uri` and a mime ≤ 10 characters. Deny-mode post-condition on the publisher: STX `<=` the quote. Refuse to sign if quote + estimated miner fee exceeds your own absolute ceiling.
5. Broadcast; wait for `tx_status success`; decode `tx_result` for `{token-id, existed}`. Verify `get-owner`, `get-inscription-creator`, `get-inscription-meta`, the three events, the post-condition on the transaction, and both balances.
6. Retry with the same key and bytes: expect your integration to reconcile without broadcasting (nonce unchanged).
7. Mint again with a new key and identical bytes: expect a new token, `existed false`, a second charge, and `get-id-by-hash` still naming the first.

Authenticated reads against a private signer over a CLI that strips quotes: `curl --oauth2-bearer $TOKEN <url>` carries the bearer without any quoted argument.

## Still open

- Stale-quote negative test (#3): coordinated fee raise between quote and broadcast; expect abort with miner fee paid, no protocol payment, no token.
- Node-level identical-signed-bytes dedup: requires retaining signed bytes after broadcast.
- Whether `existed` is vestigial on single-tx under v3.2.5 (it was `false` on both mints, including the duplicate).
- Recursive / relationships / parent-delegation paths: a later pass.
