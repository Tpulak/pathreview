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

**Branch name:** [fix/153-faithfulness-checker-none-text]

**Setup confirmation:** [x] App runs locally at localhost:5173

**Cohort ledger:** [x] Issue added to cohort ledger

## Week 8 — Reproduction & solution planning

**Reproduction commit link:** [(https://github.com/Tpulak/pathreview/commit/2b41309001a00d4b80cb6026a47fc02eef9a2f22)]

**Reproduction summary:**
Traced `FaithfulnessChecker.check()` in `rag/evaluator/faithfulness_checker.py`: when a context chunk is `{"text": None}`, `chunk.get("text", "")` returns `None` (key present), so `" ".join(...)` raises `TypeError: sequence item 0: expected str instance, NoneType found`. The same case is covered by the existing failing test `test_none_context_chunk_text` in `tests/unit/test_faithfulness_checker.py`.

**PLAN.md link:** [https://github.com/Tpulak/pathreview/blob/fix/153-faithfulness-checker-none-text/PLAN.md]


**Blockers or open questions:**
Need project deps installed locally to run the unit test / repro snippet end-to-end (`structlog` missing in bare `python`). Root cause and fix location are clear from code inspection; Week 9 fix should be a small coercion before the join.
