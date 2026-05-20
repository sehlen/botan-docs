# Review: Entropy Sources Tests

**Botan Version:** 3.12.0
**Date:** 2026-05-20
**File Reviewed:** 04_entropy_srcs.rst

---

### ENTROPY-1
**Status:** 🔄 PROPOSED CHANGES

**Notes:**
1. **Step 2b says "added entropy exactly once" but code checks >= 1.** The code does
   `result.test_sz_gte("Samples", rng.samples(), 1)`, which asserts `samples >= 1`, not
   `samples == 1`. The spec's wording "added entropy exactly once" implies an equality check,
   which is stricter than what the code enforces.

2. **Compression sub-tests are completely absent from the steps.** The intro paragraph of
   `04_entropy_srcs.rst` mentions compression testing, but ENTROPY-1's steps do not describe it.
   The code (when `BOTAN_HAS_COMPRESSION` is defined) performs significant additional checks for
   each entropy source that produced data:
   - Compresses the seed material with **zlib** and **lzma** at level 9.
   - Verifies the compressed size * 8 >= advertised entropy bits.
   - Polls the entropy source a second time.
   - Concatenates both poll results and compresses together.
   - Verifies that the two-poll compressed size is strictly larger than the one-poll compressed
     size (entropy is not constant/repeated).
   - Verifies the differential compressed size * 8 >= the second poll's entropy estimate.
   Note: bzip2 is explicitly skipped due to a known macOS issue (GitHub #394) and block-size
   effects on the differential test.

3. **Step 2a condition is incomplete.** The code gates the `samples` and `seed_material` checks
   on `if(rng.samples() > 0)`, meaning if no entropy was produced the checks are skipped
   entirely. The spec says "check that it added at least one byte" for any source that added
   entropy — the conditional logic should be explicit.

**Proposed Changes:**
- Fix step 2b: change "check that it added entropy exactly once" to "check that
  `rng.samples() >= 1`".
- Add a new step 2c covering the compression sub-tests (conditional on BOTAN_HAS_COMPRESSION):
  - For each of zlib and lzma: compress seed material and verify compressed_size * 8 >= entropy
    estimate.
  - Poll entropy source a second time; concatenate both seed materials, compress, and verify
    the combined compressed size is larger than the first; verify differential entropy >= second
    poll estimate.
  - Note that bzip2 is intentionally excluded.
- Clarify step 2b's condition: checks only apply when `rng.samples() > 0`.

---
