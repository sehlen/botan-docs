# Review: Block Ciphers Tests

**Botan Version:** 3.12.0
**Date:** 2026-05-20
**File Reviewed:** 03_block_ciphers.rst

---

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

---

## New Tests Found in Botan 3.12.0 Not in Spec

1. **`BlockCipher_ParallelOp_Test` (registered as `"bc_parop"`)** — `src/tests/test_block.cpp`.
   Tests that parallel (SIMD/bitsliced) block cipher encryption and decryption produces the
   same result as sequential 1-block-at-a-time processing, for a fixed set of ciphers:
   AES-128, AES-192, AES-256, ARIA-128, ARIA-256, Camellia-128/-192/-256, DES, TripleDES,
   IDEA, Noekeon, SEED, Serpent, SHACAL2, SM4. Uses 255 blocks to exercise tail-block
   handling. A new spec entry (e.g., BLOCK-4 or BLOCK-PAROP-1) should be added.
