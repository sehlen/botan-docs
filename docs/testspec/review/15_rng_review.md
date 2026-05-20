# Review: RNG Tests

**Botan Version:** 3.12.0
**Date:** 2026-05-20
**File Reviewed:** 15_rng.rst
**Source Files:** `src/tests/test_rng_kat.cpp`, `src/tests/test_rng_behavior.cpp`

> **Critical file reference error:** The RST states *"All unit tests for various RNGs are
> implemented in `src/tests/test_rngs.cpp`."* In Botan 3.12.0 this file is a **test-helper
> implementation** that registers no test cases. The actual RNG unit tests are in
> `src/tests/test_rng_behavior.cpp`. This reference must be corrected throughout the section.

---


> **Critical file reference error:** The RST states *"All unit tests for various RNGs are
> implemented in `src/tests/test_rngs.cpp`."* In Botan 3.12.0 this file is a **test-helper
> implementation** (providing `Fixed_Output_RNG` and `CTR_DRBG_AES256` for use by other test
> files). It registers no test cases and contains no `BOTAN_REGISTER_TEST` calls. The actual RNG
> unit tests (HMAC-DRBG unit tests, AutoSeeded_RNG, System_RNG, ChaCha_RNG unit tests) are in
> `src/tests/test_rng_behavior.cpp`. This reference must be corrected throughout the section.

---

### RNG-HMAC-DRBG-1
**Status:** ✅ CONFIRMED
**Source:** `HMAC_DRBG_Tests` (registered as `"hmac_drbg"`) in `test_rng_kat.cpp`
**Notes:** The spec correctly describes the KAT structure: `initialize_with(EntropyInput)`,
`add_entropy(EntropyInputReseed)`, two calls to `randomize_with_input` (first with AdditionalInput1
to discard, second with AdditionalInput2 producing the expected output). Test vectors come from
NIST CAVS 14.3. The example input/output hex values in the RST match entries in the vector file.
The test is registered with `BOTAN_REGISTER_SMOKE_TEST` (not `BOTAN_REGISTER_TEST`).

---

### RNG-HMAC-DRBG-2
**Status:** ✅ CONFIRMED
**Source:** `HMAC_DRBG_Unit_Tests::test_max_number_of_bytes_per_request()` in
`test_rng_behavior.cpp`
**Notes:** All four spec steps are present in the code:
1. MNBPR = 0 → `test_throws` (constructor rejects it).
2. MNBPR = 64 KiB + 1 → `test_throws` (constructor rejects it).
3. HMAC_DRBG instantiated with MNBPR = 64 and RI = 1.
4. Request of 65 bytes → 2 underlying RNG calls (split into ≤ 64-byte chunks).
The file reference in the RST should be updated from `test_rngs.cpp` to `test_rng_behavior.cpp`.

---

### RNG-HMAC-DRBG-3
**Status:** ✅ CONFIRMED
**Source:** `HMAC_DRBG_Unit_Tests::test_security_level()` in `test_rng_behavior.cpp`
**Notes:** Hash functions and corresponding security levels listed in the spec
(SHA-1 → 128, SHA-224 → 192, SHA-256/SHA-512-256/SHA-384/SHA-512 → 256) exactly match the
`approved_hash_fns` / `security_strengths` vectors in the source. The file reference in the RST
should be updated from `test_rngs.cpp` to `test_rng_behavior.cpp`.

---

### RNG-HMAC-DRBG-4
**Status:** ✅ CONFIRMED
**Source:** `HMAC_DRBG_Unit_Tests::test_reseed_kat()` in `test_rng_behavior.cpp`
**Notes:** All input/output hex values in the spec match the code exactly:
- SeedData: `0x00112233...EEFF` (32 bytes, repeated) ✓
- OutFirstRequest: `48D3B45AAB65EF92CCFCB9427EF20C90297065ECC1B8A525BFE4DC6FF36D0E38` ✓
- OutSecondRequest: `2F8FCA696832C984781123FD64F4B20C7379A25C87AB29A21C9BF468B0081CE2` ✓
- ReseedInterval = 2 ✓
The spec correctly notes that the second request triggers an automatic reseed (verifiable via
`counting_rng.randomize_count()` jumping from 0 to 1). File reference must be corrected.

