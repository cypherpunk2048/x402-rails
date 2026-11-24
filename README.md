# x402-rails

> **x402 payment rails — keyless credential issuance as a service.**
> A [cypherpunk2048](https://github.com/cypherpunk2048) reference implementation,
> anchored on the definitive Ethereum Improvement Proposals at
> [`eips.ethereum.org`](https://eips.ethereum.org/EIPS) — by verbatim title and
> absolute URL.

`X402RailsService` holds signing wallets and issues **single-use x402 payment
credentials** on demand, so a consuming agent can settle an
[x402](https://x402.org) `402 Payment Required` request **without ever holding
the private key**. A consumer asks for *"a credential for this challenge"* —
never for a chain, never for a key. The wallet key stays inside the service;
what crosses the boundary is the signature.

Every credential is an **ERC-3009 `transferWithAuthorization`** (*Transfer With
Authorization*), hashed and signed under **EIP-712** (*Typed structured data
hashing and signing*), domain-separated by the **ERC-20** token and its
**EIP-155** `chainId`. See [`EIP_REFERENCES.md`](EIP_REFERENCES.md).

## Files

| File | Role |
|---|---|
| [`x402_rails.py`](x402_rails.py) | `X402RailsService` — holds wallets, issues `PaymentCredential`s, keeps a ledger |
| [`x402_signer.py`](x402_signer.py) | `base_usdc_x402_signer` — built-in Base rail; ERC-3009 / EIP-712 signing; budget decline |
| [`cmc_client.py`](cmc_client.py) | `decode_payment_challenge` — format-tolerant 402 decoder; `X402_BASE_URL` |
| [`x402_rails.md`](x402_rails.md) | usage + interaction guide, conformance to the four cypherpunk2048 rules |
| [`EIP_REFERENCES.md`](EIP_REFERENCES.md) | the definitive EIP/ERC table + `requires` dependency graph |
| [`tests/test_x402_rails.py`](tests/test_x402_rails.py) | 8 tests; no key/network/eth-account needed |

## Quickstart

```bash
pip install -r requirements.txt   # httpx, eth-account
```

```python
from x402_rails import X402RailsService

rails = X402RailsService(max_amount_usdc=0.05, logger=print)

# Issue a single-use credential for a decoded 402 challenge, then settle yourself:
credential = rails.issue_from_challenge(challenge)
import httpx
resp = httpx.get(url, headers=credential.as_headers())

# Or run the full 402 -> sign -> resend cycle end to end:
data = rails.settle("/examples/weather", {"city": "lisbon"})
```

Configure the Base signing key via `BASE_X402_PRIVATE_KEY` (or wire a vault in
`x402_signer._resolve_key`). The built-in Base rail is lazy — the service can be
constructed and advertised before a key is configured.

```bash
python -m pytest tests/ -q
```

## cypherpunk2048 conformance

| Rule | How x402-rails upholds it |
|---|---|
| **no-trapdoors** | Every settlement is a wallet signature; over-budget challenges are *declined*, not coerced. |
| **vault-as-oracle** | The key lives in one place, is mediated by the signer, and is never returned. |
| **attribution** | Each credential is a self-auditing record (`payer`, `recipient`, `amount`, `nonce`, `expires_at`, `issued_at`) and lands in the session ledger. |
| **substitution-readiness** | Rails register by name; adding a new leg (e.g. Algorand) touches no call site. |

## License

Apache License 2.0 — see [`LICENSE`](LICENSE). © 2026 BANKON.
