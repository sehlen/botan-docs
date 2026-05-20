# Review: PKCS#11 Tests

**Botan Version:** 3.12.0
**Date:** 2026-05-20
**Files Reviewed:** `docs/testspec/src/10_pkcs11.rst`
**Source Reviewed:** `src/tests/test_pkcs11_high_level.cpp`, `src/tests/test_pkcs11.h`

---

## Preliminary Notes

- All PKCS#11 tests require a vendor-specific PKCS#11 module (HSM or SoftHSM emulator). They are not run during the regular test suite; they must be invoked with `--pkcs11-lib=<PATH>`.
- The token under test must have User PIN set to `123456` and SO PIN set to `12345678` before running. This is confirmed by `test_pkcs11.h` (`PKCS11_USER_PIN = "123456"`, `PKCS11_SO_PIN = "12345678"`, `PKCS11_TEST_USER_PIN = "654321"`, `PKCS11_TEST_SO_PIN = "87654321"`).
- The source registers **seven** test groups: `pkcs11-module`, `pkcs11-slot`, `pkcs11-session`, `pkcs11-object`, `pkcs11-rsa`, `pkcs11-ecdsa`, `pkcs11-ecdh`, `pkcs11-rng`, `pkcs11-x509`, `pkcs11-manage`. The spec is missing the entire **`pkcs11-object`** group (see "New Tests Found" below).

---

## Module Tests

### PKCS11-MODULE-1
**Status:** ✅ CONFIRMED  
**Notes:** Corresponds to the first assertion in `test_module_ctor`: `result.test_throws("Module ctor fails for non existent path", []() { Module("/a/b/c"); })`. Input value "/a/b/c" matches exactly.

---

### PKCS11-MODULE-2
**Status:** ✅ CONFIRMED  
**Notes:** Corresponds to the second part of `test_module_ctor`: after the negative test, a `Module` is successfully constructed from the valid `pkcs11_lib()` path, and `test_success` is recorded. One source function covers both MODULE-1 and MODULE-2.

---

### PKCS11-MODULE-3
**Status:** ✅ CONFIRMED  
**Notes:** Corresponds to `test_module_reload`. The three spec steps (load, reload, retrieve info) match the source: `module.reload()` then `module.get_info()`.

---

### PKCS11-MODULE-4
**Status:** ✅ CONFIRMED  
**Notes:** Corresponds to `test_multiple_modules`, which calls `result.test_throws("Module ctor fails if module is already initialized", ...)`.

---

### PKCS11-MODULE-5
**Status:** 🔄 PROPOSED CHANGES  
**Notes:** MODULE-5 is a **duplicate of MODULE-4**. Both have identical descriptions ("Attempt to load the same module twice"), identical expected outputs, and near-identical steps. There is only a single test function for this behaviour (`test_multiple_modules`). MODULE-5 should either be removed or differentiated.  
**Proposed Change:** Remove MODULE-5. The behaviour is already fully covered by MODULE-4.

---

### PKCS11-MODULE-6
**Status:** ✅ CONFIRMED  
**Notes:** Corresponds to `test_module_get_info`. The source asserts `info.cryptokiVersion.major != 0`, which matches the spec step "check that the Cryptoki major version is not 0".

---

## Slot Tests

### PKCS11-SLOT-1
**Status:** ✅ CONFIRMED  
**Notes:** Corresponds to `test_slot_get_available_slots`. Source asserts `slot_vec.size() >= 1`.

---

### PKCS11-SLOT-2
**Status:** ✅ CONFIRMED  
**Notes:** Corresponds to `test_slot_ctor`. Source checks `slot.slot_id() == slot_vec.at(0)`.

---

### PKCS11-SLOT-3
**Status:** ✅ CONFIRMED  
**Notes:** Corresponds to `test_get_slot_info`. Source checks that `slotDescription` is not empty.

---

### PKCS11-SLOT-4
**Status:** ✅ CONFIRMED  
**Notes:** Corresponds to `test_slot_invalid_id`. Source finds an ID absent from both `Slot::get_available_slots(module, true)` and `get_available_slots(module, false)`, constructs a Slot, then asserts that `get_slot_info()` throws.

---

### PKCS11-SLOT-5
**Status:** ✅ CONFIRMED  
**Notes:** Corresponds to `test_get_token_info`. Source checks that `TokenInfo.label` is not empty.

---

### PKCS11-SLOT-6
**Status:** ✅ CONFIRMED  
**Notes:** Corresponds to `test_get_mechanism_list`. Source asserts `mechanisms.size() >= 1`.

