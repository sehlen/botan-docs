# Review: AEAD, Block Ciphers, Entropy Tests

**Botan Version:** 3.12.0
**Date:** 2026-05-20
**Files Reviewed:** 01_aead.rst, 03_block_ciphers.rst, 04_entropy_srcs.rst

---

## 01_aead.rst

### AEAD-1
**Status:** 🔄 PROPOSED CHANGES

**Notes:**
The overall structure of the encryption steps is accurate. However, several issues were found:

1. **Input field "Block Cipher" is inaccurate.** The test constructor is
   `Text_Based_Test("aead", "Key,In,Out", "Nonce,AD")` and `run_one_test` receives the full
   AEAD algorithm name (e.g., `AES-128/GCM`) as `algo`, not just the underlying block cipher.
   There is no separate "Block Cipher" test input field in the data files.

2. **Pre-checks in `run_one_test` are not documented.** Before calling `test_enc`, `run_one_test`
   performs several checks that are not mentioned in AEAD-1 or any other spec entry:
   - Both enc and dec modes are verified to be `authenticated()`.
   - Both enc and dec provider strings are verified to be non-empty and equal.
   - `ideal_granularity() > minimum_final_size()` for both enc and dec.
   - `update_granularity() > 0` for enc.
   - `enc->ideal_granularity() == dec->ideal_granularity()`.
   - `enc->ideal_granularity() > enc->update_granularity()`.
   - `enc->ideal_granularity() % enc->update_granularity() == 0`.

3. **Steps 15 and 16 use incorrect terminology.** The spec says "block size blocks" and "multiples
   of block size blocks", but the code uses `update_granularity()` as the chunking unit, not
   `block_size()`. These are not necessarily the same value.

4. **`has_keying_material()` check is not mentioned.** After object creation and before the
   first exception test, the code checks `enc->has_keying_material() == false` and again
   `enc->has_keying_material() == true` after setting the key. These assertions are missing from
   the steps.

**Proposed Changes:**
- Replace "Block Cipher: The underlying block cipher, e.g., AES-128 or AES-256" with
  "Algorithm: The full AEAD algorithm name (e.g., AES-128/GCM, ChaCha20Poly1305)".
- In steps 15–16, replace "block size" with "update granularity".
- Add a step (after step 3) checking that `has_keying_material()` returns false before the key
  is set, and add a step after setting the key that checks it returns true.
- Either document the `run_one_test` pre-checks (authenticated, provider, granularity) in AEAD-1
  or add a new AEAD-0 entry covering these structural validation checks.

---

### AEAD-2
**Status:** 🔄 PROPOSED CHANGES

**Notes:**
Several issues were found in the decryption test steps:

1. **Typo in step 6.** The spec says "Set the key *Key* on the **AEAD_Encryption** object" but
   this is the decryption test — it should be "AEAD_Decryption object".

2. **Steps 10–11 are duplicates of steps 7–8, and the ordering does not match the code.**
   The actual code flow in `test_dec` after `set_key` is:
   - Set mutated AD (`dec->set_associated_data(mutate_vec(ad, rng))`).
   - Check that `update` throws without a nonce set.
   - Check that `finish` throws without a nonce set.
   - Start with mutated nonce (`dec->start(mutate_vec(nonce, rng))`).
   - Update with garbage (`dec->update(garbage)`).
   - Reset.

   The spec has the wrong order (check-before-nonce, set-nonce, set-AD, check-before-nonce,
   set-nonce-again) whereas the code sets mutated AD first, then does the two "no nonce"
   exception checks, then sets mutated nonce and processes garbage.

3. **Same issues as AEAD-1** apply here: "Block Cipher" input field name, use of "block size"
   instead of "update granularity" in steps 15–16, and missing `has_keying_material()` checks.

**Proposed Changes:**
- Fix typo in step 6: "AEAD_Encryption" → "AEAD_Decryption".
- Rewrite steps 7–12 to match actual code order:
  1. Set a modified version of AD on the AEAD_Decryption object.
  2. Check that trying to update (process data) without a nonce set throws an exception.
  3. Check that trying to finish without a nonce set throws an exception.
  4. Start the AEAD_Decryption object with a modified version of the nonce.
  5. Pass a random ciphertext value (of update_granularity length) into the object.
  6. Reset the AEAD_Decryption object.
- Apply same "Block Cipher → Algorithm" and "block size → update granularity" fixes as AEAD-1.

---

### AEAD-3
**Status:** 🔄 PROPOSED CHANGES

