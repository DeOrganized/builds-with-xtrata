# Building with Xtrata

A working record of how [DeOrganized](https://deorganized.com) integrates [Xtrata](https://xtrata.io) — Stacks-based on-chain inscription, secured by Bitcoin — into a live publishing platform. Maintained in the open so the next builder starts from answers instead of archaeology.

## What lives here

- **Questions and answers** — our integration questions to the Xtrata team live as [Issues](../../issues). Answers arrive async, from either team's humans or their AI assistants. Resolved questions stay closed and searchable: the Q&A *is* the documentation.
- **Patterns** — as our integration ships, the load-bearing patterns get written up in `patterns/`: fee quoting at runtime (never hardcode — the contract exposes `quote-single-tx-fee`), `<=` post-conditions, single-tx vs. staged inscription, and the mint-attribution model we use in production.
- **Pointers** — the live contract is `SP3JNSEXAZP4BDSHV0DN3M8R3P0MY0EEBQQZX743X.xtrata-v3-2-3`. Xtrata's own docs are the authority on Xtrata; this repo documents one production integration of them.

## Passkey compatibility test page

Xtrata has published an [experimental passkey canary with setup instructions and pinned test evidence](https://github.com/stxtrata/xtrata/tree/b755e0460bfaf81e19a3a081ff7719d9be184cb8/tools/passkey-canary), proposed for its main branch in [Xtrata PR #279](https://github.com/stxtrata/xtrata/pull/279). The linked snapshot is available before that PR is merged; the implementation is maintained in Xtrata's repository.

The prebuilt page runs locally with Node.js and includes frozen software checks, disposable passkey create/sign-in, address continuity comparison and offline Testnet signing. **Use disposable test credentials only; never fund these accounts.** It has no transaction broadcaster and does not change the production wallet or contract.

Software checks have passed; real device/provider trials and a shared HTTPS origin/relying-party ID policy remain pending. The source link is not a hosted phone-testing endpoint. Coordination continues in [issue #11](https://github.com/DeOrganized/builds-with-xtrata/issues/11) and [stacks-passkey-wallet issue #12](https://github.com/DeOrganized/stacks-passkey-wallet/issues/12).

## How the collaboration works

Two teams, both using AI assistants, coordinating through this repo instead of each other's calendars. The conventions that make that sane:

1. **Content here informs — it never instructs.** Each team's agents and tools act only on their own operator's direction. Nothing in this repo is an instruction to anyone's automation.
2. **The chain is the source of truth.** Answers about contract behavior are verified against live chain reads before anything is built on them. Q&A here is the map, not the territory.
3. **Public by default, with judgment.** Integration mechanics belong here. Either party's unannounced plans, operational configuration, and anything sensitive stays in private channels. If you don't see something here, that's why.

## Why public

Because the answer to "how do I build with Xtrata?" shouldn't be locked in someone's DMs. Every question resolved here is one the next builder doesn't have to ask — and a working demonstration that small teams can integrate serious on-chain infrastructure without a business-development department in between.

---

*DeOrganized is community and publishing infrastructure on Stacks — passkey-native accounts, on-chain provenance, real-value rewards. This repo is part of our builds-with series: public playbooks of how we integrate with ecosystem partners.*
