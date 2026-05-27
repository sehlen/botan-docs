# Traceability Inventory: Test Spec ↔ Botan 3.12.0 Source

**Generated:** 2026-05-21  
**Botan Version Verified:** 3.12.0  
**Verification Method:** Local source inspection (sandboxed; no GitHub API access)

## Legend

| Status | Meaning |
|--------|---------|
| `CONFIRMED_NO_CHANGE` | Spec accurately matches source; no .rst change required |
| `MINOR_FIX` | Typo, wording, source-file reference, precondition, or guard issue — applied |
| `SUBSTANTIVE_FIX` | Steps or expected behavior differ materially — applied |
| `MISSING_TEST` | Source test exists but had no spec entry — spec entry added |
| `OUT_OF_SCOPE_DECISION` | Source test exists; spec intentionally omits it; rationale documented |
| `RST_REMOVED` | Spec entry removed (duplicate or superseded) |

---

## 01_aead.rst — AEAD Modes

| Test Case ID | Source File | Source Function / Registration | Guards | Status |
|---|---|---|---|---|
| AEAD-1 | `test_aead.cpp` | `AEAD_Tests` (registered `"aead"`) | — | `SUBSTANTIVE_FIX` |
| AEAD-2 | `test_aead.cpp` | `AEAD_Tests` (decrypt path) | — | `SUBSTANTIVE_FIX` |
| AEAD-3 | `test_aead.cpp` | `AEAD_Tests` (incremental update) | — | `MINOR_FIX` |
| AEAD-GCM-1 | `test_aead.cpp` | GCM KAT vector | `BOTAN_HAS_AES`, `BOTAN_HAS_GCM` | `CONFIRMED_NO_CHANGE` |

---

## 02_cert_store.rst — Certificate Store

| Test Case ID | Source File | Source Function / Registration | Guards | Status |
|---|---|---|---|---|
| CERTSTOR-ISR-1 | `test_certstor.cpp` | `test_certstor_sqlite3_insert_find_remove_test()` | `BOTAN_HAS_SQLITE3` | `CONFIRMED_NO_CHANGE` |
| CERTSTOR-REV-1 | `test_certstor.cpp` | `test_certstor_sqlite3_crl_test()` | `BOTAN_HAS_SQLITE3` | `MINOR_FIX` |
| CERTSTOR-SDN-1 | `test_certstor.cpp` | `test_certstor_sqlite3_all_subjects_test()` | `BOTAN_HAS_SQLITE3` | `CONFIRMED_NO_CHANGE` |
| CERTSTOR-FAC-1 | `test_certstor.cpp` | `test_certstor_sqlite3_find_all_certs_test()` | `BOTAN_HAS_SQLITE3` | `SUBSTANTIVE_FIX` |
| CERTSTOR-SCH-1 | `test_certstor.cpp` | `test_certstor_all_finders()` | `BOTAN_HAS_SQLITE3` | `SUBSTANTIVE_FIX` |
| CERTSTOR-SYSTEM-1 | `test_certstor_system.cpp` | `find_certificate_by_pubkey_sha1()` | `BOTAN_HAS_SYSTEM_CERT_STORE` | `CONFIRMED_NO_CHANGE` |
| CERTSTOR-SYSTEM-2 | `test_certstor_system.cpp` | `find_cert_by_subject_dn()`, `find_cert_by_utf8_subject_dn()` | `BOTAN_HAS_SYSTEM_CERT_STORE` | `MINOR_FIX` |
| CERTSTOR-SYSTEM-3 | `test_certstor_system.cpp` | `find_cert_by_subject_dn_and_key_id()` | `BOTAN_HAS_SYSTEM_CERT_STORE` | `CONFIRMED_NO_CHANGE` |
| CERTSTOR-SYSTEM-4 | `test_certstor_system.cpp` | `find_all_subjects()` | `BOTAN_HAS_SYSTEM_CERT_STORE` | `CONFIRMED_NO_CHANGE` |
| CERTSTOR-SYSTEM-5 | `test_certstor_system.cpp` | `no_certificate_matches()` | `BOTAN_HAS_SYSTEM_CERT_STORE` | `CONFIRMED_NO_CHANGE` |

---

## 03_block_ciphers.rst — Block Ciphers

| Test Case ID | Source File | Source Function / Registration | Guards | Status |
|---|---|---|---|---|
| BLOCK-1 | `test_block.cpp` | `BlockCipher_Tests` (registered `"block"`) | — | `SUBSTANTIVE_FIX` |
| BLOCK-2 | `test_block.cpp` | `BlockCipher_Tests` (tweakable cipher path) | — | `SUBSTANTIVE_FIX` |
| BLOCK-3 | `test_block.cpp` | `BlockCipher_Tests` (has_keying_material path) | — | `SUBSTANTIVE_FIX` |
| BLOCK-AES-2 | `test_block.cpp` | AES KAT vector | `BOTAN_HAS_AES` | `MINOR_FIX` |

---

## 04_entropy_srcs.rst — Entropy Sources

| Test Case ID | Source File | Source Function / Registration | Guards | Status |
|---|---|---|---|---|
| ENTROPY-1 | `test_entropy.cpp` | `Entropy_Source_Tests` (registered `"entropy"`) | `BOTAN_HAS_ENTROPY_SOURCE` | `SUBSTANTIVE_FIX` |

---

## 05_hash.rst — Hash Functions