**Notes:**
The overall structure is accurate. However:

1. **`dec->reset()` calls between sub-tests are not mentioned.** Between the modified-ciphertext
   test, modified-nonce test, and modified-AD test, the code calls `dec->reset()` to clear
   message state. The spec does not mention these resets, which are important for understanding
   the state machine.

2. **Same "Block Cipher" input field issue** as AEAD-1/2.

3. **Exception type not specified.** The code explicitly catches `Botan::Integrity_Failure` (a
   specific subtype). The spec only says "should throw an exception". Specifying the expected
   exception type (`Integrity_Failure`) would be more precise.

**Proposed Changes:**
- Add a reset step between each of the three negative sub-tests (modified ciphertext, modified
  nonce, modified AD).
- Clarify that the expected exception is specifically `Botan::Integrity_Failure`.
- Apply the "Block Cipher → Algorithm" input field fix.

---

### AEAD-GCM-1
**Status:** ✅ CONFIRMED

**Notes:**
The example test vector values (Key, Nonce, In, Out) are correct for a well-known AES-128/GCM
test case. The delegation to "See generic description in test case *AEAD-1*" is appropriate since
the same `AEAD_Tests` class handles all AEAD algorithms. The constraint summary (43 test cases,
sources, key/nonce/output bit lengths) cannot be verified without counting test vectors in
`src/tests/data/aead/gcm.vec`, but the description is structurally sound.

---

## 03_block_ciphers.rst

### BLOCK-1
**Status:** 🔄 PROPOSED CHANGES

**Notes:**
1. **Step 5 says "equals" but code checks "greater than or equal to".** The code uses
   `result.test_sz_gte(provider, cipher->parallel_bytes(), cipher->block_size() * cipher->parallelism())`,
   which asserts `parallel_bytes() >= block_size() * parallelism()`. The spec says the test
   checks that `parallel_bytes` *equals* `block_size * parallelism`, which is stricter than
   what the code actually enforces.

2. **`has_keying_material()` check is missing.** Before the exception tests, the code checks
   `cipher->has_keying_material() == false`. This is not mentioned in the spec.

3. **Clear/reset test after initial key set is missing.** After the unit-test checks, the code
   sets a random key using `maximum_keylength()`, encrypts random garbage of `block_size()`, then
   calls `cipher->clear()` — before proceeding to the KAT. This intermediate clear is not
   documented.

**Proposed Changes:**
- Change step 5 wording from "equals" to "is greater than or equal to".
- Add a step checking `has_keying_material() == false` before the exception tests.
- Add steps documenting the intermediate set_key/encrypt/clear sequence (or note it as
  belonging to BLOCK-2 preamble).

---

### BLOCK-2
**Status:** 🔄 PROPOSED CHANGES

**Notes:**
1. **"Iterations" input field does not exist.** The test constructor is
   `Text_Based_Test("block", "Key,In,Out", "Tweak")` — there is no `Iterations` field in the
   block cipher test data format. Steps 10–11 describe encrypting/decrypting "Iterations times"
   but the code always encrypts and decrypts exactly once per test vector.

2. **Step 2 says "minimum key length" but code uses maximum key length.** The pre-KAT random
   key test (`cipher->set_key(this->rng().random_vec(cipher->key_spec().maximum_keylength()))`)
   uses `maximum_keylength()`, not `minimum_keylength()`.

3. **Step 3 says "random plaintext of length *key length* bits" but code uses block size.**
   The code generates `rng().random_vec(cipher->block_size())` — the random plaintext length
   is the block size, not the key length.

4. **Step 4 says "Reset" but code calls `clear()`.** In Botan's BlockCipher API, the relevant
   method is `clear()`, not a generic "reset". The spec should use the correct API name.

5. **"Tweak" input is not documented.** The test data format supports an optional `Tweak` field
   for tweakable block ciphers (`Tweakable_Block_Cipher`). When a tweak is present, the code
   dynamic-casts and calls `tbc->set_tweak()`. This is not mentioned in BLOCK-2 at all.

6. **Post-KAT clear tests not documented.** After the KAT and misaligned-buffer tests, the code
   verifies `has_keying_material() == true`, calls `cipher->clear()`, verifies
   `has_keying_material() == false`, and re-checks that encrypt and decrypt throw after clearing.
   None of this appears in BLOCK-2 or any other spec entry.