---

### PKCS11-SLOT-7
**Status:** ✅ CONFIRMED  
**Notes:** Corresponds to `test_get_mechanisms_info`. Source calls `slot.get_mechanism_info(MechanismType::RsaPkcsKeyPairGen)` and asserts success.

---

## Session Tests

### PKCS11-SESSION-1
**Status:** ✅ CONFIRMED  
**Notes:** Corresponds to the first block in `test_session_ctor`: `Session read_only_session(slot, true)`.

---

### PKCS11-SESSION-2
**Status:** ✅ CONFIRMED  
**Notes:** Corresponds to `test_session_ctor_invalid_slot`. Source uses `get_invalid_slot_id()` and then asserts `Session(slot, true)` throws.

---

### PKCS11-SESSION-3
**Status:** ✅ CONFIRMED  
**Notes:** Corresponds to the second block in `test_session_ctor`: `Session read_write_session(slot, false)`.

---

### PKCS11-SESSION-4
**Status:** ✅ CONFIRMED  
**Notes:** Corresponds to the third block in `test_session_ctor`: `Session read_write_session2(slot, flags, nullptr, nullptr)` with `flags = PKCS11::flags(Flag::SerialSession | Flag::RwSession)`.

---

### PKCS11-SESSION-5
**Status:** ✅ CONFIRMED  
**Notes:** Corresponds to the fourth block in `test_session_ctor`: opening both a read-only and read-write session on the same slot concurrently.

---

### PKCS11-SESSION-6
**Status:** ✅ CONFIRMED  
**Notes:** Corresponds to `test_session_release`. Source calls `session.release()` to obtain a raw `SessionHandle`, then constructs a new `Session(slot, handle)`.

---

### PKCS11-SESSION-7
**Status:** ✅ CONFIRMED  
**Notes:** Corresponds to the user-login portion of `test_session_login_logout`: `session.login(UserType::User, PIN())` followed by `session.logoff()`.

---

### PKCS11-SESSION-8
**Status:** 🔄 PROPOSED CHANGES  
**Notes:** Corresponds to the SO-login portion of `test_session_login_logout`: `session.login(UserType::SO, SO_PIN())`. The spec step says "Log into the Session with the SO PIN" (no log-off step shown). The source does not call `logoff()` after the SO login. The spec is accurate, but it should note that SESSION-7 and SESSION-8 are sub-cases of a single source function (`test_session_login_logout`), not two independent test runs.  
**Additional Issue:** The source also includes `test_session_info` (checks `info.slotID` and session state transitions `RwPublicSession` → `RwUserFunctions`), which has **no corresponding spec test case**. A new SESSION-9 test case should be added.  
**Proposed Change:** Add PKCS11-SESSION-9 covering `test_session_info` (see "New Tests Found" section).

---

## RSA Tests

### PKCS11-RSA-1
**Status:** ✅ CONFIRMED  
**Notes:** Corresponds to `test_rsa_privkey_import`. The spec steps match. The source also calls `pk.check_key(*rng, true)` (key self-test) which is not explicitly mentioned as a step in the spec but is a minor omission.

---

### PKCS11-RSA-2
**Status:** ✅ CONFIRMED  
**Notes:** Corresponds to `test_rsa_privkey_export`. Spec correctly requires `extractable=true` and `sensitive=false`. The source also runs `pk.check_key` and `exported.check_key`, minor omissions in spec steps.

---

### PKCS11-RSA-3
**Status:** ✅ CONFIRMED  
**Notes:** Corresponds to `test_rsa_pubkey_import`. Spec step says "to not be a private token object and to be a decryption key"; source sets `set_encrypt(true)` and `set_private(false)` which is consistent (encrypt for public key rather than decrypt — the spec says "decryption key" but public keys are used for encryption; source sets `set_encrypt(true)`).  
**Minor issue:** Spec says "decryption key" for the public key import properties but `test_rsa_pubkey_import` sets `props.set_encrypt(true)` (encrypt permission), not decrypt. The spec should read "encryption key" for the public key.  
**Proposed Change:** In RSA-3 step 2, change "to be a decryption key" to "to be an encryption key" to match the source's `props.set_encrypt(true)`.

---

### PKCS11-RSA-4
**Status:** ✅ CONFIRMED  
**Notes:** Corresponds to `test_rsa_generate_private_key`. Properties listed in spec match the source exactly.

---