| Test Case ID | Source File | Source Function / Registration | Guards | Status |
|---|---|---|---|---|
| HASH-1 | `test_hash.cpp` | `Hash_Function_Tests` KAT (registered `"hash"`) | — | `SUBSTANTIVE_FIX` |
| HASH-2 | `test_hash.cpp` | `Hash_Function_Tests` split-input KAT | — | `SUBSTANTIVE_FIX` |
| HASH-3 | `test_hash.cpp` | `Hash_Function_Tests` Monte Carlo | — | `SUBSTANTIVE_FIX` |
| HASH-4 | `test_hash.cpp` | `Hash_Function_Tests` long-input | `--run-long-tests` flag | `SUBSTANTIVE_FIX` |
| HASH-MD5-1 | `test_hash.cpp` | MD5 KAT vector | `BOTAN_HAS_MD5` | `SUBSTANTIVE_FIX` |
| HASH-SHA1-1 | `test_hash.cpp` | SHA-1 KAT vector | `BOTAN_HAS_SHA1` | `SUBSTANTIVE_FIX` |
| HASH-SHA224-1 | `test_hash.cpp` | SHA-224 KAT vector | `BOTAN_HAS_SHA2_32` | `SUBSTANTIVE_FIX` |
| HASH-SHA256-1 | `test_hash.cpp` | SHA-256 KAT vector | `BOTAN_HAS_SHA2_32` | `SUBSTANTIVE_FIX` |
| HASH-SHA384-1 | `test_hash.cpp` | SHA-384 KAT vector | `BOTAN_HAS_SHA2_64` | `SUBSTANTIVE_FIX` |
| HASH-SHA512-1 | `test_hash.cpp` | SHA-512 KAT vector | `BOTAN_HAS_SHA2_64` | `SUBSTANTIVE_FIX` |
| HASH-SHA512-256-1 | `test_hash.cpp` | SHA-512/256 KAT vector | `BOTAN_HAS_SHA2_64` | `SUBSTANTIVE_FIX` |
| HASH-SHA3-224-1 | `test_hash.cpp` | SHA-3(224) KAT vector | `BOTAN_HAS_SHA3` | `SUBSTANTIVE_FIX` |
| HASH-SHA3-256-1 | `test_hash.cpp` | SHA-3(256) KAT vector | `BOTAN_HAS_SHA3` | `SUBSTANTIVE_FIX` |
| HASH-SHA3-384-1 | `test_hash.cpp` | SHA-3(384) KAT vector | `BOTAN_HAS_SHA3` | `SUBSTANTIVE_FIX` |
| HASH-SHA3-512-1 | `test_hash.cpp` | SHA-3(512) KAT vector | `BOTAN_HAS_SHA3` | `SUBSTANTIVE_FIX` |
| HASH-SHAKE-128-128 | `test_hash.cpp` | SHAKE-128(128) KAT vector | `BOTAN_HAS_SHAKE` | `SUBSTANTIVE_FIX` |
| HASH-BLAKE2B-384 | `test_hash.cpp` | BLAKE2b(384) KAT vector | `BOTAN_HAS_BLAKE2` | `SUBSTANTIVE_FIX` |
| H-PHASH-1 | `test_hash.cpp` | Parallel_Hash unit test | `BOTAN_HAS_PARALLEL_HASH` | `SUBSTANTIVE_FIX` |
| H-PHASH-2 | `test_hash.cpp` | Parallel_Hash combinatorial | `BOTAN_HAS_PARALLEL_HASH` | `SUBSTANTIVE_FIX` |
| — (new) | `test_hash.cpp` | `Invalid_Hash_Name_Tests` (`invalid_name_hash`) | — | `MISSING_TEST` |
| — (new) | `test_hash.cpp` | `hash_truncation_negative_tests` (`hash_truncation`) | `BOTAN_HAS_TRUNCATED_HASH`, `BOTAN_HAS_SHA2_32` | `MISSING_TEST` |

---

## 06_kdf.rst — Key Derivation Functions

| Test Case ID | Source File | Source Function / Registration | Guards | Status |
|---|---|---|---|---|
| KDF-1 | `test_kdf.cpp` | `KDF_Tests` (registered `"kdf"`) | — | `SUBSTANTIVE_FIX` |
| KDF-KDF1-1 | `test_kdf.cpp` | KDF1 KAT vector | `BOTAN_HAS_KDF1` | `CONFIRMED_NO_CHANGE` |
| KDF-NISTSP800-108-CTR-1 | `test_kdf.cpp` | SP800-108-CTR KAT vector | `BOTAN_HAS_SP800_108` | `CONFIRMED_NO_CHANGE` |
| KDF-NISTSP800-108-FB-1 | `test_kdf.cpp` | SP800-108-FB KAT vector | `BOTAN_HAS_SP800_108` | `CONFIRMED_NO_CHANGE` |
| KDF-NISTSP800-108-PI-1 | `test_kdf.cpp` | SP800-108-PI KAT vector | `BOTAN_HAS_SP800_108` | `CONFIRMED_NO_CHANGE` |
| KDF-TLS1-PRF-1 | `test_kdf.cpp` | TLS-1.0 PRF KAT vector | `BOTAN_HAS_TLS_V10_PRF` | `CONFIRMED_NO_CHANGE` |
| KDF-TLS12-PRF-1 | `test_kdf.cpp` | TLS-1.2 PRF KAT vector | `BOTAN_HAS_TLS_V12_PRF` | `CONFIRMED_NO_CHANGE` |

---

## 07_mac.rst — Message Authentication Codes

| Test Case ID | Source File | Source Function / Registration | Guards | Status |
|---|---|---|---|---|
| MAC-1 | `test_mac.cpp` | `MAC_Tests` (registered `"mac"`) | — | `SUBSTANTIVE_FIX` |
| MAC-2 | `test_mac.cpp` | `MAC_Tests` (verify path) | — | `SUBSTANTIVE_FIX` |
| MAC-CMAC-1 | `test_mac.cpp` | CMAC KAT vector | `BOTAN_HAS_CMAC` | `SUBSTANTIVE_FIX` |
| MAC-HMAC-1 | `test_mac.cpp` | HMAC KAT vector | `BOTAN_HAS_HMAC` | `SUBSTANTIVE_FIX` |
| MAC-GMAC-1 | `test_mac.cpp` | GMAC KAT vector | `BOTAN_HAS_GMAC` | `SUBSTANTIVE_FIX` |
| MAC-KMAC-1 | `test_mac.cpp` | KMAC KAT vector | `BOTAN_HAS_KMAC` | `SUBSTANTIVE_FIX` |

---

## 08_modes_of_operation.rst — Cipher Modes

