# Review: KDF, MAC, Modes of Operation, PBKDF Tests

**Botan Version:** 3.12.0
**Date:** 2026-05-20
**Files Reviewed:** 06_kdf.rst, 07_mac.rst, 08_modes_of_operation.rst, 09_pbkdf.rst

---

## 06_kdf.rst

All KDF tests use the single `KDF_KAT_Tests` class in `src/tests/test_kdf.cpp`:

```cpp
KDF_KAT_Tests() : Text_Based_Test("kdf", "Secret,Output", "Salt,Label,IKM,XTS") {}
```

Required fields: `Secret`, `Output`. Optional fields: `Salt`, `Label`, `IKM`, `XTS`.
The test loads all `.vec` files under `src/tests/data/kdf/`.

---

### KDF-1
**Status:** 🔄 PROPOSED CHANGES

**Notes:**
- Core flow (create, name check, derive key, clone check) is accurately described.
- Step 2 has a typo: says "*InputSalt*" — should be "*Salt*".
- The spec does not mention the optional `IKM` (Input Keying Material) and `XTS` fields accepted by the test framework. Some KDF algorithms (notably HKDF variants) pass these via test vectors.
- The spec omits an additional test path: when the expected output is exactly 32 bytes, the code also calls the fixed-size variant `kdf->derive_key<32>(secret, salt, label)` and compares results. This is an additional positive test not described.
- The spec does not mention that output length is inferred from the `Output` field length (not a separate input parameter).

**Proposed Changes:**
- Fix typo "*InputSalt*" → "*Salt*" in Step 2.
- Add a note on optional `IKM` and `XTS` parameters (used by HKDF and HKDF-Extract).
- Add a step: "If the expected output is 32 bytes, also derive using the fixed-size `derive_key<32>()` overload and compare results."

---

### KDF-KDF1-1
**Status:** ✅ CONFIRMED

**Notes:**
- The spec correctly references `src/tests/data/kdf/kdf1_iso18033.vec` as the source of test vectors for KDF1 (ISO 18033-2).
- Number of test cases (2), hash functions (SHA-1, SHA-256), input/output sizes, and example vector are all accurate.
- Steps accurately describe creating the KDF1_18033 object and deriving a key.
- Note: there is also a separate `kdf1.vec` file for KDF1 per X9.63/IEEE 1363a — that algorithm is not mentioned anywhere in the spec (see New Tests section).

---

### KDF-NISTSP800-108-CTR-1
**Status:** ✅ CONFIRMED

**Notes:**
- Spec correctly identifies the test class, data file (`sp800_108_ctr.vec`), MAC variants, and parameter ranges.
- The example vector (HMAC-SHA1, 80-bit salt, 160-bit secret, 16-bit output) is accurate.
- Steps correctly describe passing `Salt` and `Secret` via the KDF interface.
- Minor omission: `Label` is also an optional parameter consumed by `derive_key()`, though it may be empty in these test vectors.

---

### KDF-NISTSP800-108-FB-1
**Status:** ✅ CONFIRMED

**Notes:**
- Spec correctly references `sp800_108_fb.vec`, constraints (240 test cases, same MAC set), and example vector.
- Steps are accurate.

---

### KDF-NISTSP800-108-PI-1
**Status:** ✅ CONFIRMED

**Notes:**
- Spec correctly references `sp800_108_pipe.vec`, constraints, and example vector.
- Steps are accurate.

---

### KDF-TLS1-PRF-1
**Status:** ✅ CONFIRMED

**Notes:**
- Spec correctly identifies the TLS 1.0/1.1 PRF, references `tls_prf.vec`, and the example vector (208-bit salt, 152-bit secret, 8-bit output) is accurate.

---

### KDF-TLS12-PRF-1
**Status:** ✅ CONFIRMED

**Notes:**
- Spec correctly identifies the TLS 1.2 PRF, references `tls_prf.vec`, constraints (4 test cases, SHA-224/256/384/512), and the example vector is accurate.
- Label is correctly listed as an additional input for TLS 1.2 PRF (unlike TLS 1.0/1.1).

---

## 07_mac.rst

All MAC algorithms are tested through the single `Message_Auth_Tests` class in `src/tests/test_mac.cpp`:

```cpp
Message_Auth_Tests() : Text_Based_Test("mac", "Key,In,Out", "IV") {}
```

Required fields: `Key`, `In`, `Out`. Optional: `IV` (used for GMAC and KMAC).

---

### MAC-1
**Status:** 🔄 PROPOSED CHANGES

**Notes:**
- The overall description is accurate: tests reset behavior, whole-message MAC computation, and same-object reuse.
- The spec's Step 4 ("Repeat twice: set key, input In, calculate tag") corresponds to two iterations in code (`correct mac` and `correct mac (try 2)`). However, for MACs with no IV (HMAC, CMAC), the code performs a **third** iteration (`correct mac (no start call)`) that calls `update()` directly without a prior `start()`. This third check is **not described in the spec**.
- The spec does not mention that `start(iv)` is called before `update()` in each iteration (relevant for GMAC and KMAC).
- The spec does not mention that `has_keying_material()` is checked after creation (false) and after `set_key()` (true).
- MAC providers are iterated in the code (`possible_providers(algo)`) but the spec does not mention provider-level testing.
- After `clear()`, the code verifies `has_keying_material() == false` — not explicitly listed in the spec.
- The code also calls `mac->final(buf)` (pointer overload) with no key set and expects `Invalid_State` — this is described in the spec as the final step but only mentions the `update()` variant.

**Proposed Changes:**
- Add step after "Repeat twice": "For MACs that do not require an IV, also compute the MAC by calling `update()` directly without a prior `start()` call and compare with *Out*."
- Add note: "For each iteration where an IV is required, call `start(IV)` before `update()`."
- Add note: "After creation, verify `has_keying_material()` returns false; after `set_key()`, verify it returns true."

---

### MAC-2
**Status:** 🔄 PROPOSED CHANGES

**Notes:**
- The core chunked-MAC logic (feed first byte, middle, last byte; compare) is accurate.
- The spec is missing `start(iv)` before each chunked MAC computation (required for GMAC and KMAC).
- Step 11: "Input *In* into the MAC and verify the tag with the expected output value *Out*" is misleading. The code does **not** feed `In` again — it calls `verify_mac(expected)` on the accumulated chunked result from steps 8–10. The message is not re-fed.
- The spec does not mention that `set_key()` must be called before the first chunked compute (the code comment even notes "Poly1305 requires the re-key").

**Proposed Changes:**
- Add "Set the key *Key*" as the very first step before step 3.
- Add `start(IV)` note after each `set_key()` for MAC algorithms that require an IV.
- Correct step 11: remove "Input *In* into the MAC and" — the verify step follows from the chunked updates in steps 8–10.

---

### MAC-CMAC-1
**Status:** 🔄 PROPOSED CHANGES

**Notes:**
- The spec describes a subset of the full `Message_Auth_Tests` flow and is essentially accurate for what it covers.
- Steps correctly show: create, name check, set key, compute, reset, re-key, verify.
- **Missing:** The code also runs MAC-2 (chunked) steps for CMAC when `In` is longer than 1 byte. The spec describes only the whole-message path.
- **Missing:** Clone test (from MAC-1 step 10–14) is also executed for CMAC but not shown.
- **Missing:** `has_keying_material()` state checks.

**Proposed Changes:**
- Add a note that CMAC also undergoes the chunked test (MAC-2 steps) and the clone test as part of the unified test class.

---

### MAC-HMAC-1
**Status:** 🔄 PROPOSED CHANGES

**Notes:**
- Same observations as MAC-CMAC-1: spec shows a subset of the full test flow.
- Steps correctly describe create, name check, set key, compute, reset, re-key, verify.
- **Missing:** Chunked test (MAC-2), clone test, `has_keying_material()` state checks.

**Proposed Changes:**
- Same as MAC-CMAC-1: add note that HMAC undergoes the full `Message_Auth_Tests` flow including chunked and clone tests.

---

### MAC-GMAC-1
**Status:** 🔄 PROPOSED CHANGES

**Notes:**
- The spec correctly shows GMAC's IV/nonce handling via `start(IV)` before update.
- Step 5 → Step 6 ("Reset GMAC"): the code runs **two** full compute iterations (`correct mac` and `correct mac (try 2)`) before the reset, but the spec shows only one. The "try 2" iteration is missing from the spec.
- The split-input test (step 9 in spec) correctly describes three `update()` calls. However, the code also calls `verify_mac()` after the same chunked input, which the spec does not mention.
- **Missing:** Clone test and `has_keying_material()` state checks (same as all MACs).