### PKCS11-RSA-5
**Status:** ✅ CONFIRMED  
**Notes:** Corresponds to `test_rsa_generate_key_pair`. All properties listed in spec match `generate_rsa_keypair()` helper.

---

### PKCS11-RSA-6
**Status:** ✅ CONFIRMED  
**Notes:** Corresponds to the `encrypt_and_decrypt(plaintext, "Raw", false)` call in `test_rsa_encrypt_decrypt`. Plaintext is 256 bytes (2048 bits), matching "0x000102030405060708090A0B… (2048 bits)". No blinding applies to Raw (the `false` parameter).

---

### PKCS11-RSA-7
**Status:** 🔄 PROPOSED CHANGES  
**Notes:** Corresponds to the PKCS#1 v1.5 portion of `test_rsa_encrypt_decrypt`. The source runs encrypt/decrypt **twice**: once without userspace blinding (`false`) and once **with** userspace blinding (`true`) by calling `keypair.second.set_use_software_padding(blinding)`. The spec only documents one scenario and does not mention the blinding variant.  
**Proposed Change:** Add a step noting that the decrypt is performed both with and without userspace blinding (`set_use_software_padding(false)` and `set_use_software_padding(true)`), or split into two sub-tests (RSA-7a/RSA-7b).

---

### PKCS11-RSA-8
**Status:** 🔄 PROPOSED CHANGES  
**Notes:** Same issue as RSA-7: `test_rsa_encrypt_decrypt` also calls `encrypt_and_decrypt(plaintext, "OAEP(SHA-1)", false)` and `encrypt_and_decrypt(plaintext, "OAEP(SHA-1)", true)`. The spec does not mention the blinding variant.  
**Proposed Change:** Same as RSA-7 — document or split the blinding and non-blinding sub-cases.

---

### PKCS11-RSA-9
**Status:** ✅ CONFIRMED  
**Notes:** Corresponds to `sign_and_verify("Raw", false)` in `test_rsa_sign_verify`.

---

### PKCS11-RSA-10
**Status:** ✅ CONFIRMED  
**Notes:** Corresponds to `sign_and_verify("PKCS1v15(SHA-256)", false)` (single-part) in `test_rsa_sign_verify`.

---

### PKCS11-RSA-11
**Status:** ✅ CONFIRMED  
**Notes:** Corresponds to `sign_and_verify("PSS(SHA-256)", false)` (single-part) in `test_rsa_sign_verify`.

---

### PKCS11-RSA-12
**Status:** ✅ CONFIRMED  
**Notes:** Corresponds to `sign_and_verify("PKCS1v15(SHA-256)", true)` (multi-part, `multipart=true`) in `test_rsa_sign_verify`. Multi-part processing splits the message in half using `signer.update(...)` / `signer.sign_message(...)`.

---

### PKCS11-RSA-13
**Status:** ✅ CONFIRMED  
**Notes:** Corresponds to `sign_and_verify("PSS(SHA-256)", true)` (multi-part) in `test_rsa_sign_verify`.

---

## ECDSA Tests

### PKCS11-ECDSA-1
**Status:** ✅ CONFIRMED  
**Notes:** Corresponds to `test_ecdsa_privkey_import`. Spec steps match source properties. Source additionally calls `pk.set_public_point(priv_key._public_ec_point())` and runs `pk.check_key(*rng, false)` — minor omission in spec steps.

---

### PKCS11-ECDSA-2
**Status:** ✅ CONFIRMED  
**Notes:** Corresponds to `test_ecdsa_privkey_export`. Spec steps match. Source additionally calls `exported.check_key` and compares `private_key_bits()` — minor omission in spec steps.

---

### PKCS11-ECDSA-3
**Status:** ✅ CONFIRMED  
**Notes:** Corresponds to `test_ecdsa_pubkey_import`. Spec steps match source properties.

---

### PKCS11-ECDSA-4
**Status:** ✅ CONFIRMED  
**Notes:** Corresponds to `test_ecdsa_pubkey_export`. Spec says "Export the public key and compare it with the generated public key", but the source does not perform a comparison (it calls `pk.export_key()` and records success only). The "compare" step is not present in the source.  
**Proposed Change:** Remove the comparison assertion from the spec steps for ECDSA-4, or note that only export success is verified (no byte-level comparison in source).

---