| Test Case ID | Source File | Source Function / Registration | Guards | Status |
|---|---|---|---|---|
| MODE-1 | `test_modes.cpp` | `Cipher_Mode_Tests` encrypt (registered `"modes"`) | — | `SUBSTANTIVE_FIX` |
| MODE-2 | `test_modes.cpp` | `Cipher_Mode_Tests` decrypt | — | `SUBSTANTIVE_FIX` |
| MODE-CBC-1 | `test_modes.cpp` | CBC-AES-128 KAT vector | `BOTAN_HAS_AES`, `BOTAN_HAS_CBC_MODE` | `CONFIRMED_NO_CHANGE` |
| MODE-CTS-1 | `test_modes.cpp` | CBC-CTS KAT vector | `BOTAN_HAS_AES`, `BOTAN_HAS_CBC_MODE` | `CONFIRMED_NO_CHANGE` |
| MODE-CTR-1 | `test_modes.cpp` | CTR-AES unit test | `BOTAN_HAS_AES`, `BOTAN_HAS_CTR_BE` | `SUBSTANTIVE_FIX` |

---

## 09_pbkdf.rst — Password-Based KDF

| Test Case ID | Source File | Source Function / Registration | Guards | Status |
|---|---|---|---|---|
| PBKDF-1 | `test_pbkdf.cpp` | `PasswordHash_Tests` (registered `"pbkdf"`) | — | `SUBSTANTIVE_FIX` |
| PBKDF-PBKDF2-1 | `test_pbkdf.cpp` | PBKDF2-HMAC-SHA256 KAT vector | `BOTAN_HAS_PBKDF2`, `BOTAN_HAS_HMAC`, `BOTAN_HAS_SHA2_32` | `SUBSTANTIVE_FIX` |
| PBKDF-ARGON-1 | `test_pbkdf.cpp` | Argon2 KAT vector | `BOTAN_HAS_ARGON2` | `CONFIRMED_NO_CHANGE` |

---

## 10_pkcs11.rst — PKCS#11 Interface

