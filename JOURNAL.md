# Journal

## Week 7 — Issue selection

**Issue link:** https://github.com/ascherj/pathreview/issues/153

**Issue title:** Faithfulness checker crashes when a context chunk has `text: None`

**Tier:** [x] Tier 1  [ ] Tier 2  [ ] Tier 3

**Problem summary:**
The `FaithfulnessChecker.check()` method in `rag/evaluator/faithfulness_checker.py` builds a single context string by joining the `text` field from each retrieved context chunk, falling back to `""` when a chunk has no `text` key. The bug is that `dict.get("text", "")` only applies its default when the key is *missing* — if a chunk explicitly has `text: None`, `.get()` returns `None` instead of an empty string, and passing that into `" ".join(...)` raises a `TypeError`. This affects the RAG evaluation pipeline, which is used to score how well generated feedback is supported by retrieved context. A successful fix would treat a `None` text value the same as a missing one (i.e., normalize it to `""` before joining) so the checker degrades gracefully instead of crashing, and would make the existing `test_none_context_chunk_text` test pass.

**Branch name:** 153-RAGfaithfulnessCHecker

**Setup confirmation:** [ ] App runs locally at localhost:5173

**Cohort ledger:** [x] Issue added to cohort ledger

## Local reproduction notes

Reproduced locally by invoking the faithfulness checker with a context chunk whose text field is explicitly `None`:

```python
from rag.evaluator.faithfulness_checker import FaithfulnessChecker

checker = FaithfulnessChecker()
checker.check("Has Python skills", [{"text": None}])
```

Observed behavior: the code reaches the context join step and raises `TypeError: sequence item 0: expected str instance, NoneType found` because `None` is not normalized to `""` before joining.

## Week 8 — Reproduction & solution planning

**Reproduction commit link:** <https://github.com/Bijay-Thakur/pathreview/commit/e8b421c143ef52ad33433b45b3dd324e9fef8a69>

**Reproduction summary:**
Reproduced by calling `FaithfulnessChecker().check("Has Python skills", [{"text": None}])` directly, which raises `TypeError: sequence item 0: expected str instance, NoneType found` at the `" ".join(...)` step, since `chunk.get("text", "")` returns `None` (not the default) when the key exists but is explicitly `None`.

**PLAN.md link:** <https://github.com/Bijay-Thakur/pathreview/blob/153-RAGfaithfulnessCHecker/PLAN.md>

**Walkthrough video (recommended):** <https://www.loom.com/share/4765631dfa014718a4bde88d5904fd3b>

## Week 9 — Solution building & PR submission

### Check-in 1 (mid-week)

**Current progress:**
Implemented the fix in `rag/evaluator/faithfulness_checker.py`: the context-join step now uses `chunk.get("text") or ""` instead of `chunk.get("text", "")`, so an explicit `text: None` is normalized to an empty string instead of crashing the `" ".join(...)` call. The existing regression test `test_none_context_chunk_text` in `tests/unit/test_faithfulness_checker.py` now passes. Ran the full unit suite to confirm no regressions — the only other failures in that test file (partial/multi-chunk scoring tests) are pre-existing and unrelated to this bug, confirmed by checking them against the pre-fix baseline. All sub-tasks from PLAN.md are complete.

**Next steps:**
Finish self-review against `docs/CONTRIBUTING.md` conventions, open the PR, and fill out the PR template.

**Blockers:**
None. Local environment was missing dependencies (`structlog` and other project extras) needed to run the test suite; resolved by installing them.

---

### Check-in 2 (end of week)

**PR link:** <https://github.com/ascherj/pathreview/pull/994>

**Branch:** `153-RAGfaithfulnessCHecker`

**What you built:**
Fixed a crash in `FaithfulnessChecker.check()` where a context chunk with an explicit `text: None` value raised a `TypeError`. Changed `chunk.get("text", "")` to `chunk.get("text") or ""` when concatenating context chunk text, since `.get()`'s default only applies when the key is missing, not when it's present but `None`.

**Tests added or updated:**
`tests/unit/test_faithfulness_checker.py` — the regression test `test_none_context_chunk_text` already existed for this case and now passes; no new test was needed since it was already in place but failing before the fix.

**Self-review confirmation:** [x] make check passes (scoped to the touched file — repo-wide lint/format debt is pre-existing and unrelated)  [x] make test-unit passes (target test now passes; other pre-existing failures elsewhere in the suite are unrelated to this change)

**Draft PR feedback received from:** none yet
