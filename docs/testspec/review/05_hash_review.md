# Review: Hash Functions Tests

**Botan Version:** 3.12.0
**Date:** 2026-05-20
**File Reviewed:** 05_hash.rst

---

## 05_hash.rst

### HASH-1
**Status:** 🔄 PROPOSED CHANGES
**Notes:** Several inaccuracies in the Steps description compared to `Hash_Function_Tests::run_one_test` in `test_hash.cpp`.

**Proposed Changes:**

1. **Loop count is wrong.** The spec says "Repeat **five** times in a loop" but the code runs `for(size_t i = 0; i != 3; ++i)` — exactly **three** iterations.
   - Change: "Repeat five times" → "Repeat three times"

2. **Misaligned-data test is missing.** After the clear/reset block, the code performs a misaligned-pointer test before the copy_state block. This step is absent from the spec. Add a step:
   - "If *In* is non-empty, construct a misaligned copy of *In* and feed it to the hash function; verify the digest matches *Out*."

3. **Copy-state step is conditional.** The split / `copy_state` test at the end of the loop (spec steps 9–12) is guarded by `if(input.size() > 5)`. For very short or empty inputs (e.g. the zero-length test vectors used in HASH-MD5-1 etc.) these steps are **not** executed. The spec should note this precondition.

4. **clone vs. copy_state semantics.** At the very beginning of `run_one_test`, a `clone = hash->new_object()` is created (a fresh, empty object sharing only the algorithm). Spec step 5 says "Copy HashFunction object and its state" — but that description matches `copy_state()` (performed later, before the split test). The `new_object()` clone only verifies name equality and produces an independent result on a fresh input; it does not carry forward the partially-fed state.

5. **Fork feeds bytes differently.** The spec step says "Feed rest of *In* into **both** the original and the copied hash functions", implying identical treatment. In the code:
   - The **original** hash receives the remaining bytes in randomly-sized chunks (via `rng().next_byte()`).
   - The **fork** (copy_state) receives bytes `[1 .. n-2]` in one call, then byte `[n-1]` in a second call.
   Both end up hashing the full input, but the mechanism differs. The spec should clarify that the split is performed in random-sized increments for the original and in two chunks for the fork.

---

### HASH-2
**Status:** 🔄 PROPOSED CHANGES
**Notes:** The spec presents HASH-2 as a standalone "Known Answer Test that hashes a message in **two** chunks (byte 1, then bytes 2..n)". This no longer matches the source code.

**Proposed Changes:**

The two-chunk split behaviour is **not** a separate test class; it is the tail of `Hash_Function_Tests::run_one_test` (the same function that implements HASH-1). In the current code the remaining bytes are fed in **random-sized** chunks (not exactly one more chunk). The random chunking is driven by `rng().next_byte()`.

Additionally, the precondition "In must be of length n > 1 byte" should read "n > **5** bytes" (the guard in the code is `if(input.size() > 5)`).

Options:
- Merge HASH-2's description into HASH-1 as the final sub-steps, or
- Update HASH-2 to say the message is split into **randomly-sized increments** (rather than exactly two chunks) and fix the precondition to n > 5.

---

### HASH-3
**Status:** 🔄 PROPOSED CHANGES
**Notes:** The inner-loop bound is described correctly (i = 3 to 1002, 1000 iterations). One step description is misleading.

**Proposed Changes:**

Step 3.1.4 reads: "Feed *In[0]* into the hash function and calculate the message digest."

This is wrong. The code calls:
```cpp
hash->final(input[0].data());
```
This writes the digest **into** `In[0]`'s buffer — it does **not** feed `In[0]` as additional input. The correct wording should be:
- "Calculate the message digest and overwrite *In[0]* with the result."

Also, the Expected Output label in the spec uses "Out", but the test data file (`hash_mc.vec`) uses the field name `Output`. The spec should use the actual field name for clarity.

---

### HASH-4
**Status:** 🔄 PROPOSED CHANGES
**Notes:** Multiple mismatches with `Hash_LongRepeat_Tests` in `test_hash.cpp`.

