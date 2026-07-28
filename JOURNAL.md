# PathReview Journal

## Week 7 — Issue selection

**Issue link:** https://github.com/ascherj/pathreview/issues/153

**Issue title:** Faithfulness checker crashes when a context chunk has `text: None`

**Tier:** [x] Tier 1  [ ] Tier 2  [ ] Tier 3

**Problem summary:**
The faithfulness checker is the piece of the RAG evaluator that decides how much of the generated feedback is actually backed by the retrieved context. Before it can score anything, it stitches all the context chunks together into one string. The bug is in how it pulls the text out of each chunk. It uses `chunk.get("text", "")`, which people usually assume hands back an empty string when there is no text. That default only kicks in when the key is missing entirely. If a chunk comes through as `{"text": None}`, the key is present, so `.get` happily returns `None`, and the `" ".join(...)` right after it blows up with a TypeError. A good fix makes the checker treat a missing value and a `None` value the same way, so a single malformed chunk no longer takes down the whole evaluation, and I want to add a regression test so this exact case stays covered.

**Branch name:** fix/153-faithfulness-none-text-chunk

**Setup confirmation:** [x] App runs locally at localhost:5173

**Cohort ledger:** [x] Issue added to cohort ledger

---

### "Is this right for me?" checklist reasoning

I spent some time reading the actual code before committing to this one, and that changed how confident I felt. The whole thing lives in a single file, `rag/evaluator/faithfulness_checker.py`, and the failure is a one line assumption about how `dict.get` behaves. That felt honest for a first contribution. I am not touching the database, the API routes, or the frontend, so the blast radius is small and I can reason about the fix without holding the entire system in my head.

What sold me was that there is already a test file at `tests/unit/test_faithfulness_checker.py` and the issue even points to a named test for this case. That means I can reproduce the crash quickly, watch it fail, and then watch it pass, which is the kind of tight feedback loop I wanted while I am still learning the repo. I could also explain the root cause out loud, which the checklist treats as a real signal that you understand the problem rather than just pattern matching a fix.

The one thing I want to stay careful about is scope creep. It would be tempting to start cleaning up the claim extraction logic while I am in there, but that is not what the issue asks for, so I am going to keep the change focused on the None handling and the test that proves it.

## Week 8 — Reproduction & solution planning
**Reproduction commit link:**  [https://github.com/Yd025/ai201-pathreview/commit/a56837386879814f18742d96974c63b0fb1efcf1]

**Reproduction summary:**
I reproduced the crash two ways. First I called the checker directly with `FaithfulnessChecker().check('Knows Python.', [{'text': None}])` and watched it raise `TypeError: sequence item 0: expected str instance, NoneType found`. Then I ran the existing unit test `test_none_context_chunk_text`, which fails on the same line, `rag/evaluator/faithfulness_checker.py:34`, so I know exactly where the bug lives and I have a test that will flip to green once I fix it.

**PLAN.md link:** https://github.com/Yd025/ai201-pathreview/blob/fix/153-faithfulness-none-text-chunk/PLAN.md

**Blockers or open questions:**
My one open question going into Week 9 is whether null text should ever reach the evaluator at all, or whether something further up in the retriever is letting bad chunks through. My fix makes the checker resilient either way, but I want to grep the retriever and generator before I decide the null guard is the whole story.

## Week 9 — Solution building & PR submission

### Check-in 1 (mid-week)

**Current progress:**
I have the fix drafted locally and the null test passing. Before I touched anything I ran the full unit suite and wrote down the baseline, which was 53 failing tests across the repo that have nothing to do with my issue. The change I am leaning toward is swapping the context concatenation in `faithfulness_checker.py` from `chunk.get("text", "")` to `chunk.get("text") or ""`, which lines up with sub-tasks 1 through 3 of my PLAN. With that in place `test_none_context_chunk_text` goes green for me, and I am sketching two more regression tests, one for a null chunk mixed in with a good chunk and one for an all null context list, before I commit the code and open the PR.

**Next steps:**
I want to close the loop on my open question from Week 8 and grep the retriever and generator to see where a null text value could come from, then finish the PR description and run make check one more time before I mark it ready.

**Blockers:**
None that are stopping me. The wrinkle I hit was that three other tests in the same file fail because of a separate scoring bug, so I had to be careful that my new mixed chunk test asserted on not crashing rather than on a specific score, since the score path is broken for a different reason.

---

### Check-in 2 (end of week)

**PR link:** [link to your submitted pull request]

**Branch:** [the branch name you worked on, e.g. `fix/123-short-description`]

**What you built:**
[1–3 sentences summarizing what your fix does and how it works]

**Tests added or updated:**
[Which test files did you touch? What do they cover?]

**Self-review confirmation:** [ ] make check passes  [ ] make test-unit passes

**Draft PR feedback received from:** [name or Slack handle, or "none"]

