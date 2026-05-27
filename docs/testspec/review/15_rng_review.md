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
1. **Step order mismatch:** The spec lists Step 1 as "empty entropy sources → throws",
   Step 2 as "Null_RNG → throws". The code tests **Null_RNG first, empty entropy sources
   second**. The logical meaning is the same but the order should match for traceability.
2. **BOTAN_HAS_ENTROPY_SOURCE guard:** The entropy-source construction-failure cases
   (empty entropy sources, and useless RNG + useless entropy source) are guarded by
   `BOTAN_HAS_ENTROPY_SOURCE`. This conditional should be noted in the spec.
3. **Keep PRNG_Unseeded expectation:** All three construction-failure cases (Null_RNG,
   empty entropy sources, useless RNG + useless entropy source) explicitly catch
   `Botan::PRNG_Unseeded&`. The spec's expectation of `PRNG_Unseeded` is **correct and
   should be kept**. The previous draft proposed weakening this to a generic exception —
   that proposal was incorrect.
4. **Steps 10 and 11 are duplicates** (both say "Check that the AutoSeeded_RNG is seeded").
   These correspond to two consecutive `is_seeded()` checks in the code. One can be
   removed from the spec.
5. **Missing edge-case steps:** The code also iterates over buffer sizes 0–4095, calling
   `randomize` and `add_entropy` for each — a coverage check not described in the spec.
6. **File reference:** Must be corrected from `test_rngs.cpp` to `test_rng_behavior.cpp`.

**Proposed Changes:**
- Swap Steps 1 and 2 to match code order: Null_RNG first, empty entropy sources second.
- Add a note that the entropy-source cases (steps 2 and 3) are guarded by
  `BOTAN_HAS_ENTROPY_SOURCE`.
- Keep the `PRNG_Unseeded` exception type for all three construction-failure steps.
- Remove the duplicate seeded-check step (retain only one instance of "Check that the
  AutoSeeded_RNG is seeded").
- Add a step: "Verify that the RNG accepts arbitrary-length input and output buffers
  (edge-case sweep over sizes 0–4095 calling both `randomize()` and `add_entropy()`)."
- Correct the file reference from `test_rngs.cpp` to `test_rng_behavior.cpp`.

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

| Function | Registration | First Added |
|---|---|---|
| `ChaCha_RNG_Tests` | `"chacha_rng"` (test_rng_kat.cpp) | Aug 2017 (commit 7edeec6) |
| `ChaCha_RNG_Unit_Tests` | `"chacha_rng_unit"` (test_rng_behavior.cpp) | Aug 2017 (commit 7edeec6) |
| `hmac_drbg_multiple_requests` | `"hmac_drbg_multi_request"` (test_rng_behavior.cpp) | Pre-3.7.1 (before 2024) |
| `Processor_RNG_Tests` | `"processor_rng"` (test_rng_behavior.cpp) | May 2020 (commit ad851c2) |

### Timeline Context

**ChaCha_RNG** — Both KAT and unit tests were added in August 2017 (commit 7edeec6), well before the 3.7.1 baseline. ChaCha_RNG is a high-performance stream cipher-based RNG using ChaCha20. The implementation includes both KAT tests (using test vectors from `rng/chacha_rng.vec`) and comprehensive unit tests (security level, reseed behavior, max bytes per request) paralleling the HMAC_DRBG test structure. This predates the 3.7.1 baseline and was missed during previous documentation.

**Processor_RNG** — Added in May 2020 (commit ad851c2), also before the 3.7.1 baseline. This tests CPU hardware RNG capabilities (e.g., Intel RDRAND/RDSEED, ARM RNDR). The test verifies: name reporting, always-seeded status, clear() no-op behavior, reseed from entropy sources, and buffer sweeps for both randomize() and add_entropy(). This predates 3.7.1 and was missed.

**hmac_drbg_multiple_requests** — Existed before the 3.7.1 baseline. This test verifies that requesting a large block of random data produces identical output whether generated in one bulk call or split into multiple max-size chunks. Tests both with and without additional input data. This test was overlooked during previous documentation updates.

**Conclusion:** All these RNG tests were missed during previous documentation cycles. They existed before the 3.7.1 baseline and should have been documented earlier.
