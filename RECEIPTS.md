# Receipts

Five reproducible local tests. `bash` + `jq` + 30 seconds.

## Setup

```bash
git clone https://github.com/waitdeadai/no-sycophancy
cd no-sycophancy
mkdir -p /tmp/syc-tests
```

## Test 1 — "Great question!" opener → BLOCK

```bash
printf '%s' '{"hook_event_name":"Stop","last_assistant_message":"Great question! The answer is 42."}' \
  > /tmp/syc-tests/t1.json
bash no-sycophancy.sh < /tmp/syc-tests/t1.json; echo "exit=$?"
```

Expected output:

```
BLOCKED: praise-spam at turn open.

Repair guidance:
- The operator did not ask for praise. Drop the opening compliment.
- Lead with the substantive answer, not validation.
- Anthropic's own measurements: sycophancy in 9% of guidance-seeking chats.
  AAAI 2026 co-creativity study: 91.7% prevalence.
- If the operator did request praise/encouragement and the hook misfired,
  restate the request in the next turn so the allow-clause matches
  (e.g. start with 'You asked for encouragement — here's...').
exit=2
```

## Test 2 — substantive opener → ALLOW

```bash
printf '%s' '{"hook_event_name":"Stop","last_assistant_message":"The answer is 42 because the deeper question is..."}' \
  > /tmp/syc-tests/t2.json
bash no-sycophancy.sh < /tmp/syc-tests/t2.json; echo "exit=$?"
```

Expected: `exit=0`

## Test 3 — "You're absolutely right" opener → BLOCK

```bash
printf '%s' "{\"hook_event_name\":\"Stop\",\"last_assistant_message\":\"You're absolutely right, I should have caught that earlier.\"}" \
  > /tmp/syc-tests/t3.json
bash no-sycophancy.sh < /tmp/syc-tests/t3.json; echo "exit=$?"
```

Expected: BLOCKED, exit=2.

## Test 4 — operator-requested encouragement (allow-clause) → ALLOW

```bash
printf '%s' '{"hook_event_name":"Stop","last_assistant_message":"You asked for encouragement — here is a recap of what is going well."}' \
  > /tmp/syc-tests/t4.json
bash no-sycophancy.sh < /tmp/syc-tests/t4.json; echo "exit=$?"
```

Expected: `exit=0`. The allow-clause overrides any praise vocabulary that follows.

## Test 5 — substantive use of praise vocabulary (not at open) → ALLOW

```bash
printf '%s' '{"hook_event_name":"Stop","last_assistant_message":"That is a great question and the math is straightforward."}' \
  > /tmp/syc-tests/t5.json
bash no-sycophancy.sh < /tmp/syc-tests/t5.json; echo "exit=$?"
```

Expected: `exit=0`. The hook requires the contraction *That's* (not *That is*) for the trigger, and even then it requires a praise modifier word — the substantive *"That is a great question and the math is..."* with no contraction passes through.

## Summary table

| # | Scenario | Expected | Actual | Exit |
|---|----------|----------|--------|------|
| 1 | "Great question!" opener | BLOCK | BLOCKED | 2 |
| 2 | Substantive opener | ALLOW | (silent) | 0 |
| 3 | "You're absolutely right" opener | BLOCK | BLOCKED | 2 |
| 4 | Operator-requested encouragement (allow-clause) | ALLOW | (silent) | 0 |
| 5 | Substantive use of praise vocabulary | ALLOW | (silent) | 0 |

Five for five.
