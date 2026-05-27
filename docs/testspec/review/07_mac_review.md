# Review: Message Authentication Codes Tests

**Botan Version:** 3.12.0
**Date:** 2026-05-20
**File Reviewed:** 07_mac.rst

---


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

---

## New Tests Found in Botan 3.12.0 Not in Spec

| Test ID | Algorithm | Test Data File | First Added |
|---------|-----------|----------------|-------------|
| — | BLAKE2b-MAC | `blake2bmac.vec` | Pre-3.7.1 (before 2024) |
| — | Poly1305 | `poly1305.vec` | Pre-3.7.1 (before 2024) |
| — | SipHash | `siphash.vec` | Jan 2015 (commit b07e980) |
| — | X9.19 MAC (ANSI) | `x919_mac.vec` | Pre-3.7.1 (before 2024) |

Additionally, `test_mac.cpp` contains a test for MAC algorithms without calling `start()` for non-nonce MACs, added in Sept 2023 (commit 799e720).

### Timeline Context

**SipHash** — Added in January 2015, well before the 3.7.1 baseline. The test data file and implementation were introduced when SipHash was first added to Botan. This test was missed during the previous documentation update.

**BLAKE2b-MAC, Poly1305, X9.19 MAC** — All existed before the Botan 3.7.1 baseline (~mid-2024). These algorithms and their test vectors were already present in the codebase during previous documentation cycles and were missed.

**GMAC** — While GMAC is documented in the spec (MAC-GMAC-1), the implementation was significantly refactored in October 2016 (commit 9ad816a) to use GHASH directly rather than GCM_Mode. This is not a new test but represents a substantial implementation change.

**Conclusion:** SipHash and other MAC tests were missed during previous documentation updates. They predate the 3.7.1 baseline and should have been documented earlier.