**Proposed Changes:**
- Add a second "Compute #2" iteration before the reset step.
- After step 9, add: "Verify the tag from the chunked input using `verify_mac()` and compare with *Out*."

---

### MAC-KMAC-1
**Status:** 🔄 PROPOSED CHANGES (includes errors)

**Notes:**
1. **Bug — Description text:** The description field says "calculates the **GMAC** tag on a test message." This is a copy-paste error; it should say "**KMAC** tag."
2. **Bug — Key bit-length:** The example vector shows `Key = 0x404142…5E5F (128 bits)`. The hex string is 32 bytes = **256 bits**, not 128 bits. The parenthetical bit count is wrong.
3. The spec correctly notes that KMAC uses a nonce (via `IV` field in the test framework) passed through `start()`.
4. Same structural omissions as MAC-GMAC-1: missing "try 2" iteration before reset, missing `verify_mac()` after split test, missing clone test.
5. The spec says KMAC is implemented in `test_mac.cpp` — this is confirmed correct; KMAC implements the MAC interface and is tested via the same `Message_Auth_Tests` class with vectors from `src/tests/data/mac/kmac.vec`.

**Proposed Changes:**
- Fix description: "GMAC" → "KMAC".
- Fix key size annotation: "(128 bits)" → "(256 bits)".
- Add missing "try 2" iteration before reset step.
- Add `verify_mac()` note after split-input step.

---

## 08_modes_of_operation.rst

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

## 09_pbkdf.rst

---

### PBKDF-1
**Status:** 🔄 PROPOSED CHANGES

**Notes:**
- The spec describes creating a `PBKDF` object and deriving a key via `PBKDF::derive_key(outlen, passphrase, salt, salt_len, iterations)`. This is accurate for the first code path.
- **Missing second test path:** The code **also** tests the same vector via the `PasswordHashFamily` / `PasswordHash` interface: `PasswordHashFamily::create(pbkdf_name)` → `from_params(iterations)` → `hash(output, passphrase, salt)`. This second path is not described at all in the spec.
- The spec lists "Hash Function" and "MAC" as separate input values. In the code, these are embedded in the algorithm name string (e.g., `PBKDF2(HMAC(SHA-1))`), not passed as distinct parameters. The spec should clarify this.

**Proposed Changes:**
- Add a step: "Also derive the key using the `PasswordHashFamily` interface with the same parameters and compare with *Out*."
- Clarify that "Hash Function" / "MAC" are part of the algorithm name, not separate inputs to the derive call.

---

### PBKDF-PBKDF2-1
**Status:** 🔄 PROPOSED CHANGES

**Notes:**
- The example vector (HMAC-SHA1, 10000 iterations, 64-bit salt `0x0001020304050607`, empty passphrase, 256-bit output) is accurate and confirmed against `src/tests/data/pbkdf/pbkdf2.vec`.
- Same missing step as PBKDF-1: the code also tests via `PasswordHashFamily::from_params(iterations)` → `hash()`. This second path is omitted from the spec.

**Proposed Changes:**
- Add step: "Also verify key derivation via the `PasswordHashFamily` interface with the same parameters."

---

### PBKDF-ARGON-1
**Status:** ✅ CONFIRMED (with minor notes)

**Notes:**
- The spec correctly identifies the test class (`Argon2_KAT_Tests`), data file (`src/tests/data/argon2.vec`), parameter ranges (M, T, P, optional Secret and AD), and the three Argon2 variants (Argon2i, Argon2d, Argon2id).
- The example vector (M=64, T=3, P=4, 256-bit passphrase, 128-bit salt, 96-bit AD, 64-bit secret) is accurate.
- Unlike PBKDF2, Argon2 is tested **only** via `PasswordHashFamily` (no legacy `PBKDF::derive_key()` path), which the spec reflects by saying "Create the Argon2[i][d] object" — this is the PasswordHash object from `PasswordHashFamily::from_params(M, T, P)`.
- Test vectors file path `src/tests/data/argon2.vec` (directly in data root, not in a subdirectory) is correct.
- The spec says "Output Length: 32 bits - 2560 bits"; this should probably say "32 **bytes** - 320 bytes" or "256 bits - 2560 bits" — the current phrasing "32 bits" (= 4 bytes) is unusual but not necessarily wrong.

