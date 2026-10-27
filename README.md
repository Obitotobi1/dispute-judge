# DisputeCourt

A multi-stage dispute resolution Intelligent Contract for GenLayer. Two parties argue a case on-chain, validators reach consensus on an AI-generated verdict, either party can appeal once with new evidence, and final outcomes build a per-address win/loss record that other contracts can read.

- **Deployed (GenLayer Studio):** `0x8587f4fFa73A55bcb679f913aD478B1AfC724Ec1`
- **Contract:** [`dispute_court.py`](./dispute_court.py)

## Lifecycle

```
OPEN --respond--> RESPONDED --judge--> JUDGED --finalize--> FINAL
                                          |
                                          +--appeal (once)--> FINAL
```

| Method | Who | Effect |
|---|---|---|
| `open_case(title, claim_a, respondent, rubric)` | anyone (becomes claimant) | creates a case in `OPEN` |
| `respond(case_id, claim_b)` | named respondent only | stores Side B, moves to `RESPONDED` |
| `judge(case_id)` | anyone | runs LLM consensus, moves to `JUDGED` |
| `appeal(case_id, new_evidence)` | claimant or respondent, once | re-judges using the prior verdict plus new evidence, moves to `FINAL` |
| `finalize(case_id)` | anyone | closes a `JUDGED` case without appeal, moves to `FINAL` |

State guards reject out-of-order calls. Role checks restrict who can respond and appeal.

## Consensus design

Each judgment returns structured JSON:

```json
{"verdict": "SIDE_A | SIDE_B | SPLIT", "confidence": 1-5, "reason": "one sentence"}
```

Validators use `gl.eq_principle.prompt_comparative` with this rule:

- `verdict` must be identical
- `confidence` may differ by at most 1
- `reason` wording is free

Safety behaviours:

- Claims, rubric and appeal text are treated as evidence only, so instructions hidden inside them are ignored (prompt-injection guard).
- Invalid or unparseable model output is normalized to `SPLIT` with confidence 1.
- On appeal the model sees the prior verdict and is told to change it only if the new evidence is material.

## Reputation records

When a case reaches `FINAL` with `SIDE_A` or `SIDE_B`, the winning address gets +1 win and the losing address +1 loss. `SPLIT` changes nothing. Addresses are stored lowercase.

## Composable read API

Other contracts can read results without running a judgment:

- `get_case(case_id)` returns the full case record as JSON
- `get_verdict(case_id)` returns `{status, verdict, confidence}`
- `get_record(address)` returns `{wins, losses}`

## Tested in GenLayer Studio

| Step | Result |
|---|---|
| `open_case` -> `respond` -> `judge` | status `JUDGED`, verdict `SIDE_B`, confidence 3 |
| `appeal` with new evidence | status `FINAL`, verdict flipped to `SIDE_A`, confidence 5 |
| `get_record` after fixing address casing | `wins: 1, losses: 1` |
| `open_case` -> `respond` -> `judge` -> `finalize` (no appeal) | status `FINAL`, verdict `SIDE_B`, confidence 4 |

## Limits

- One appeal per case.
- Claim and evidence text capped at 500 characters, title at 100, rubric at 300.
- Verdict quality depends on the LLM; `SPLIT` is the fallback when output is invalid.
- No stake, payment or enforcement logic. This is a verdict and reputation primitive.

## Run it

1. Open GenLayer Studio and create a new contract file.
2. Paste `dispute_court.py` and deploy (constructor takes no arguments).
3. Call `open_case`, then `respond`, `judge`, and `finalize` or `appeal`.
4. Read results with `get_verdict`, `get_case` and `get_record`.