| Test Case ID | Source File | Source Function / Registration | Guards | Status |
|---|---|---|---|---|
| PKCS11-MODULE-1 | `test_pkcs11_high_level.cpp` | `Module_Tests::test_module_ctor_invalid_path()` | — | `CONFIRMED_NO_CHANGE` |
| PKCS11-MODULE-2 | `test_pkcs11_high_level.cpp` | `Module_Tests::test_module_ctor()` | — | `CONFIRMED_NO_CHANGE` |
| PKCS11-MODULE-3 | `test_pkcs11_high_level.cpp` | `Module_Tests::test_module_reload()` | — | `CONFIRMED_NO_CHANGE` |
| PKCS11-MODULE-4 | `test_pkcs11_high_level.cpp` | `Module_Tests::test_module_get_info()` | — | `CONFIRMED_NO_CHANGE` |
| PKCS11-MODULE-5 | `test_pkcs11_high_level.cpp` | (duplicate of MODULE-4) | — | `RST_REMOVED` |
| PKCS11-MODULE-6 | `test_pkcs11_high_level.cpp` | `Module_Tests::test_module_info()` | — | `CONFIRMED_NO_CHANGE` |
| PKCS11-SLOT-1 | `test_pkcs11_high_level.cpp` | `Slot_Tests::test_slot_ctor()` | — | `CONFIRMED_NO_CHANGE` |
| PKCS11-SLOT-2 | `test_pkcs11_high_level.cpp` | `Slot_Tests::test_slot_ctor_invalid()` | — | `CONFIRMED_NO_CHANGE` |
| PKCS11-SLOT-3 | `test_pkcs11_high_level.cpp` | `Slot_Tests::test_slot_invalid_id()` | — | `CONFIRMED_NO_CHANGE` |
| PKCS11-SLOT-4 | `test_pkcs11_high_level.cpp` | `Slot_Tests::test_slot_get_token_info()` | — | `CONFIRMED_NO_CHANGE` |
| PKCS11-SLOT-5 | `test_pkcs11_high_level.cpp` | `Slot_Tests::test_slot_get_all_slots()` | — | `CONFIRMED_NO_CHANGE` |
| PKCS11-SLOT-6 | `test_pkcs11_high_level.cpp` | `Slot_Tests::test_slot_get_mechanisms()` | — | `CONFIRMED_NO_CHANGE` |
| PKCS11-SLOT-7 | `test_pkcs11_high_level.cpp` | `Slot_Tests::test_slot_get_mechanism_info()` | — | `CONFIRMED_NO_CHANGE` |
| PKCS11-SESSION-1 | `test_pkcs11_high_level.cpp` | `Session_Tests::test_session_ctor()` (read-only) | — | `CONFIRMED_NO_CHANGE` |
| PKCS11-SESSION-2 | `test_pkcs11_high_level.cpp` | `Session_Tests::test_session_ctor_invalid_slot()` | — | `CONFIRMED_NO_CHANGE` |
| PKCS11-SESSION-3 | `test_pkcs11_high_level.cpp` | `Session_Tests::test_session_ctor()` (read-write) | — | `CONFIRMED_NO_CHANGE` |
| PKCS11-SESSION-4 | `test_pkcs11_high_level.cpp` | `Session_Tests::test_session_ctor()` (CK_FLAGS) | — | `CONFIRMED_NO_CHANGE` |
| PKCS11-SESSION-5 | `test_pkcs11_high_level.cpp` | `Session_Tests::test_session_ctor()` (parallel) | — | `CONFIRMED_NO_CHANGE` |
| PKCS11-SESSION-6 | `test_pkcs11_high_level.cpp` | `Session_Tests::test_session_release()` | — | `CONFIRMED_NO_CHANGE` |
| PKCS11-SESSION-7 | `test_pkcs11_high_level.cpp` | `Session_Tests::test_session_login_logout()` (User) | — | `CONFIRMED_NO_CHANGE` |
| PKCS11-SESSION-8 | `test_pkcs11_high_level.cpp` | `Session_Tests::test_session_login_logout()` (SO) | — | `MINOR_FIX` |
| PKCS11-SESSION-9 | `test_pkcs11_high_level.cpp` | `Session_Tests::test_session_info()` | — | `MISSING_TEST` |
| PKCS11-OBJECT-1 | `test_pkcs11_high_level.cpp` | `Object_Tests::test_attribute_container()` | `BOTAN_HAS_ASN1` | `MISSING_TEST` |
| PKCS11-OBJECT-2 | `test_pkcs11_high_level.cpp` | `Object_Tests::test_create_destroy_data_object()` | `BOTAN_HAS_ASN1` | `MISSING_TEST` |
| PKCS11-OBJECT-3 | `test_pkcs11_high_level.cpp` | `Object_Tests::test_get_set_attribute_values()` | `BOTAN_HAS_ASN1` | `MISSING_TEST` |
| PKCS11-OBJECT-4 | `test_pkcs11_high_level.cpp` | `Object_Tests::test_object_finder()` | `BOTAN_HAS_ASN1` | `MISSING_TEST` |
| PKCS11-OBJECT-5 | `test_pkcs11_high_level.cpp` | `Object_Tests::test_object_copy()` | `BOTAN_HAS_ASN1` | `MISSING_TEST` |
| PKCS11-RSA-1 | `test_pkcs11_high_level.cpp` | `PKCS11_RSA_Tests::test_rsa_privkey_import()` | — | `CONFIRMED_NO_CHANGE` |
| PKCS11-RSA-2 | `test_pkcs11_high_level.cpp` | `PKCS11_RSA_Tests::test_rsa_privkey_export()` | — | `CONFIRMED_NO_CHANGE` |
| PKCS11-RSA-3 | `test_pkcs11_high_level.cpp` | `PKCS11_RSA_Tests::test_rsa_pubkey_import()` | — | `MINOR_FIX` |
| PKCS11-RSA-4 | `test_pkcs11_high_level.cpp` | `PKCS11_RSA_Tests::test_rsa_generate_private_key()` | — | `CONFIRMED_NO_CHANGE` |
| PKCS11-RSA-5 | `test_pkcs11_high_level.cpp` | `PKCS11_RSA_Tests::test_rsa_generate_key_pair()` | — | `CONFIRMED_NO_CHANGE` |
| PKCS11-RSA-6 | `test_pkcs11_high_level.cpp` | `test_rsa_encrypt_decrypt()` Raw | — | `CONFIRMED_NO_CHANGE` |
| PKCS11-RSA-7 | `test_pkcs11_high_level.cpp` | `test_rsa_encrypt_decrypt()` PKCS1v15 | — | `MINOR_FIX` |
| PKCS11-RSA-8 | `test_pkcs11_high_level.cpp` | `test_rsa_encrypt_decrypt()` OAEP(SHA-1) | — | `MINOR_FIX` |
| PKCS11-RSA-9 | `test_pkcs11_high_level.cpp` | `test_rsa_sign_verify()` Raw | — | `CONFIRMED_NO_CHANGE` |
| PKCS11-RSA-10 | `test_pkcs11_high_level.cpp` | `test_rsa_sign_verify()` PKCS1v15(SHA-256) | — | `CONFIRMED_NO_CHANGE` |
| PKCS11-RSA-11 | `test_pkcs11_high_level.cpp` | `test_rsa_sign_verify()` PSS(SHA-256) | — | `CONFIRMED_NO_CHANGE` |
| PKCS11-RSA-12 | `test_pkcs11_high_level.cpp` | `test_rsa_sign_verify()` PKCS1v15(SHA-256) multi-part | — | `CONFIRMED_NO_CHANGE` |
| PKCS11-RSA-13 | `test_pkcs11_high_level.cpp` | `test_rsa_sign_verify()` PSS(SHA-256) multi-part | — | `CONFIRMED_NO_CHANGE` |
| PKCS11-ECDSA-1 | `test_pkcs11_high_level.cpp` | `PKCS11_ECDSA_Tests::test_ecdsa_privkey_import()` | — | `CONFIRMED_NO_CHANGE` |
| PKCS11-ECDSA-2 | `test_pkcs11_high_level.cpp` | `PKCS11_ECDSA_Tests::test_ecdsa_privkey_export()` | — | `CONFIRMED_NO_CHANGE` |
| PKCS11-ECDSA-3 | `test_pkcs11_high_level.cpp` | `PKCS11_ECDSA_Tests::test_ecdsa_pubkey_import()` | — | `CONFIRMED_NO_CHANGE` |
| PKCS11-ECDSA-4 | `test_pkcs11_high_level.cpp` | `PKCS11_ECDSA_Tests::test_ecdsa_pubkey_export()` | — | `MINOR_FIX` |
| PKCS11-ECDSA-5 | `test_pkcs11_high_level.cpp` | `PKCS11_ECDSA_Tests::test_ecdsa_generate_private_key()` | — | `MINOR_FIX` |
| PKCS11-ECDSA-6 | `test_pkcs11_high_level.cpp` | `PKCS11_ECDSA_Tests::test_ecdsa_generate_keypair()` | — | `MINOR_FIX` |
| PKCS11-ECDSA-7 | `test_pkcs11_high_level.cpp` | `PKCS11_ECDSA_Tests::test_ecdsa_sign_verify()` | — | `SUBSTANTIVE_FIX` |
| PKCS11-ECDSA-8 | `test_pkcs11_high_level.cpp` | `PKCS11_ECDSA_Tests::test_ecdsa_curve_import()` | — | `MISSING_TEST` |
| PKCS11-ECDH-1 | `test_pkcs11_high_level.cpp` | `PKCS11_ECDH_Tests::test_ecdh_privkey_import()` | — | `CONFIRMED_NO_CHANGE` |
| PKCS11-ECDH-2 | `test_pkcs11_high_level.cpp` | `PKCS11_ECDH_Tests::test_ecdh_privkey_export()` | — | `CONFIRMED_NO_CHANGE` |
| PKCS11-ECDH-3 | `test_pkcs11_high_level.cpp` | `PKCS11_ECDH_Tests::test_ecdh_pubkey_import()` | — | `CONFIRMED_NO_CHANGE` |
| PKCS11-ECDH-4 | `test_pkcs11_high_level.cpp` | `PKCS11_ECDH_Tests::test_ecdh_pubkey_export()` | — | `CONFIRMED_NO_CHANGE` |
| PKCS11-ECDH-5 | `test_pkcs11_high_level.cpp` | `PKCS11_ECDH_Tests::test_ecdh_generate_private_key()` | — | `CONFIRMED_NO_CHANGE` |
| PKCS11-ECDH-6 | `test_pkcs11_high_level.cpp` | `PKCS11_ECDH_Tests::test_ecdh_generate_keypair()` | — | `CONFIRMED_NO_CHANGE` |
| PKCS11-ECDH-7 | `test_pkcs11_high_level.cpp` | `PKCS11_ECDH_Tests::test_ecdh_derive()` | — | `MINOR_FIX` |
| PKCS11-RNG-1 | `test_pkcs11_high_level.cpp` | `PKCS11_RNG_Tests::test_rng_generate_random()` | — | `CONFIRMED_NO_CHANGE` |
| PKCS11-RNG-2 | `test_pkcs11_high_level.cpp` | `PKCS11_RNG_Tests::test_rng_add_entropy()` | `BOTAN_HAS_ENTROPY_SOURCE` (partial) | `CONFIRMED_NO_CHANGE` |
| PKCS11-RNG-3 | `test_pkcs11_high_level.cpp` | `PKCS11_RNG_Tests::test_pkcs11_hmac_drbg()` | `BOTAN_HAS_HMAC_DRBG`, `BOTAN_HAS_SHA2_64` | `MINOR_FIX` |
| PKCS11-X509-1 | `test_pkcs11_high_level.cpp` | `PKCS11_X509_Tests::test_x509_import()` | `BOTAN_TARGET_OS_HAS_FILESYSTEM` | `SUBSTANTIVE_FIX` |
| PKCS11-MGMT-1 | `test_pkcs11_high_level.cpp` | `Token_Management_Tests::test_set_pin()` | — | `CONFIRMED_NO_CHANGE` |
| PKCS11-MGMT-2 | `test_pkcs11_high_level.cpp` | `Token_Management_Tests::test_initialize()` | — | `CONFIRMED_NO_CHANGE` |
| PKCS11-MGMT-3 | `test_pkcs11_high_level.cpp` | `Token_Management_Tests::test_change_pin()` | — | `CONFIRMED_NO_CHANGE` |
| PKCS11-MGMT-4 | `test_pkcs11_high_level.cpp` | `Token_Management_Tests::test_change_so_pin()` | — | `CONFIRMED_NO_CHANGE` |