---

## New Tests Found in Botan 3.12.0 Not in Spec

### 06_kdf.rst — Missing KDF algorithms/tests

| Test data file | Algorithm | Notes |
|---|---|---|
| `src/tests/data/kdf/kdf1.vec` | KDF1 (X9.63 / IEEE 1363a) | Different from KDF1 ISO 18033-2; not mentioned |
| `src/tests/data/kdf/kdf2.vec` | KDF2 (ISO 18033-2) | Not mentioned anywhere in spec |
| `src/tests/data/kdf/sp800_56a.vec` | SP800-56A KDF | ACVP-sourced vectors with HMAC and KMAC variants; not mentioned |
| `src/tests/data/kdf/x942_prf.vec` | X9.42 PRF | Not mentioned |
| `src/tests/data/kdf/hkdf_label.vec` | HKDF-Expand-Label | Tested by a **separate** class `HKDF_Expand_Label_Tests` (not `KDF_KAT_Tests`); not covered in spec |

The `KDF_KAT_Tests` class also accepts `IKM` and `XTS` optional fields used by certain KDFs; the spec does not document these fields.

### 07_mac.rst — Missing MAC algorithms

| Test data file | Algorithm | Notes |
|---|---|---|
| `src/tests/data/mac/blake2bmac.vec` | BLAKE2b-MAC | Not mentioned |
| `src/tests/data/mac/poly1305.vec` | Poly1305 | Not mentioned; requires fresh key per message |
| `src/tests/data/mac/siphash.vec` | SipHash | Not mentioned |
| `src/tests/data/mac/x919_mac.vec` | X9.19 MAC (ANSI) | Not mentioned |

### 08_modes_of_operation.rst — Missing mode tests

| Test data file | Mode | Notes |
|---|---|---|
| `src/tests/data/modes/cfb.vec` | CFB | Not mentioned in spec |
| `src/tests/data/modes/xts.vec` | XTS | Not mentioned in spec |
| `src/tests/data/modes/ctr.vec` | CTR (Cipher_Mode interface) | Separate from the stream/ctr.vec; used by `Cipher_Mode_IV_Carry_Tests` in `test_modes.cpp` — not mentioned |
| `src/tests/data/stream/chacha.vec` | ChaCha20 | Stream cipher; not mentioned |
| `src/tests/data/stream/ofb.vec` | OFB | Stream cipher mode; not mentioned |
| `src/tests/data/stream/rc4.vec` | RC4 | Stream cipher; not mentioned |
| `src/tests/data/stream/salsa20.vec` | Salsa20 | Stream cipher; not mentioned |
| `src/tests/data/stream/shake.vec` | SHAKE (XOF as stream) | Stream cipher; not mentioned |
| `Cipher_Mode_IV_Carry_Tests` | CBC, CFB, CTR IV carry-over | Separate test class for multi-message IV carry behavior; tests CBC, CFB, and CTR with sequential messages reusing the carry-over IV — not mentioned in spec |

### 09_pbkdf.rst — Missing PBKDF tests

| Test / File | Algorithm | Notes |
|---|---|---|
| `Bcrypt_PBKDF_KAT_Tests` / `src/tests/data/bcrypt_pbkdf.vec` | Bcrypt-PBKDF | Separate test class; not mentioned in spec |
| `Scrypt_KAT_Tests` / `src/tests/data/scrypt.vec` | Scrypt | Separate test class; not mentioned in spec |
| `Pwdhash_Tests` | All pwdhash families | Tests `tune_params()`, `default_params()`, and round-trip consistency for all PasswordHashFamily implementations; not mentioned |
| `PGP_S2K_Iter_Test` | OpenPGP S2K iteration encoding | Tests `RFC4880_encode_count()` and `RFC4880_decode_count()` for all 256 encoded values; not mentioned |
| `src/tests/data/pbkdf/pgp_s2k.vec` | OpenPGP S2K (KAT) | KAT for PGP S2K; tested via `PBKDF_KAT_Tests` but no spec entry for it |
