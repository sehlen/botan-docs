# Review: Modes of Operation Tests

**Botan Version:** 3.12.0
**Date:** 2026-05-20
**File Reviewed:** 08_modes_of_operation.rst

---


Cipher mode tests (CBC, CTS, CFB, XTS) use `Cipher_Mode_Tests` in `src/tests/test_modes.cpp`. CTR is a stream cipher in Botan and is tested by `Stream_Cipher_Tests` in `src/tests/test_stream.cpp`.

---

### MODE-1
**Status:** 🔄 PROPOSED CHANGES

**Notes:**
- The core encrypt flow (create, name check, not-authenticated check, unkeyed throws, set key, start nonce, finish, update-block, update-all, clear, unkeyed-after-clear throws) is accurately described.
- **Missing test steps** present in the code but not in the spec:
  - **Granularity checks:** The code verifies that enc and dec `update_granularity()` are equal, both are non-zero, `ideal_granularity()` equals for enc/dec, ideal is greater than update granularity, and ideal is a multiple of update granularity.
  - **`output_length()` check:** The code calls `mode.output_length(input.size())` and verifies it equals `expected.size()` (with `<=`/`>=` for CBC due to padding).
  - **Valid nonce size check:** The code checks `valid_nonce_length(default_nonce_length())` returns true before the "large nonce" rejection test.
  - **`finish()` with non-zero offset:** The code tests that `finish(buf, offset)` correctly processes data starting at `offset` and leaves prefix bytes untouched.
  - **`update()` + `finish()` with non-zero offset:** Same offset test for the update+finish split.
  - **`process()` method:** The code also tests the `process(buf, bytes)` method variant, not just `update()` and `finish()`.
  - **Key mutation before real key:** Before setting the real key, the code sets a mutated (random-modified) key and a mutated nonce in intermediate steps to verify reset behavior. These are internal test harness details not visible to spec.

**Proposed Changes:**
- Add step: "Verify that `update_granularity()`, `ideal_granularity()`, and `output_length()` return correct and self-consistent values."
- Add step: "Verify that `valid_nonce_length(default_nonce_length())` returns true."
- Add step after the all-in-one encrypt: "Also encrypt using `finish()` with a non-zero buffer offset, verifying prefix bytes are unchanged."
- Add step: "If input is long enough, also encrypt using `update()` + `finish()` with non-zero offsets."
- Add step: "If input is long enough, also test the `process()` method."

---

### MODE-2
**Status:** 🔄 PROPOSED CHANGES

**Notes:**
- Same structural omissions as MODE-1 (granularity, output_length, valid nonce, offset variants, process()).
- **Bug in step 6:** "Set the key *Key* on the Cipher_Mode **encryption** object" — should be "**decryption** object" (copy-paste error from MODE-1).
- **Confusing wording in step 8:** "Calculate the plaintext of output value *In*" — should be "Calculate the plaintext of ciphertext *Out* and compare the result with the expected plaintext *In*." The variable names are inverted relative to how decryption works.

**Proposed Changes:**
- Fix step 6: "encryption object" → "decryption object".
- Reword step 8 for clarity: "Decrypt ciphertext *Out* and compare the result with the expected plaintext *In*."
- Add the same missing steps as MODE-1 (granularity, output_length, offset variants, process()).

---

### MODE-CBC-1
**Status:** ✅ CONFIRMED (with minor notes)

**Notes:**
- The spec accurately describes the CBC encryption test with the NIST AES-128 CBC vector (128-bit key, 128-bit nonce, 512-bit plaintext, 512-bit ciphertext).
- Steps correctly describe the core flow: create, name check, not-authenticated check, unkeyed-throws, large-nonce-throws, set key, set nonce, encrypt, clear, unkeyed-after-clear-throws.
- This is a subset of the full `Cipher_Mode_Tests` flow (same omissions as MODE-1: granularity checks, offset tests, process() test), but as a per-algorithm example test case it is acceptable.
- The spec says "Number of test cases: 3" (AES-128, AES-192, AES-256). Confirmed correct (these are the NoPadding CBC vectors; the same `cbc.vec` file also contains CTS vectors under different algorithm names).

---

### MODE-CTS-1
**Status:** ✅ CONFIRMED