### PKCS11-ECDSA-5
**Status:** 🔄 PROPOSED CHANGES  
**Notes:** Corresponds to `test_ecdsa_generate_private_key`. The spec **Input Values** field says "Curve = secp256r1, brainpool512r1", but the source only generates on **secp256r1**:
```cpp
const PKCS11_ECDSA_PrivateKey pk(test_session.session(), EC_Group::from_name("secp256r1").DER_encode(), props);
```
The brainpool512r1 curve is only used in `test_ecdsa_generate_keypair` (ECDSA-6).  
**Proposed Change:** Change Input Values for ECDSA-5 to "Curve = secp256r1" only.

---

### PKCS11-ECDSA-6
**Status:** 🔄 PROPOSED CHANGES  
**Notes:** Corresponds to `test_ecdsa_generate_keypair`. The spec steps list only "curve = secp256r1", but the source iterates **both secp256r1 and brainpool512r1**:
```cpp
curves.push_back("secp256r1");
curves.push_back("brainpool512r1");
for(const auto& curve : curves) { ... }
```
**Proposed Change:** Update ECDSA-6 to reflect that the test runs for both secp256r1 and brainpool512r1 curves.

---

### PKCS11-ECDSA-7
**Status:** 🔄 PROPOSED CHANGES  
**Notes:** Corresponds to `test_ecdsa_sign_verify` (which calls `test_ecdsa_sign_verify_core` with `EC_Group_Encoding::NamedCurve`). Several discrepancies:

1. **Description mismatch:** The spec says "Sign and verify a message in the token with **no padding**", but the source tests multiple padding/hash combinations:
   - `sign_and_verify("SHA-256", Botan::Signature_Format::Standard, true)` — with SHA-256
   - `sign_and_verify("SHA-256", Botan::Signature_Format::DerSequence, true)` — SHA-256 with DER encoding
   - `sign_and_verify("Raw", Botan::Signature_Format::Standard, true/false)` — Raw
   
   The "SHA-256" variants are skipped for SoftHSMv2 (manufacturer check in source). "No padding" only describes the "Raw" variant.

2. **Both secp256r1 and brainpool512r1** are tested (not stated in spec).

3. **Cross-verification** against software implementation is performed in source but not mentioned in spec.

4. **Missing: `test_ecdsa_curve_import`** — this is the 8th ECDSA test registered in the source, testing explicit curve parameter encoding (`EC_Group_Encoding::Explicit`). It is completely absent from the spec.

**Proposed Changes:**
- Update description to: "Sign and verify a message in the token using SHA-256, SHA-256/DER, and Raw padding across secp256r1 and brainpool512r1 curves".
- Add note about SoftHSM SHA-256 limitation.
- Add a new PKCS11-ECDSA-8 test case for `test_ecdsa_curve_import`.

---

## ECDH Tests

### PKCS11-ECDH-1
**Status:** ✅ CONFIRMED  
**Notes:** Corresponds to `test_ecdh_privkey_import`. Spec steps and properties match source exactly.

---

### PKCS11-ECDH-2
**Status:** ✅ CONFIRMED  
**Notes:** Corresponds to `test_ecdh_privkey_export`. Spec steps match source.

---

### PKCS11-ECDH-3
**Status:** ✅ CONFIRMED  
**Notes:** Corresponds to `test_ecdh_pubkey_import`. Spec steps match source.

---

### PKCS11-ECDH-4
**Status:** ✅ CONFIRMED  
**Notes:** Corresponds to `test_ecdh_pubkey_export`. Spec says "Export the public key" — source calls `pk.export_key()` and records success. Consistent.

---

### PKCS11-ECDH-5
**Status:** ✅ CONFIRMED  
**Notes:** Corresponds to `test_ecdh_generate_private_key`. Uses secp256r1, matching spec.

---

### PKCS11-ECDH-6
**Status:** ✅ CONFIRMED  
**Notes:** Corresponds to `test_ecdh_generate_keypair`. Labels in spec ("Botan test ECDH key1_PUB_KEY" / "_PRIV_KEY") match exactly.

---

### PKCS11-ECDH-7
**Status:** ✅ CONFIRMED  
**Notes:** Corresponds to `test_ecdh_derive`. Two keypairs are generated ("Botan test ECDH key1" / "Botan test ECDH key2"), `PK_Key_Agreement` with "Raw" KDF is used (SoftHSMv2 only supports `CKD_NULL` — this is noted in source comment but missing from spec). `derive_key(32, ...)` produces 256 bits, matching spec.  
**Minor issue:** Spec does not mention the SoftHSM/Raw KDF limitation. Worth noting as a precondition/note.

---

## Random Generator Tests