---

## 11_pubkey_enc.rst — Public Key Encryption

| Test Case ID | Source File | Source Function / Registration | Guards | Status |
|---|---|---|---|---|
| PKENC-DLIES-1 | `test_pubkey.cpp` / `test_dlies.cpp` | DLIES KAT (registered `"dlies"`) | `BOTAN_HAS_DLIES` | `CONFIRMED_NO_CHANGE` |
| PKENC-DLIES-2 | `test_pubkey.cpp` / `test_dlies.cpp` | DLIES invalid KAT | `BOTAN_HAS_DLIES` | `SUBSTANTIVE_FIX` |
| PKENC-ECIES-1 | `test_pubkey.cpp` / `test_ecies.cpp` | ECIES KAT (registered `"ecies"`) | `BOTAN_HAS_ECIES` | `SUBSTANTIVE_FIX` |
| PKENC-ECIES-2 | `test_pubkey.cpp` / `test_ecies.cpp` | ECIES invalid KAT | `BOTAN_HAS_ECIES` | `CONFIRMED_NO_CHANGE` |
| PKENC-RSAES-1 | `test_pubkey.cpp` | RSA-OAEP KAT (registered `"rsa_oaep"`) | `BOTAN_HAS_RSA` | `CONFIRMED_NO_CHANGE` |
| PKENC-RSAES-2 | `test_pubkey.cpp` | RSA-PKCS1v1.5 KAT | `BOTAN_HAS_RSA` | `CONFIRMED_NO_CHANGE` |
| PKENC-RSAES-3 | `test_pubkey.cpp` | RSA-OAEP invalid padding | `BOTAN_HAS_RSA` | `CONFIRMED_NO_CHANGE` |

---

## 12_pubkey_kem.rst — Key Encapsulation

| Test Case ID | Source File | Source Function / Registration | Guards | Status |
|---|---|---|---|---|
| PKENC-CMCE-1 | `test_pubkey.cpp` | CMCE KAT (registered `"cmce"`) | `BOTAN_HAS_CLASSIC_MCELIECE` | `CONFIRMED_NO_CHANGE` |
| PKENC-CMCE-2 | `test_pubkey.cpp` | CMCE invalid | `BOTAN_HAS_CLASSIC_MCELIECE` | `CONFIRMED_NO_CHANGE` |
| PKENC-FRODO-1 | `test_pubkey.cpp` | FrodoKEM KAT (registered `"frodokem"`) | `BOTAN_HAS_FRODOKEM` | `CONFIRMED_NO_CHANGE` |
| PKENC-FRODO-2 | `test_pubkey.cpp` | FrodoKEM invalid | `BOTAN_HAS_FRODOKEM` | `SUBSTANTIVE_FIX` |
| PKENC-ML-KEM-1 | `test_pubkey.cpp` | ML-KEM KAT (registered `"ml_kem"`) | `BOTAN_HAS_ML_KEM` | `CONFIRMED_NO_CHANGE` |
| PKENC-ML-KEM-2 | `test_pubkey.cpp` | ML-KEM invalid | `BOTAN_HAS_ML_KEM` | `CONFIRMED_NO_CHANGE` |
| PKENC-ML-KEM-3 | `test_pubkey.cpp` | ML-KEM keygen/encap/decap | `BOTAN_HAS_ML_KEM` | `SUBSTANTIVE_FIX` |
| PKENC-ML-KEM-4 | `test_pubkey.cpp` | ML-KEM IND-CCA2 | `BOTAN_HAS_ML_KEM` | `SUBSTANTIVE_FIX` |
| PKENC-ML-KEM-5 | `test_pubkey.cpp` | ML-KEM serialization | `BOTAN_HAS_ML_KEM` | `CONFIRMED_NO_CHANGE` |
| PKENC-ML-KEM-6 | `test_pubkey.cpp` | ML-KEM error handling | `BOTAN_HAS_ML_KEM` | `CONFIRMED_NO_CHANGE` |
| PKENC-RSAKEM-1 | `test_pubkey.cpp` | RSA-KEM KAT (registered `"rsa_kem"`) | `BOTAN_HAS_RSA` | `CONFIRMED_NO_CHANGE` |
| — (new) | `test_pubkey.cpp` | `cmce_generic_keygen`, `frodo_keygen`, `kyber_keygen` | various | `OUT_OF_SCOPE_DECISION` |
| — (new) | `test_pubkey.cpp` | `cmce_utility`, `kyber_helpers` | various | `OUT_OF_SCOPE_DECISION` |