**Notes:**
- The spec correctly identifies CTS (CBC-CS3) tested via `Cipher_Mode_Tests`, vectors in `src/tests/data/modes/cbc.vec`.
- Number of test cases (6) and parameter ranges (AES-128, 128-bit key, 128-bit nonce, varying input 136–512 bits) are accurate.
- The example vector (RFC 3962 first test vector) is accurate.
- Same subset-of-flow note as MODE-CBC-1.

---

### MODE-CTR-1
**Status:** 🔄 PROPOSED CHANGES

**Notes:**
- The spec correctly identifies CTR mode as a stream cipher tested in `test_stream.cpp` with vectors in `src/tests/data/stream/ctr.vec`.
- Steps correctly describe: create, name check, set key, set IV, clone test (different pointer, same name), set random key on clone, encrypt and compare.
- **Missing steps** present in `Stream_Cipher_Tests` but not described:
  - Checks that `valid_iv_length(default_iv_length())` returns true.
  - Checks that `buffer_size() > 0`.
  - Negative test: encrypting without a key set throws `Invalid_State`.
  - Negative test: seeking without a key set throws `Invalid_State` (or `Not_Implemented`).
  - Large nonce rejection test.
  - Re-encrypt after `set_iv()` reset ("encrypt 2").
  - **`write_keystream()` test:** generates keystream separately and XORs with input, verifying the result equals the expected ciphertext.
  - `clear()` and post-clear encryption fails.
- Note: there is also a `src/tests/data/modes/ctr.vec` file and CTR IV-carry tests in `Cipher_Mode_IV_Carry_Tests` (in `test_modes.cpp`) which are completely unmentioned in the spec.

**Proposed Changes:**
- Add negative test steps: "Verify that encrypting without setting a key throws `Invalid_State`."
- Add: "Verify `valid_iv_length(default_iv_length())` returns true and large IV sizes are rejected."
- Add: "Verify `buffer_size() > 0`."
- Add: "After the initial encrypt, re-encrypt after resetting IV and verify the same output."
- Add: "Verify `write_keystream()` produces keystream that, XOR'd with input, yields the expected ciphertext."
- Add: "Clear the cipher and verify post-clear encryption throws `Invalid_State`."

---

---

## New Tests Found in Botan 3.12.0 Not in Spec

| Test ID | Mode/Algorithm | Test Data File / Class | First Added |
|---------|----------------|----------------------|-------------|
| — | CFB | `modes/cfb.vec` | Pre-3.7.1 (before 2024) |
| — | XTS | `modes/xts.vec` | Pre-3.7.1 (before 2024) |
| — | CTR (Cipher_Mode) | `modes/ctr.vec` | Pre-3.7.1 (before 2024) |
| — | ChaCha20 | `stream/chacha.vec` | Jan 2014 (commit e11024f) |
| — | OFB | `stream/ofb.vec` | Pre-3.7.1 (before 2024) |
| — | RC4 | `stream/rc4.vec` | Pre-3.7.1 (before 2024) |
| — | Salsa20 | `stream/salsa20.vec` | Pre-3.7.1 (before 2024) |
| — | SHAKE (XOF as stream) | `stream/shake.vec` | Pre-3.7.1 (before 2024) |
| — | `Cipher_Mode_IV_Carry_Tests` | CBC, CFB, CTR IV carry | May 2017 (commit 2914fcf) |

### Timeline Context

**ChaCha20** — Stream cipher added in January 2014 (commit e11024f), well before the 3.7.1 baseline. This is a widely-used modern stream cipher that was missed in previous documentation.

**Cipher_Mode_IV_Carry_Tests** — Added in May 2017 (commit 2914fcf) to test IV carry-over behavior across multiple messages in CBC, CFB, and CTR modes. This allows empty nonce to mean "continue using current cipher state." The feature and its tests predate 3.7.1 and were missed.

**CFB, XTS, CTR, OFB, RC4, Salsa20, SHAKE** — All existed before the 3.7.1 baseline. These are core cipher modes and stream ciphers that have been in Botan for years and were overlooked during previous documentation updates.

**Conclusion:** All these cipher mode and stream cipher tests were missed during previous documentation cycles. They existed before the 3.7.1 baseline and should have been documented earlier.
