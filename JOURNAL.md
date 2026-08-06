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

**Reproduction commit link:** [https://github.com/Tpulak/pathreview/commit/2b41309001a00d4b80cb6026a47fc02eef9a2f22]

**Reproduction summary:**
Traced `FaithfulnessChecker.check()` in `rag/evaluator/faithfulness_checker.py`: when a context chunk is `{"text": None}`, `chunk.get("text", "")` returns `None` (key present), so `" ".join(...)` raises `TypeError: sequence item 0: expected str instance, NoneType found`. The same case is covered by the existing failing test `test_none_context_chunk_text` in `tests/unit/test_faithfulness_checker.py`.

**PLAN.md link:** [https://github.com/Tpulak/pathreview/blob/fix/153-faithfulness-checker-none-text/PLAN.md]

**Blockers or open questions:**
Need project deps installed locally to run the unit test / repro snippet end-to-end (`structlog` missing in bare `python`). Root cause and fix location are clear from code inspection; Week 9 fix should be a small coercion before the join.

## Week 9 — Solution building & PR submission

### Check-in 1 (mid-week)

**Current progress:**
Implemented the `None`-safe context join in `FaithfulnessChecker.check()` per PLAN.md
(sub-tasks 1–2): use `(chunk.get("text") or "")` so missing and `None` text both
become empty strings before `" ".join(...)`. Confirmed the issue repro snippet no
longer raises and `test_none_context_chunk_text` passes.

**Next steps:**
Run full `make check` / `make test-unit`, document any pre-existing failures,
open the PR against upstream, and fill Check-in 2 with the PR link.

**Blockers:**
None for the fix itself. `gh` CLI was not available locally for PR creation at first.

---

### Check-in 2 (end of week)

**PR link:** [REPLACE_AFTER_PR — paste GitHub PR URL here]

**Branch:** [fix/153-faithfulness-checker-none-text]

**What you built:**
Fixed a crash in the RAG faithfulness checker when a retrieved context chunk has
`"text": None`. `dict.get("text", "")` does not substitute the default when the
key is present with a `None` value, so the join now coerces falsy/`None` text to
`""` and returns a normal float score instead of raising `TypeError`.

**Tests added or updated:**
No new test file needed — existing coverage in
`tests/unit/test_faithfulness_checker.py` (`test_none_context_chunk_text`,
`test_missing_text_key_in_chunk`) already asserts graceful handling. Both pass
after the fix.

**Self-review confirmation:** [x] make check passes (for changed file; see notes)  [x] make test-unit passes for #153 tests (suite has documented pre-existing failures unrelated to this change)

**Draft PR feedback received from:** [none]

**Pre-existing failures note:**
Full `pytest tests/unit -m unit` reported many failures outside this issue
(e.g. bias_detector, pii_scrubber, resume_parser, review_service, and three
unrelated faithfulness scoring assertion tests). This change only touches the
context-text join in `faithfulness_checker.py` and makes
`test_none_context_chunk_text` pass; it does not introduce those other failures.
`mypy rag/evaluator/faithfulness_checker.py` is clean; `eval_suite.py` has a
pre-existing untyped-def error unrelated to this fix.