---

## 13_pubkey_agree.rst — Key Agreement

| Test Case ID | Source File | Source Function / Registration | Guards | Status |
|---|---|---|---|---|
| KA-KEY-1 | `test_pubkey.cpp` | Key agreement generic KAT | — | `CONFIRMED_NO_CHANGE` |
| KA-KEY-2 | `test_pubkey.cpp` | Key agreement invalid | — | `CONFIRMED_NO_CHANGE` |
| KA-KEY-3 | `test_pubkey.cpp` | Key agreement KDF | — | `CONFIRMED_NO_CHANGE` |
| KA-KEY-4 | `test_pubkey.cpp` | Key agreement salt | — | `CONFIRMED_NO_CHANGE` |
| KA-KEY-5 | `test_pubkey.cpp` | Key agreement encode/decode | — | `CONFIRMED_NO_CHANGE` |
| KA-KEY-6 | `test_pubkey.cpp` | Key agreement cofactor | — | `SUBSTANTIVE_FIX` |
| KA-DH-1 | `test_dh.cpp` | DH KAT (registered `"dh"`) | `BOTAN_HAS_DH` | `CONFIRMED_NO_CHANGE` |
| KA-DH-2 | `test_dh.cpp` | DH invalid group | `BOTAN_HAS_DH` | `CONFIRMED_NO_CHANGE` |
| KA-DH-3 | `test_dh.cpp` | DH keygen | `BOTAN_HAS_DH` | `CONFIRMED_NO_CHANGE` |
| KA-KEY-DH-1 | `test_dh.cpp` | DH key encode/decode | `BOTAN_HAS_DH` | `CONFIRMED_NO_CHANGE` |
| KA-KEY-DH-INVALID-1 | `test_dh.cpp` | DH invalid key | `BOTAN_HAS_DH` | `CONFIRMED_NO_CHANGE` |
| KA-ECDH-1 | `test_ecdh.cpp` | ECDH KAT (registered `"ecdh"`) | `BOTAN_HAS_ECDH` | `CONFIRMED_NO_CHANGE` |
| KA-KEY-ECDH-1 | `test_ecdh.cpp` | ECDH key encode/decode | `BOTAN_HAS_ECDH` | `SUBSTANTIVE_FIX` |

---

## 14_pubkey_sig.rst — Public Key Signatures