### PKCS11-RNG-1
**Status:** ✅ CONFIRMED  
**Notes:** Corresponds to `test_rng_generate_random`. Source requests 20 random bytes and asserts they are not all zeroes. Spec matches.

---

### PKCS11-RNG-2
**Status:** ✅ CONFIRMED  
**Notes:** Corresponds to `test_rng_add_entropy`. Source confirms RNG is seeded after construction, calls `clear()` (ignored), calls `reseed_from_sources()` (returns 0, ignored), and calls `add_entropy()`. All spec steps match.  
**Minor issue:** The `reseed_from_sources()` step is conditional on `BOTAN_HAS_ENTROPY_SOURCE` in the source, which is not mentioned in the spec.

---

### PKCS11-RNG-3
**Status:** 🔄 PROPOSED CHANGES  
**Notes:** Corresponds to `test_pkcs11_hmac_drbg`. Spec steps match the source logic. However:

1. **Conditional compilation:** In the source, `test_pkcs11_hmac_drbg` is only included when `BOTAN_HAS_HMAC_DRBG && BOTAN_HAS_SHA2_64` are both defined. If either is absent the test is silently skipped. This precondition is not documented in the spec.
2. **Execution order:** In the `PKCS11_RNG_Tests::run()` vector, the HMAC_DRBG test is registered **first** (before `test_rng_generate_random` and `test_rng_add_entropy`). The spec lists it as RNG-3 (last). While ordering in spec documentation is independent of execution, this may cause confusion.

**Proposed Change:** Add a note to RNG-3 preconditions: "Requires `BOTAN_HAS_HMAC_DRBG` and `BOTAN_HAS_SHA2_64` to be enabled at compile time."

---

## X.509 Tests

### PKCS11-X509-1
**Status:** 🔄 PROPOSED CHANGES  
**Notes:** Corresponds to `test_x509_import`. Two discrepancies found:

1. **Wrong precondition — session type:** The spec says "A **read-only** session is open with the token using the User PIN". However, the source uses `TestSession test_session(true)`, which (per `TestSession` constructor) opens a **read-write** session (`Session(*m_slot, false)` where `false` = read-write) and logs in as User. Importing a token object requires a read-write session.  
   **Proposed Change:** Change precondition to "A **read-write** session is open with the token using the User PIN".

2. **Wrong certificate path:** The spec lists the Certificate File as `"src/tests/data/nist_x509/test01/end.crt"`. The source calls `Test::data_file("x509/nist/test01/end.crt")`, which resolves to `src/tests/data/x509/nist/test01/end.crt`. The `nist_x509` directory name in the spec is incorrect.  
   **Proposed Change:** Update Certificate File path to `"src/tests/data/x509/nist/test01/end.crt"`.

3. **Conditional compilation:** The actual import code is wrapped in `#if defined(BOTAN_TARGET_OS_HAS_FILESYSTEM)`. On platforms without a filesystem, the test returns an empty result. This is not noted in the spec.  
   **Proposed Change:** Add precondition note: "Requires filesystem access (`BOTAN_TARGET_OS_HAS_FILESYSTEM`)."

---

## Token Management Tests

### PKCS11-MGMT-1
**Status:** ✅ CONFIRMED  
**Notes:** Corresponds to `test_set_pin`. Source calls `PKCS11::set_pin(slot, SO_PIN(), TEST_PIN())` (654321) then `PKCS11::set_pin(slot, SO_PIN(), PIN())` (back to 123456). Spec steps match exactly.

---

### PKCS11-MGMT-2
**Status:** ✅ CONFIRMED  
**Notes:** Corresponds to `test_initialize`. Source calls `PKCS11::initialize_token(slot, "Botan PKCS#11 tests", SO_PIN(), PIN())`. Spec steps match.

---

### PKCS11-MGMT-3
**Status:** ✅ CONFIRMED  
**Notes:** Corresponds to `test_change_pin`. Source calls `PKCS11::change_pin(slot, PIN(), TEST_PIN())` then `PKCS11::change_pin(slot, TEST_PIN(), PIN())`. Spec steps match exactly.

---

### PKCS11-MGMT-4
**Status:** ✅ CONFIRMED  
**Notes:** Corresponds to `test_change_so_pin`. Source calls `PKCS11::change_so_pin(slot, SO_PIN(), TEST_SO_PIN())` (87654321) then reverts. Spec steps match exactly.

---

## New Tests Found in Botan 3.12.0 Not in Spec

### 1. Object Tests Group (`pkcs11-object`) — Entirely Missing