**Proposed Changes:**

1. **Wrong field names.** The test data file (`hash_rep.vec`) uses fields `Input`, `TotalLength`, and `Digest`. The spec uses `In` and `Out`. Update:
   - "In" → "Input"
   - "Out" (Expected Output) → "Digest"

2. **TotalLength is a byte count, not a repetition count.** The spec says "The number of **times** *In* should be processed by the hash function." In the code `TotalLength` is the **total number of bytes** to hash. The input buffer is first expanded to ≥ 256 bytes via `expand_input()`, then the code computes how many full copies plus a leftover fit into `TotalLength` bytes and feeds them accordingly. Fix:
   - "The **total number of bytes** to be hashed"

3. **Steps are oversimplified.** The spec says "Feed *In* *TotalLength* times into the hash function" which is doubly wrong (TotalLength is bytes, not iterations, and the expanded buffer is used). Updated steps should read approximately:
   1. Expand *Input* by repeating it until the buffer is at least 256 bytes long.
   2. Compute `full_copies = TotalLength / len(expanded_Input)` and `leftover = TotalLength mod len(expanded_Input)`.
   3. Feed the expanded buffer to the hash function `full_copies` times.
   4. Feed the first `leftover` bytes of the expanded buffer.
   5. Calculate the message digest and compare with *Digest*.

4. **Long-test guard.** When `TotalLength > 1 000 000`, the test is skipped unless the `--run-long-tests` flag is set. This is not mentioned in the spec.

---

### HASH-MD5-1
**Status:** 🔄 PROPOSED CHANGES
**Notes:** The Steps mirror HASH-1 but are instantiated for MD5 specifically. Same corrections as HASH-1 apply:

**Proposed Changes:**
- "Repeat five times" → "Repeat three times" (step 3 of HASH-1 style loop).
- Add the misaligned-data step.
- Clarify that the copy_state/split test (steps 10–12) is **only** executed when `In` has more than 5 bytes. Because the example test vector uses an **empty** `In`, steps 10–12 are not actually run for this particular test vector.

---

### HASH-SHA1-1
**Status:** 🔄 PROPOSED CHANGES
**Notes:** Same structural issues as HASH-MD5-1.

**Proposed Changes:**
- "Repeat five times" → "Repeat three times".
- Add the misaligned-data step.
- Note that copy_state/split steps are skipped for the empty-input test vector.

---

### HASH-SHA224-1
**Status:** 🔄 PROPOSED CHANGES
**Notes:** Same structural issues as HASH-MD5-1. Also: the spec states SHA-224 has only **2** test cases (empty message and 8-bit message). Confirm against current `sha2_32.vec` — the vec file also covers SHA-256, so the count "2" for SHA-224 specifically should be re-verified.

**Proposed Changes:**
- "Repeat five times" → "Repeat three times".
- Add misaligned-data step.
- Clarify copy_state/split step conditionality.

---

### HASH-SHA256-1
**Status:** 🔄 PROPOSED CHANGES
**Notes:** Same structural issues as HASH-MD5-1.

**Proposed Changes:**
- "Repeat five times" → "Repeat three times".
- Add misaligned-data step.
- Clarify copy_state/split step conditionality (for empty input vector).

---

### HASH-SHA384-1
**Status:** 🔄 PROPOSED CHANGES
**Notes:** Same structural issues as HASH-MD5-1.

**Proposed Changes:**
- "Repeat five times" → "Repeat three times".
- Add misaligned-data step.
- Clarify copy_state/split step conditionality.

---

### HASH-SHA512-1
**Status:** 🔄 PROPOSED CHANGES
**Notes:** Same structural issues. Minor typo in the spec: step 7 reads "Feed an input value of length zero into **the**SHA512" (missing space before "SHA512").

**Proposed Changes:**
- "Repeat five times" → "Repeat three times".
- Fix typo: "into theSHA512" → "into the SHA512".
- Add misaligned-data step.
- Clarify copy_state/split step conditionality.