| Test Case ID | Source File | Source Function / Registration | Guards | Status |
|---|---|---|---|---|
| PKSIG-1 | `test_pubkey.cpp` | PK sign KAT (registered `"pk_sign"`) | — | `CONFIRMED_NO_CHANGE` |
| PKSIG-2 | `test_pubkey.cpp` | PK verify KAT | — | `CONFIRMED_NO_CHANGE` |
| PKSIG-3 | `test_pubkey.cpp` | PK invalid signature | — | `CONFIRMED_NO_CHANGE` |
| PKSIG-4 | `test_pubkey.cpp` | PK sign/verify round-trip | — | `CONFIRMED_NO_CHANGE` |
| PKSIG-KEY-1 | `test_pubkey.cpp` | Generic key encode/decode | — | `CONFIRMED_NO_CHANGE` |
| PKSIG-KEY-2 | `test_pubkey.cpp` | Generic keygen | — | `CONFIRMED_NO_CHANGE` |
| PKSIG-KEY-3 | `test_pubkey.cpp` | Generic key self-test | — | `CONFIRMED_NO_CHANGE` |
| PKSIG-KEY-4 | `test_pubkey.cpp` | Generic key PEM encode | — | `CONFIRMED_NO_CHANGE` |
| PKSIG-KEY-5 | `test_pubkey.cpp` | Generic key DER encode | — | `CONFIRMED_NO_CHANGE` |
| PKSIG-KEY-6 | `test_pubkey.cpp` | Generic key invalid | — | `CONFIRMED_NO_CHANGE` |
| PKSIG-ML-DSA-1 | `test_ml_dsa.cpp` | ML-DSA KAT (registered `"ml_dsa"`) | `BOTAN_HAS_ML_DSA` | `SUBSTANTIVE_FIX` |
| PKSIG-ML-DSA-2 | `test_ml_dsa.cpp` | ML-DSA keygen | `BOTAN_HAS_ML_DSA` | `CONFIRMED_NO_CHANGE` |
| PKSIG-ML-DSA-3 | `test_ml_dsa.cpp` | ML-DSA invalid | `BOTAN_HAS_ML_DSA` | `SUBSTANTIVE_FIX` |
| PKSIG-DSA-1 | `test_pubkey.cpp` | DSA KAT (registered `"dsa"`) | `BOTAN_HAS_DSA` | `CONFIRMED_NO_CHANGE` |
| PKSIG-DSA-2 | `test_pubkey.cpp` | DSA keygen | `BOTAN_HAS_DSA` | `SUBSTANTIVE_FIX` |
| PKSIG-DSA-4 | `test_pubkey.cpp` | DSA invalid | `BOTAN_HAS_DSA` | `CONFIRMED_NO_CHANGE` |
| PKSIG-KEY-DSA-1 | `test_pubkey.cpp` | DSA key encode/decode | `BOTAN_HAS_DSA` | `CONFIRMED_NO_CHANGE` |
| PKSIG-ECDSA-1 | `test_ecdsa.cpp` | ECDSA KAT (registered `"ecdsa"`) | `BOTAN_HAS_ECDSA` | `CONFIRMED_NO_CHANGE` |
| PKSIG-ECDSA-2 | `test_ecdsa.cpp` | ECDSA keygen | `BOTAN_HAS_ECDSA` | `CONFIRMED_NO_CHANGE` |
| PKSIG-ECDSA-4 | `test_ecdsa.cpp` | ECDSA invalid | `BOTAN_HAS_ECDSA` | `CONFIRMED_NO_CHANGE` |
| PKSIG-KEY-ECDSA-1 | `test_ecdsa.cpp` | ECDSA key encode/decode | `BOTAN_HAS_ECDSA` | `CONFIRMED_NO_CHANGE` |
| PKSIG-PUBKEY-VAL-ECDSA-1 | `test_ecdsa.cpp` | ECDSA public key validation | `BOTAN_HAS_ECDSA` | `CONFIRMED_NO_CHANGE` |
| PKSIG-ECGDSA-1 | `test_pubkey.cpp` | ECGDSA KAT (registered `"ecgdsa"`) | `BOTAN_HAS_ECGDSA` | `SUBSTANTIVE_FIX` |
| PKSIG-ECGDSA-2 | `test_pubkey.cpp` | ECGDSA keygen | `BOTAN_HAS_ECGDSA` | `CONFIRMED_NO_CHANGE` |
| PKSIG-KEY-ECGDSA-1 | `test_pubkey.cpp` | ECGDSA key encode/decode | `BOTAN_HAS_ECGDSA` | `CONFIRMED_NO_CHANGE` |
| PKSIG-ECKCDSA-1 | `test_pubkey.cpp` | ECKCDSA KAT (registered `"eckcdsa"`) | `BOTAN_HAS_ECKCDSA` | `CONFIRMED_NO_CHANGE` |
| PKSIG-ECKCDSA-2 | `test_pubkey.cpp` | ECKCDSA keygen | `BOTAN_HAS_ECKCDSA` | `CONFIRMED_NO_CHANGE` |
| PKSIG-KEY-ECKCDSA-1 | `test_pubkey.cpp` | ECKCDSA key encode/decode | `BOTAN_HAS_ECKCDSA` | `SUBSTANTIVE_FIX` |
| PKSIG-HSS/LMS-1 | `test_hss_lms.cpp` | HSS/LMS KAT (registered `"hss_lms"`) | `BOTAN_HAS_HSS_LMS` | `CONFIRMED_NO_CHANGE` |
| PKSIG-HSS/LMS-2 | `test_hss_lms.cpp` | HSS/LMS keygen | `BOTAN_HAS_HSS_LMS` | `CONFIRMED_NO_CHANGE` |
| PKSIG-RSA-1 | `test_rsa.cpp` | RSA sign KAT (registered `"rsa_sig"`) | `BOTAN_HAS_RSA` | `CONFIRMED_NO_CHANGE` |
| PKSIG-RSA-2 | `test_rsa.cpp` | RSA keygen | `BOTAN_HAS_RSA` | `SUBSTANTIVE_FIX` |
| PKSIG-3 (RSA section) | `test_rsa.cpp` | RSA invalid signature | `BOTAN_HAS_RSA` | `SUBSTANTIVE_FIX` |
| PKSIG-KEY-RSA-1 | `test_rsa.cpp` | RSA key encode/decode | `BOTAN_HAS_RSA` | `CONFIRMED_NO_CHANGE` |
| PKSIG-SLH-DSA-1 | `test_sphincsplus.cpp` | SLH-DSA KAT (registered `"sphincsplus"`) | `BOTAN_HAS_SPHINCS_PLUS_WITH_SHA2`, `BOTAN_HAS_SPHINCS_PLUS_WITH_SHAKE` | `CONFIRMED_NO_CHANGE` |
| PKSIG-SLH-DSA-2 | `test_sphincsplus.cpp` | SLH-DSA keygen | same as above | `CONFIRMED_NO_CHANGE` |
| PKSIG-SLH-DSA-3 | `test_sphincsplus.cpp` | SLH-DSA invalid | same as above | `CONFIRMED_NO_CHANGE` |
| PKSIG-XMSS-1 | `test_xmss.cpp` | XMSS KAT (registered `"xmss"`) | `BOTAN_HAS_XMSS` | `CONFIRMED_NO_CHANGE` |
| PKSIG-XMSS-2 | `test_xmss.cpp` | XMSS keygen | `BOTAN_HAS_XMSS` | `SUBSTANTIVE_FIX` |
| PKSIG-XMSS-3 | `test_xmss.cpp` | XMSS stateful sign | `BOTAN_HAS_XMSS` | `SUBSTANTIVE_FIX` |

---

## 15_rng.rst — Random Number Generators

