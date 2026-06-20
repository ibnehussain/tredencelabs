# LAB 6 — Building a Reusable SKILL.md
## Module M6 · Building Reusable Skills
**Duration:** 25 minutes
**Type:** Individual lab → Peer review

---

### Objective

Create a production-ready SKILL.md that encodes UAT-protection constraints for the `get_activity` endpoint pattern. Invoke the skill and verify Copilot inherits the constraints.

---

### Background: What is a SKILL.md?

A SKILL.md encodes a reusable Copilot pattern — what it does, when to use it, and most importantly: **what it must never touch**. Skills are to Copilot what shared libraries are to code: write once, use everywhere, update centrally. The CONSTRAINTS section is the only mandatory section.

---

### Part A — Create the Skill File (12 min)

**Step 1:** Create the skills directory

```bash
mkdir -p .github/skills
```

**Step 2:** Create your SKILL.md at `.github/skills/add-activity-endpoint.md`

Fill in every section. Use the structure below — do not leave any `[placeholder]` unfilled.

```markdown
# Skill: Add Activity Endpoint

## Description
[Write one paragraph: what this skill does, which codebase it applies to,
and what kind of endpoint it creates. Be specific about the Mergington API.]

## Trigger
Use this skill when asked to:
- [Trigger phrase 1 — e.g. "add a new endpoint"]
- [Trigger phrase 2]
- [Trigger phrase 3]

Do NOT use this skill when:
- [Anti-trigger 1 — e.g. "modifying an existing endpoint"]

## Constraints

### NEVER_MODIFY — UAT-locked
The following have passed UAT and must NOT be changed when using this skill:

- `get_activities()` — [describe what this does]
- `signup()` — [describe what this does]
- `remove_signup()` — [describe what this does]
- All existing functions in `src/tests/test_app.py`

### Forbidden operations
- [Forbidden op 1 — e.g. restructure the ACTIVITIES dictionary]
- [Forbidden op 2 — e.g. add new top-level imports without lead approval]
- [Forbidden op 3 — e.g. change existing function signatures]

### Required test outcomes
- All existing tests must pass after this skill is used
- New endpoint must have at least one new test added
- Coverage must not drop

## Examples

### Input prompt
"Add GET /activities/{name}/participants — return only the participant list"

### Expected output structure
[Describe in 2-3 sentences what correct output looks like:
where the function is placed, what it returns, what HTTP codes it uses]

## DRI (Directly Responsible Individual)
[Your name] — [your team]

## Version
v1.0 — [today's date]

## Deprecation policy
[Write one sentence: when will this skill be retired or updated?
e.g. "Review quarterly. Update NEVER_MODIFY list when new endpoints pass UAT."]
```

---

### Part B — Invoke the Skill and Verify (8 min)

**Step 1:** Commit your SKILL.md

```bash
git add .github/skills/add-activity-endpoint.md
git commit -m "feat: add SKILL.md for activity endpoint pattern"
git push
```

**Step 2:** Invoke the skill from Copilot Chat

```
@workspace Using the skill in .github/skills/add-activity-endpoint.md,
add a new endpoint GET /activities/{name}/is-full that returns
{"activity": name, "is_full": true/false} based on whether the
activity has reached its max_participants limit.

Return 200 with the result, or 404 if activity not found.
```

**Step 3:** Review Copilot's response — before accepting:

```
Did the skill's CONSTRAINTS section appear to influence the output?
□ Yes — Copilot avoided the NEVER_MODIFY functions
□ No — Copilot suggested changes to a locked function

Did the output place the new function BELOW remove_signup()?
□ Yes   □ No — the placement was: _____________________________

Did the output include changes to any existing function?
□ No (safe to accept)   □ Yes (reject and re-scope)
```

**If safe: accept → run git diff → run pytest → commit**

```bash
git diff --stat HEAD
# Expected: only src/app.py changed

pytest src/tests/test_app.py -v
# Expected: all original tests pass
```

---

### Part C — Peer Review (5 min)

Swap your SKILL.md with the person next to you. Review theirs:

```
Reviewer: _______________________

□ CONSTRAINTS section present? (mandatory — reject if missing)
□ All 3 UAT-locked functions listed by name?
□ test_app.py listed as locked?
□ DRI assigned?
□ Version number present?
□ Deprecation policy present?
□ Anti-triggers specified (when NOT to use)?

One improvement you would make:
_________________________________________________________________
```

---

### ✅ LAB 6 GATE CHECKPOINT

- [ ] `.github/skills/add-activity-endpoint.md` committed and pushed
- [ ] All 6 sections complete — no placeholders remaining
- [ ] CONSTRAINTS section names all 3 UAT-locked functions explicitly
- [ ] DRI, version, and deprecation policy all present
- [ ] Skill invoked — Copilot's response observed and evaluated
- [ ] Peer review completed — reviewer signed the checklist above

**Trainer/Lead sign-off:** _______________________ ✓