---

### RNG-AUTO-RNG-1
**Status:** 🔄 PROPOSED CHANGES
**Source:** `AutoSeeded_RNG_Tests` (registered as `"auto_rng_unit"`) in `test_rng_behavior.cpp`
**Notes:**
1. **Step order mismatch:** The spec lists Step 1 as "empty entropy sources → throws", Step 2 as
   "Null_RNG → throws". The code tests **Null_RNG first, empty entropy sources second**. The
   logical meaning is the same but the order should match for traceability.
2. **Exception type imprecision:** The spec says each of steps 1–3 "throws a PRNG_Unseeded
   exception". The code uses `test_success` after a `try/catch(Botan::Not_Implemented&)` or
   `catch(std::exception&)` — not specifically `PRNG_Unseeded`. The actual exception is
   implementation-defined; the spec should not be overly specific.
3. **Steps 10 and 11 are duplicates** (both say "Check that the AutoSeeded_RNG is seeded").
   These correspond to two consecutive `is_seeded()` checks in the code at lines ~737/739. One
   can be removed from the spec.
4. **Missing edge-case steps:** The code also iterates over buffer sizes 0–4095, calling
   `randomize` and `add_entropy` for each — a coverage check not described in the spec.
5. **File reference:** Must be corrected from `test_rngs.cpp` to `test_rng_behavior.cpp`.

**Proposed Changes:**
- Swap Steps 1 and 2 to match code order (or add a note that order is implementation-dependent).
- Replace "throws a PRNG_Unseeded exception" with "throws an exception (construction fails)".
- Remove the duplicate seeded-check step (retain only one instance).
- Add a step: "Verify that the RNG accepts arbitrary-length input and output buffers (edge-case
  sweep over sizes 0–4095)."
- Correct the file reference.

---

### RNG-SYS-RNG-1
**Status:** ✅ CONFIRMED
**Source:** `System_RNG_Tests` (registered as `"system_rng"`) in `test_rng_behavior.cpp`
**Notes:** All steps in the spec are confirmed:
- Name is non-empty ✓
- Always seeded; `clear()` is a no-op that does not unseed ✓
- Reseed from global entropy sources ✓
- Buffer sweep (1–128 bytes) with `randomize` and `add_entropy` ✓
- 64-bit regression test (4 GiB + 1024 bytes) guarded by `run_long_tests() &&
  run_memory_intensive_tests() && (sizeof(size_t) > 4)` ✓

The precondition "(partially) 64-bit system" is accurate.
**File reference must be corrected** from `test_rngs.cpp` to `test_rng_behavior.cpp`.

---

---

## New Tests Found in Botan 3.12.0 Not in Spec

| Function | Registration | Description |
|---|---|---|
| `ChaCha_RNG_Tests` | `"chacha_rng"` (test_rng_kat.cpp) | KAT tests for ChaCha_RNG using vector file `rng/chacha_rng.vec`. |
| `ChaCha_RNG_Unit_Tests` | `"chacha_rng_unit"` (test_rng_behavior.cpp) | Unit tests for ChaCha RNG paralleling HMAC_DRBG unit tests. |
| `hmac_drbg_multiple_requests` | `"hmac_drbg_multi_request"` (test_rng_behavior.cpp) | Tests that a bulk randomize request produces the same output as the equivalent split into max-size chunks, both with and without additional input. |
| `Processor_RNG_Tests` | `"processor_rng"` (test_rng_behavior.cpp) | Tests CPU hardware RNG (e.g., RDRAND). |
