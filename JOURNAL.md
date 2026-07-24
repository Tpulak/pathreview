## Week 7 — Issue selection

**Issue link:** [https://github.com/ascherj/pathreview/issues/153]

**Issue title:** [Faithfulness checker crashes when a context chunk has text: None]

**Tier:** [x] Tier 1  [ ] Tier 2  [ ] Tier 3

**Problem summary:**
In `rag/evaluator/faithfulness_checker.py`, `FaithfulnessChecker.check()` builds
context by joining each chunk's `text` with `chunk.get("text", "")`. That default
only applies when the key is missing, so a chunk like `{"text": None}` still
returns `None` and `" ".join(...)` raises a `TypeError`. A successful fix should
treat `None` (and similarly invalid values) as an empty string so faithfulness
scoring can run without crashing, which is covered by
`test_none_context_chunk_text` in `tests/unit/test_faithfulness_checker.py`.

**Branch name:** [docs/CONTRIBUTING.md]

**Setup confirmation:** [x] App runs locally at localhost:5173

**Cohort ledger:** [x] Issue added to cohort ledger