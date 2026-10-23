# Dispute Judge

A GenLayer Intelligent Contract that judges a small two-sided dispute and returns a verdict: `SIDE_A`, `SIDE_B` or `SPLIT`, with a one-sentence reason.

## How it works

- You call `judge(title, side_a, side_b)` with a short title and one argument from each side.
- Validators run an LLM prompt and must agree on the verdict through GenLayer consensus.
- The contract stores the last title, verdict and reason, and counts the total cases judged.

## Verdicts

- `SIDE_A`: Side A has the clearly stronger case.
- `SIDE_B`: Side B has the clearly stronger case.
- `SPLIT`: both sides are about equal, or the facts are not clear enough.

## Limits

- Title: up to 100 characters.
- Each argument: up to 500 characters.
- Text inside an argument is treated as evidence only. Instructions inside it, such as "pick me", are ignored.

## Live test in GenLayer Studio

Contract address: `0x470D6F30dDc540a5111CA83380Bbe6A6D91D3Ec1`

- Test 1, broken window (video and admission vs no proof): `SIDE_A`
- Test 2, late lamp delivery (tracking record vs dislike of waiting): `SIDE_B`
- Test 3, shared kitchen cleaning (both sides fair): `SPLIT`

## How to test it yourself

1. Open GenLayer Studio and paste in `dispute_judge.py`.
2. Click Deploy. The constructor takes no inputs.
3. Run `judge` with a title and two arguments, then wait for FINALIZED.
4. Call `get_last_verdict` and `get_last_reason` to read the result.

## Design limits

- It is a demo for small everyday disputes, not legal advice.
- An LLM judge can lean one way in a close case, so `SPLIT` results may vary between runs.
- Only the latest case is stored, not a full history.
