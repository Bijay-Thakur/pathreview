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