---

### HASH-SHA512-256-1
**Status:** 🔄 PROPOSED CHANGES
**Notes:** Same structural issues as HASH-MD5-1.

**Proposed Changes:**
- "Repeat five times" → "Repeat three times".
- Add misaligned-data step.
- Clarify copy_state/split step conditionality.

---

### HASH-SHA3-224-1
**Status:** 🔄 PROPOSED CHANGES
**Notes:** Same structural issues as HASH-MD5-1. Additionally the **Steps cell has a severe formatting defect**: all early numbered steps are collapsed into a single run-on paragraph (lines 673–679 of the RST), making the description unreadable.

**Proposed Changes:**
- Fix the RST formatting of the Steps cell — restore numbered list formatting for steps 1–9 (currently all run together without line breaks / list markers).
- "Repeat five times" → "Repeat three times".
- Add misaligned-data step.
- Clarify copy_state/split step conditionality.

---

### HASH-SHA3-256-1
**Status:** 🔄 PROPOSED CHANGES
**Notes:** Same structural issues as HASH-MD5-1.

**Proposed Changes:**
- "Repeat five times" → "Repeat three times".
- Add misaligned-data step.
- Clarify copy_state/split step conditionality.

---

### HASH-SHA3-384-1
**Status:** 🔄 PROPOSED CHANGES
**Notes:** Same structural issues as HASH-MD5-1.

**Proposed Changes:**
- "Repeat five times" → "Repeat three times".
- Add misaligned-data step.
- Clarify copy_state/split step conditionality.

---

### HASH-SHA3-512-1
**Status:** 🔄 PROPOSED CHANGES
**Notes:** Same structural issues as HASH-MD5-1.

**Proposed Changes:**
- "Repeat five times" → "Repeat three times".
- Add misaligned-data step.
- Clarify copy_state/split step conditionality.

---

### HASH-SHAKE-128-128
**Status:** 🔄 PROPOSED CHANGES
**Notes:** Same structural issues as HASH-1 (loop count, missing misaligned step). The test vector itself (In and Out values) is consistent with what is in `shake.vec`. The SHAKE test uses `Hash_Function_Tests` so all the same caveats apply.

**Proposed Changes:**
- "Repeat five times" → "Repeat three times".
- Add misaligned-data step.
- Copy_state/split steps are executed for this test vector since `In` is 16 bytes (> 5).

---

### HASH-BLAKE2B-384
**Status:** 🔄 PROPOSED CHANGES
**Notes:** Same structural issues as HASH-1 (loop count, missing misaligned step). Test vector values (In / Out) are consistent with `blake2b.vec`. Note the spec uses "Blake2b" but Botan's canonical name is `BLAKE2b`.

**Proposed Changes:**
- "Repeat five times" → "Repeat three times".
- Add misaligned-data step.
- Copy_state/split steps run for this test vector (In is 21 bytes > 5).
- Consider using the canonical Botan algorithm name `BLAKE2b(384)` (uppercase B) throughout for consistency.

---

### H-PHASH-1
**Status:** 🔄 PROPOSED CHANGES
**Notes:** This test is run by the generic `Hash_Function_Tests` class (registered as `hash_algos`) against the test vectors in `src/tests/data/hash/parallel.vec`. There is no dedicated Parallel hash unit test class in `test_hash.cpp`.

**Proposed Changes:**

1. **Description is misleading.** The title says "Unit test for **cloning** of a Parallel hash object" but the test is a standard KAT driven by `Hash_Function_Tests` — the same test logic as all other hash algorithms. The description should be updated to reflect this.

2. **Steps do not match the code.** The listed steps (create object, feed empty input, compute digest, clone, reset clone, feed empty input, compute digest again) roughly describe the `new_object()` + `clear()` sequence inside `Hash_Function_Tests::run_one_test`, but:
   - The code creates `new_object()` at the start (not after the first digest check).
   - The code repeats the hash-and-check loop 3 times (not described).
   - The code also performs misaligned-data testing and `copy_state()` testing.
   The steps should either be replaced with a reference to HASH-1 or updated to reflect the actual sequence.

