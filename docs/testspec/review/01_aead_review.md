# Review: AEAD Tests

**Botan Version:** 3.12.0
**Date:** 2026-05-20
**File Reviewed:** 01_aead.rst

---

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

---

## New Tests Found in Botan 3.12.0 Not in Spec

See also `03_block_ciphers_review.md` for the parallel-ops test.

- **AEAD `run_one_test` structural checks** — Pre-checks for `authenticated()`, provider consistency, and granularity relationships not covered by any spec entry. Could be added as AEAD-0 or merged into AEAD-1.
- **AEAD post-processing clear checks** — Both `test_enc` and `test_dec` end with `clear()` followed by re-verification of throw behaviour. Partially captured in AEAD-1 steps 17–18 and AEAD-2 steps 15–16.
