# PathReview Journal

## Week 7 — Issue selection

**Issue link:** https://github.com/ascherj/pathreview/issues/153

**Issue title:** Faithfulness checker crashes when a context chunk has `text: None`

**Tier:** [x] Tier 1  [ ] Tier 2  [ ] Tier 3

**Problem summary:**
The faithfulness checker is the piece of the RAG evaluator that decides how much of the generated feedback is actually backed by the retrieved context. Before it can score anything, it stitches all the context chunks together into one string. The bug is in how it pulls the text out of each chunk. It uses `chunk.get("text", "")`, which people usually assume hands back an empty string when there is no text. That default only kicks in when the key is missing entirely. If a chunk comes through as `{"text": None}`, the key is present, so `.get` happily returns `None`, and the `" ".join(...)` right after it blows up with a TypeError. A good fix makes the checker treat a missing value and a `None` value the same way, so a single malformed chunk no longer takes down the whole evaluation, and I want to add a regression test so this exact case stays covered.

**Branch name:** fix/153-faithfulness-none-text-chunk

**Setup confirmation:** [x] App runs locally at localhost:5173

**Cohort ledger:** [ ] Issue added to cohort ledger

---

### "Is this right for me?" checklist reasoning

I spent some time reading the actual code before committing to this one, and that changed how confident I felt. The whole thing lives in a single file, `rag/evaluator/faithfulness_checker.py`, and the failure is a one line assumption about how `dict.get` behaves. That felt honest for a first contribution. I am not touching the database, the API routes, or the frontend, so the blast radius is small and I can reason about the fix without holding the entire system in my head.

What sold me was that there is already a test file at `tests/unit/test_faithfulness_checker.py` and the issue even points to a named test for this case. That means I can reproduce the crash quickly, watch it fail, and then watch it pass, which is the kind of tight feedback loop I wanted while I am still learning the repo. I could also explain the root cause out loud, which the checklist treats as a real signal that you understand the problem rather than just pattern matching a fix.

The one thing I want to stay careful about is scope creep. It would be tempting to start cleaning up the claim extraction logic while I am in there, but that is not what the issue asks for, so I am going to keep the change focused on the None handling and the test that proves it.
