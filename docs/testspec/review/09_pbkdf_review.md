# Review: Password-Based KDF Tests

**Botan Version:** 3.12.0
**Date:** 2026-05-20
**File Reviewed:** 09_pbkdf.rst

---


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

---

## New Tests Found in Botan 3.12.0 Not in Spec


| Test / File | Algorithm | Notes |
|---|---|---|
| `Bcrypt_PBKDF_KAT_Tests` / `src/tests/data/bcrypt_pbkdf.vec` | Bcrypt-PBKDF | Separate test class; not mentioned in spec |
| `Scrypt_KAT_Tests` / `src/tests/data/scrypt.vec` | Scrypt | Separate test class; not mentioned in spec |
| `Pwdhash_Tests` | All pwdhash families | Tests `tune_params()`, `default_params()`, and round-trip consistency for all PasswordHashFamily implementations; not mentioned |
| `PGP_S2K_Iter_Test` | OpenPGP S2K iteration encoding | Tests `RFC4880_encode_count()` and `RFC4880_decode_count()` for all 256 encoded values; not mentioned |
| `src/tests/data/pbkdf/pgp_s2k.vec` | OpenPGP S2K (KAT) | KAT for PGP S2K; tested via `PBKDF_KAT_Tests` but no spec entry for it |