3. **Algorithm name.** The spec says "SHA-160" but the test vector in `parallel.vec` uses `Parallel(MD5,SHA-1)`. SHA-160 is an alternative name for SHA-1 in Botan, so functionally this is the same, but for clarity the spec should use the name as it appears in the test vector: `SHA-1`.

---

### H-PHASH-2
**Status:** 🔄 PROPOSED CHANGES
**Notes:** Same structural issues as H-PHASH-1. This test runs `Hash_Function_Tests` for the `Parallel(SHA-256,SHA-512)` entry in `parallel.vec`.

**Proposed Changes:**

1. **Description.** "Unit test for **construction** of a Parallel hash object" — no custom construction is tested beyond passing the algorithm name string to `HashFunction::create`. The description should align with what `Hash_Function_Tests` actually verifies.

2. **Steps.** "Create a SHA-256 object", "Create a SHA-512 object", "Create a Parallel hash object with the SHA-256 and SHA-512 objects" — the actual test calls `Botan::HashFunction::create("Parallel(SHA-256,SHA-512)", provider)`, not explicit construction from pre-existing sub-hash objects. The spec should reflect the string-based algorithm lookup.

3. All the same loop-count and misaligned-data issues apply (see HASH-1 notes).

---

## New Tests Found in Botan 3.12.0 Not in Spec

| Test ID | Class/Test | Registration | First Added |
|---------|------------|--------------|-------------|
| — | `Invalid_Hash_Name_Tests` | `hash` / `invalid_name_hash` | Mar 2018 (commit 30a0bc7) |
| — | `hash_truncation_negative_tests` | `hash` / `hash_truncation` | Feb 2023 (commit 30a0bc7) |

### Timeline Context

**Invalid_Hash_Name_Tests** — Added in March 2018 (commit 30a0bc7) to test that `HashFunction::create_or_throw` raises appropriate exceptions for invalid algorithm names. Tests various error cases:
- `NonExistentHash` → `Lookup_Error`
- `Blake2b(9)` → `Invalid_Argument` with message "Bad output bits size for BLAKE2b"
- `Comb4P(MD5,MD5)` → `Invalid_Argument` with message "Comb4P: Must use two distinct hashes"
- `Comb4P(MD5,SHA-256)` → `Invalid_Argument` with message "Comb4P: Incompatible hashes MD5 and SHA-256"
- `Keccak-1600(160)` → `Invalid_Argument` with message "Keccak_1600: Invalid output length 160"
- `SHA-3(160)` → `Invalid_Argument` with message "SHA_3: Invalid output length 160"

This test predates the 3.7.1 baseline and was missed during previous documentation.

**hash_truncation_negative_tests** — Added in February 2023 (commit 30a0bc7) to test parameter validation for the `Truncated(...)` wrapper:
- `Truncated(SHA-256,0)` → `Invalid_Argument`
- `Truncated(SHA-256,257)` → `Invalid_Argument` (more bits than underlying hash)
- `Truncated(NonExistentHash-256,128)` → returns `nullptr` (not created)

This test was added relatively recently but still predates 3.7.1 and was missed.

### Additional KAT coverage for hash algorithms

The `Hash_Function_Tests` class processes all `.vec` files in `src/tests/data/hash/`. The following algorithms have test vectors but are not mentioned in `05_hash.rst`:
- `Adler32`, `Ascon-Hash256`, `BLAKE2s`, `Comb4P`, `CRC24`, `CRC32`, `GOST-34.11`, `Keccak-1600`, `MD4`, `RIPEMD-160`, `SM3`, `Skein-512`, `Streebog-256/512`, `Truncated(SHA-256,...)`, `Whirlpool`

All of these algorithms and their test vectors existed before the 3.7.1 baseline.

**Conclusion:** The Invalid_Hash_Name_Tests and hash_truncation tests were missed during previous documentation updates. Both predate the 3.7.1 baseline and should have been documented earlier.