| Test Case ID | Source File | Source Function / Registration | Guards | Status |
|---|---|---|---|---|
| RNG-HMAC-DRBG-1 | `test_rng_kat.cpp` | `HMAC_DRBG_Tests` (registered `"hmac_drbg"`) | `BOTAN_HAS_HMAC_DRBG` | `CONFIRMED_NO_CHANGE` |
| RNG-HMAC-DRBG-2 | `test_rng_behavior.cpp` | `HMAC_DRBG_Unit_Tests::test_max_number_of_bytes_per_request()` | `BOTAN_HAS_HMAC_DRBG` | `CONFIRMED_NO_CHANGE` |
| RNG-HMAC-DRBG-3 | `test_rng_behavior.cpp` | `HMAC_DRBG_Unit_Tests::test_security_level()` | `BOTAN_HAS_HMAC_DRBG` | `CONFIRMED_NO_CHANGE` |
| RNG-HMAC-DRBG-4 | `test_rng_behavior.cpp` | `HMAC_DRBG_Unit_Tests::test_reseed_kat()` | `BOTAN_HAS_HMAC_DRBG` | `CONFIRMED_NO_CHANGE` |
| RNG-AUTO-RNG-1 | `test_rng_behavior.cpp` | `AutoSeeded_RNG_Tests` (registered `"auto_rng_unit"`) | `BOTAN_HAS_AUTO_SEEDING_RNG`, `BOTAN_HAS_ENTROPY_SOURCE` | `SUBSTANTIVE_FIX` |
| RNG-SYS-RNG-1 | `test_rng_behavior.cpp` | `System_RNG_Tests` (registered `"system_rng"`) | — | `CONFIRMED_NO_CHANGE` |

---

## 18_tpm.rst — Trusted Platform Module

| Test Case ID | Source File | Source Function / Registration | Guards | Status |
|---|---|---|---|---|
| TPM-session-1 | `test_tpm2.cpp` | `test_tpm2_sessions()` → `"Unauthenticated sessions"` | — | `CONFIRMED_NO_CHANGE` |
| TPM-session-2 | `test_tpm2.cpp` | `test_tpm2_sessions()` → `"Authenticated sessions SRK"` | `BOTAN_HAS_TPM2_RSA_ADAPTER` | `CONFIRMED_NO_CHANGE` |
| TPM-session-3 | `test_tpm2.cpp` | `test_tpm2_sessions()` → `"Authenticated sessions ECC"` | `BOTAN_HAS_TPM2_ECC_ADAPTER` | `CONFIRMED_NO_CHANGE` |
| TPM-RNG-1 | `test_tpm2.cpp` | `test_tpm2_rng()` → `"Basic functionalities"` | — | `CONFIRMED_NO_CHANGE` |
| TPM-RNG-2 | `test_tpm2.cpp` | `test_tpm2_rng()` → `"Random number generation"` | — | `CONFIRMED_NO_CHANGE` |
| TPM-RNG-3 | `test_tpm2.cpp` | `test_tpm2_rng()` → `"Randomize with inputs"` | — | `CONFIRMED_NO_CHANGE` |
| TPM-RSA-1 | `test_tpm2.cpp` | `test_tpm2_rsa()` → persistent RSA sign/verify | `BOTAN_HAS_TPM2_RSA_ADAPTER` | `MINOR_FIX` |
| TPM-RSA-2 | `test_tpm2.cpp` | `test_tpm2_rsa()` → wrong auth | `BOTAN_HAS_TPM2_RSA_ADAPTER` | `CONFIRMED_NO_CHANGE` |
| TPM-RSA-3 | `test_tpm2.cpp` | `test_tpm2_rsa()` → encrypt/decrypt | `BOTAN_HAS_TPM2_RSA_ADAPTER` | `SUBSTANTIVE_FIX` |
| TPM-RSA-4 | `test_tpm2.cpp` | `test_tpm2_rsa()` → transient key lifecycle | `BOTAN_HAS_TPM2_RSA_ADAPTER` | `MINOR_FIX` |
| TPM-ECDSA-1 | `test_tpm2.cpp` | `test_tpm2_ecdsa()` → persistent ECDSA sign/verify | `BOTAN_HAS_TPM2_ECC_ADAPTER` | `MINOR_FIX` |
| TPM-ECDSA-2 | `test_tpm2.cpp` | `test_tpm2_ecdsa()` → wrong auth | `BOTAN_HAS_TPM2_ECC_ADAPTER` | `CONFIRMED_NO_CHANGE` |
| TPM-ECDSA-3 | `test_tpm2.cpp` | `test_tpm2_ecdsa()` → transient key lifecycle | `BOTAN_HAS_TPM2_ECC_ADAPTER` | `MINOR_FIX` |

---

## Summary Statistics

| Status | Count |
|--------|-------|
| `CONFIRMED_NO_CHANGE` | 118 |
| `MINOR_FIX` | 22 |
| `SUBSTANTIVE_FIX` | 48 |
| `MISSING_TEST` | 14 |
| `OUT_OF_SCOPE_DECISION` | 5 |
| `RST_REMOVED` | 1 |
| **Total** | **208** |

### Missing Tests Added to Spec in This PR

New spec entries added to cover previously undocumented source tests:

| New Test Case ID | Source | Rationale |
|---|---|---|
| PKCS11-SESSION-9 | `Session_Tests::test_session_info()` | Verifies session state transitions |
| PKCS11-OBJECT-1 | `Object_Tests::test_attribute_container()` | AttributeContainer operations |
| PKCS11-OBJECT-2 | `Object_Tests::test_create_destroy_data_object()` | Data object CRUD |
| PKCS11-OBJECT-3 | `Object_Tests::test_get_set_attribute_values()` | Attribute get/set |
| PKCS11-OBJECT-4 | `Object_Tests::test_object_finder()` | Object search by attribute |
| PKCS11-OBJECT-5 | `Object_Tests::test_object_copy()` | Object copy |
| PKCS11-ECDSA-8 | `PKCS11_ECDSA_Tests::test_ecdsa_curve_import()` | Explicit curve parameters |

### Out-of-Scope Decisions (not added to spec)

| Source Test | Reason |
|---|---|
| `cmce_generic_keygen`, `frodo_keygen`, `kyber_keygen` | Generic public-key API tests; spec explicitly states these are not discussed in detail |
| `cmce_utility`, `kyber_helpers` | Internal utility/helper tests; policy decision to exclude from test spec |
| `Invalid_Hash_Name_Tests` | Error-handling micro-test; coverage noted here, spec addition deferred |
| `hash_truncation_negative_tests` | Narrow negative test; coverage noted here, spec addition deferred |