The source registers a complete `Object_Tests` class (`BOTAN_REGISTER_SERIALIZED_TEST("pkcs11", "pkcs11-object", Object_Tests)`) with five test functions not covered anywhere in the spec:

| Suggested ID | Function | Description |
|---|---|---|
| PKCS11-OBJECT-1 | `test_attribute_container` | Tests `AttributeContainer` operations: adding class, string, binary, bool, numeric attributes; overwriting an existing numeric attribute; verifying count and individual attribute types/values. Requires `BOTAN_HAS_ASN1` for some parts (numeric-only part has no conditional). |
| PKCS11-OBJECT-2 | `test_create_destroy_data_object` | Creates a data object in the token with label, value, application, object ID properties; verifies creation; destroys it. Requires `BOTAN_HAS_ASN1`. |
| PKCS11-OBJECT-3 | `test_get_set_attribute_values` | Creates a data object; reads its `Label` attribute; modifies the label; re-reads and verifies the updated value. Requires `BOTAN_HAS_ASN1`. |
| PKCS11-OBJECT-4 | `test_object_finder` | Creates a data object; uses `ObjectFinder` and `Object::search<Object>()` to find it by label; verifies the found object matches by Application and Label attributes. Requires `BOTAN_HAS_ASN1`. |
| PKCS11-OBJECT-5 | `test_object_copy` | Creates a data object; copies it with a new label via `data_obj.copy(copy_attributes)`; verifies the copy is findable by `ObjectFinder`; destroys both. Requires `BOTAN_HAS_ASN1`. |

### 2. PKCS11-SESSION-9 (Missing: `test_session_info`)

The source's `Session_Tests` class includes `test_session_info`, which verifies:
- After opening an R/W session: `info.slotID == slot_vec.at(0)` and `info.state == SessionState::RwPublicSession`
- After logging in as User: `info.state == SessionState::RwUserFunctions`
- That login/logout and SO login succeed thereafter

This is registered and runs as part of `pkcs11-session` but has no corresponding spec test case.

### 3. PKCS11-ECDSA-8 (Missing: `test_ecdsa_curve_import`)

The source's `PKCS11_ECDSA_Tests` class includes an 8th test `test_ecdsa_curve_import`, which calls `test_ecdsa_sign_verify_core(EC_Group_Encoding::Explicit, ...)`. This differs from ECDSA-7 (`test_ecdsa_sign_verify`) by passing **explicit curve parameters** (DER-encoded full curve parameters) to the PKCS#11 library instead of a named curve OID. It exercises the same sign/verify flow over secp256r1 and brainpool512r1 but validates that the token can handle explicitly-encoded EC group parameters. This test is completely absent from the spec.

---

## Summary of Required Changes

| Issue | Severity | Action |
|---|---|---|
| MODULE-5 duplicates MODULE-4 | Medium | Remove MODULE-5 |
| RSA-3 says "decryption key" for public key import | Minor | Change to "encryption key" |
| RSA-7/RSA-8 missing blinding variants | Minor | Document both non-blinding and blinding decrypt sub-cases |
| ECDSA-5 incorrectly lists brainpool512r1 | Minor | Remove brainpool512r1 from ECDSA-5 Input Values |
| ECDSA-6 missing brainpool512r1 | Minor | Add brainpool512r1 to ECDSA-6 steps |
| ECDSA-7 description "no padding" is incomplete | Medium | Update description; add SHA-256 and DerSequence variants |
| ECDSA-7 missing SoftHSM conditional note | Minor | Add note about SoftHSM SHA-256 limitation |
| X509-1 precondition says "read-only" session | Medium | Change to "read-write" session |
| X509-1 wrong certificate file path | Medium | Fix to `src/tests/data/x509/nist/test01/end.crt` |
| X509-1 missing filesystem precondition | Minor | Add `BOTAN_TARGET_OS_HAS_FILESYSTEM` note |
| RNG-3 missing compile-time precondition | Minor | Add `BOTAN_HAS_HMAC_DRBG && BOTAN_HAS_SHA2_64` note |
| Missing: entire Object Tests group (5 tests) | **High** | Add PKCS11-OBJECT-1 through PKCS11-OBJECT-5 |
| Missing: SESSION-9 (`test_session_info`) | Medium | Add PKCS11-SESSION-9 |
| Missing: ECDSA-8 (`test_ecdsa_curve_import`) | Medium | Add PKCS11-ECDSA-8 |
| ECDH-7 missing SoftHSM/Raw KDF note | Minor | Add note about CKD_NULL KDF limitation |
