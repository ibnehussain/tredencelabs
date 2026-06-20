# LAB 7 — Eval-Driven Development
## Module M7 · Eval-Driven Development (EDD)
**Duration:** 30 minutes
**Type:** Individual lab

---

### Objective

Write a `promptfooconfig.yaml` that validates your Copilot prompt before you use it in production. Add the eval gate to GitHub Actions so it blocks PRs automatically.

---

### Background: Why EDD?

Manual review fails in three ways: inconsistency, speed pressure, and lag. EDD flips the order — you define what "correct" looks like **before** writing the code. The pipeline enforces it automatically, on every PR, regardless of whether anyone remembers.

**The EDD principle:** Write the eval before you build the system. If the eval passes, the system is correct. If it fails, the PR is blocked.

---

### Part A — Write Your First Eval (15 min)

**Step 1:** Install promptfoo

```bash
npm install -g promptfoo
npx promptfoo --version    # verify installation
```

**Step 2:** Create `promptfooconfig.yaml` in the repo root

This eval tests the prompt you use to add new activity endpoints — the same type of prompt you used in Labs 4 and 6.

```yaml
# promptfooconfig.yaml
prompts:
  - |
    Add a new Flask route function called get_activity_count to src/app.py.

    Task: Implement GET /activities/<string:activity_name>/count that returns
    {"activity": name, "participant_count": N} with HTTP 200, or
    {"error": "Activity not found"} with HTTP 404 if not found.

    Scope: Add this function BELOW remove_signup(). Do NOT modify
    get_activities(), signup(), or remove_signup(). Do NOT touch
    src/tests/test_app.py or src/static/.

    Constraint: Use jsonify() for all responses. Return HTTP 200 or 404 only.
    Do not add new import statements. Do not change the ACTIVITIES dictionary.

    Format: Single function named get_activity_count with a one-line docstring.

providers:
  - openai:gpt-4o

tests:

  - description: "Output contains the correct function name"
    assert:
      - type: contains
        value: "def get_activity_count"

  - description: "Output uses the correct route path"
    assert:
      - type: contains
        value: "/count"

  - description: "Output does NOT modify get_activities — UAT-locked"
    assert:
      - type: not-contains
        value: "def get_activities"

  - description: "Output does NOT modify signup — UAT-locked"
    assert:
      - type: not-contains
        value: "def signup"

  - description: "Output does NOT modify remove_signup — UAT-locked"
    assert:
      - type: not-contains
        value: "def remove_signup"

  - description: "Output uses jsonify"
    assert:
      - type: contains
        value: "jsonify"

  - description: "Output returns 404 for not-found activities"
    assert:
      - type: contains
        value: "404"
```

**Step 3:** Run the eval

```bash
npx promptfoo eval
```

Record the results:

```
Total assertions: ______
Assertions passed: ______
Assertions failed: ______
Overall pass rate: ______%

Which assertions failed (if any)?
_________________________________________________________________

Did any not-contains assertion fail?  □ Yes (scope violation!)  □ No
```

**Step 4:** If any assertion failed — fix the prompt and re-run

A failing `not-contains` assertion means your prompt would let Copilot touch a UAT-locked function. Tighten the Scope section: name the functions explicitly, add "Return ONLY the new function in your output."

Re-run until all 7 assertions pass (100% pass rate).

```
Final pass rate after fixing: ______%
Number of prompt iterations needed: ______
```

---

### Part B — Add the Eval Gate to GitHub Actions (10 min)

**Step 1:** Create `.github/workflows/eval.yml`

```yaml
name: Copilot Eval Gate

on: [pull_request]

jobs:
  eval:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Setup Node
        uses: actions/setup-node@v3
        with:
          node-version: "18"

      - name: Install promptfoo
        run: npm install -g promptfoo

      - name: Run eval assertions
        env:
          OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}
        run: promptfoo eval --output results.json

      - name: Check pass rate ≥ 80%
        run: |
          python3 -c "
          import json, sys
          with open('results.json') as f:
              d = json.load(f)
          s = d['results']['stats']
          rate = s['successes'] / s['total'] * 100
          print(f'Eval pass rate: {rate:.1f}% ({s[\"successes\"]}/{s[\"total\"]})')
          if rate < 80:
              print('BLOCKED: Pass rate below 80% threshold.')
              sys.exit(1)
          print('PASSED: Eval gate cleared.')
          "
```

**Step 2:** Commit and push everything

```bash
git add promptfooconfig.yaml .github/workflows/eval.yml
git commit -m "ci: add promptfoo eval gate — blocks PR if pass rate < 80%"
git push
```

**Step 3:** Verify the gate works with a test PR

Create a branch with a deliberately broken prompt (change the `not-contains` assertion value to something that WILL be in the output):

```bash
git checkout -b test/eval-gate-demo
```

Edit `promptfooconfig.yaml`. In the assertion for "does NOT modify get_activities", change `not-contains` to `contains`. This forces the assertion to look for the UAT-locked function name, which will fail:

```yaml
  - description: "DEMO FAILURE — triggers the gate"
    assert:
      - type: contains
        value: "this_string_will_never_appear_xyz"
```

Push and open a PR:

```bash
git add promptfooconfig.yaml
git commit -m "demo: trigger eval gate failure"
git push -u origin test/eval-gate-demo
```

Open a PR from this branch on GitHub.

```
Did the Actions tab show the eval gate step failing?  □ Yes  □ No
What was the exact message shown in the CI log?
_________________________________________________________________

Was the PR blocked from merging?   □ Yes — working correctly
                                    □ No — check workflow YAML
```

**Restore and clean up:**

```bash
git checkout main
git branch -d test/eval-gate-demo
git push origin --delete test/eval-gate-demo
```

---

### Part C — Add One More Assertion (5 min)

Add a fourth dimension to your eval: verify the output does NOT touch `src/tests/test_app.py`. This catches test-file scope violations automatically.

Add this assertion to your `promptfooconfig.yaml`:

```yaml
  - description: "Output does NOT include test file modifications"
    assert:
      - type: not-contains
        value: "test_app"
```

Commit and push:

```bash
git add promptfooconfig.yaml
git commit -m "ci: add test-file scope assertion to eval gate"
git push
```

---

### ✅ LAB 7 GATE CHECKPOINT

- [ ] `promptfooconfig.yaml` created with at least 7 assertions (3 `not-contains`)
- [ ] `npx promptfoo eval` runs locally and achieves 100% pass rate
- [ ] `.github/workflows/eval.yml` committed and pushing to `main` triggers the gate
- [ ] Demo PR confirmed blocked by the eval gate (screenshot or CI log noted)
- [ ] PR cleaned up — demo branch deleted
- [ ] Custom assertion added for test-file scope

**Trainer/Lead sign-off:** _______________________ ✓
