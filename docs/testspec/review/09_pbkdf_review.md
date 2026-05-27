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


| Test / File | Algorithm | First Added |
|---|---|---|
| `Bcrypt_PBKDF_KAT_Tests` / `src/tests/data/bcrypt_pbkdf.vec` | Bcrypt-PBKDF | June 2019 (commit 184a782) |
| `Scrypt_KAT_Tests` / `src/tests/data/scrypt.vec` | Scrypt | May 2018 (commit 556aac9) |
| `Pwdhash_Tests` | All pwdhash families | May 2019 (commit 1d283a6) |
| `PGP_S2K_Iter_Test` | OpenPGP S2K iteration encoding | Pre-3.7.1 (before 2024) |
| `src/tests/data/pbkdf/pgp_s2k.vec` | OpenPGP S2K (KAT) | Pre-3.7.1 (before 2024) |

### Timeline Context

**Scrypt** — Added in May 2018 (commit 556aac9), well before the 3.7.1 baseline. Scrypt is a widely-used memory-hard PBKDF designed to resist brute-force attacks with custom hardware. It was added to Botan years ago and was missed during previous documentation updates.

**Bcrypt-PBKDF** — Added in June 2019 (commit 184a782), also before the 3.7.1 baseline. This is the OpenBSD Bcrypt-based PBKDF used in OpenSSH private key encryption. The implementation and tests predate the documentation baseline and were missed.

**Pwdhash_Tests** — Added in May 2019 (commit 1d283a6) alongside Argon2 support. This test class verifies that all PasswordHashFamily implementations support the common interface methods (`tune_params()`, `default_params()`, round-trip consistency). It was added as part of the Argon2 implementation but tests all PBKDF implementations. This predates 3.7.1 and was missed.

**PGP S2K** — Both the iteration encoding test and KAT vectors existed before the 3.7.1 baseline. OpenPGP S2K (String-to-Key) is a PBKDF variant used in PGP/GPG key derivation. These tests have been in Botan for years and were overlooked.

**Conclusion:** All these PBKDF tests were missed during previous documentation cycles. They existed before the 3.7.1 baseline and should have been documented earlier.