**Proposed Changes:**
- Remove the "Iterations" input field and rewrite steps 10–11 to say "Encrypt the input value
  *In* once and compare the result with the expected value *Out*" and "Decrypt the result once
  and compare with *In*".
- Fix step 2: "minimum key length" → "maximum key length".
- Fix step 3: "of length *key length* bits" → "of length *block size* bits".
- Fix step 4: "Reset" → "Clear (`clear()`)".
- Add documentation for the optional Tweak input and the `Tweakable_Block_Cipher` path.
- Add a post-KAT sub-section documenting the clear/re-check sequence.

---

### BLOCK-3
**Status:** 🔄 PROPOSED CHANGES

**Notes:**
BLOCK-3 is described as a separate decrypt-only KAT, but the Botan source code has no
separate decrypt test class or method. Decryption is performed within the same `run_one_test`
function as encryption (BLOCK-2). The spec implies these are two independent test runs with
different inputs, which is misleading.

Specific issues:
1. **"Iterations" input field does not exist** (same as BLOCK-2).
2. **Step 3 says "Encrypt *Iterations* times the value *In*"** — this is inconsistent with a
   decryption test spec. The code simply does `buf = expected; cipher->decrypt(buf)`, using the
   known ciphertext (*Out*) directly without re-encrypting.
3. **Step 4 says "compare with the input value *Out*"** but the expected output header says
   "*In*: The original test message". The comparison in step 4 should be with *In*, not *Out*.
4. **The same "Iterations" issue applies** as BLOCK-2.

**Proposed Changes:**
- Clarify that BLOCK-3 is not a separate test run but rather the decryption half of the same
  KAT as BLOCK-2, executed within the same `run_one_test` invocation.
- Remove step 3 entirely ("Encrypt *Iterations* times the value *In*") — the code does not
  re-encrypt as part of the decrypt test; it decrypts the provided expected ciphertext.
- Rewrite step 4: "Decrypt the ciphertext *Out* once and compare the result with *In*".
- Remove the "Iterations" input field.

---

### BLOCK-AES-2
**Status:** 🔄 PROPOSED CHANGES

**Notes:**
The example test vector values (Key, In, Out) are correct for a standard AES-128 test case
(FIPS 197 Appendix B). The steps follow the BLOCK-2 template, so the same issues apply:

1. **Step 2 says "minimum key length"** — should be "maximum key length".
2. **Step 3 says "key length"** — should be "block size" (128 bits = 16 bytes for AES-128).
3. **Steps 6–7 reference only single encryption/decryption (no Iterations)** — the test vector
   for this example uses `Iterations` implicitly set to 1, so this is effectively correct for
   the example case, but the BLOCK-2 template it references has the Iterations errors noted above.

**Proposed Changes:**
- Apply same fixes as BLOCK-2 (minimum → maximum key length; key length → block size).
- Ensure the "Steps" reference to BLOCK-2 template is updated once BLOCK-2 is corrected.

---

## 04_entropy_srcs.rst

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

## New Tests Found in Botan 3.12.0 Not in Spec

1. **`BlockCipher_ParallelOp_Test` (registered as `"bc_parop"`)** — `src/tests/test_block.cpp`.
   This test class is entirely absent from `03_block_ciphers.rst`. It tests that parallel
   (SIMD/bitsliced) block cipher encryption and decryption produces the same result as sequential
   1-block-at-a-time processing, for a fixed set of ciphers: AES-128, AES-192, AES-256,
   ARIA-128, ARIA-256, Camellia-128, Camellia-192, Camellia-256, DES, TripleDES, IDEA, Noekeon,
   SEED, Serpent, SHACAL2, SM4. The test uses 255 blocks (`128+64+32+16+8+4+2+1`) to exercise
   tail-block handling. A new spec entry (e.g., BLOCK-4 or BLOCK-PAROP-1) should be added.

2. **AEAD `run_one_test` structural checks** — `src/tests/test_aead.cpp`. The checks for
   `authenticated()`, provider consistency, and granularity relationships (described in AEAD-1
   notes above) are not covered by any spec entry. These could be added as an AEAD-0 "Structural
   Validation" entry or merged into AEAD-1.

3. **AEAD post-processing clear checks** — Both `test_enc` and `test_dec` end with `clear()`
   followed by re-verification that the object throws for operations without a key. These are
   documented in AEAD-1 steps 17–18 and AEAD-2 steps 15–16, so they are covered; however, the
   AD-after-clear check (`enc->set_associated_data(ad)` before `enc->clear()`, then re-checking
   the clear state) is only partially captured.
