# Review: Certificate Store Tests

**Botan Version:** 3.12.0
**Date:** 2026-05-20
**File Reviewed:** 02_cert_store.rst
**Source Files:** `src/tests/test_certstor.cpp`, `src/tests/test_certstor_system.cpp`

---


### CERTSTOR-ISR-1
**Status:** ✅ CONFIRMED
**Source:** `test_certstor_sqlite3_insert_find_remove_test()` in `test_certstor.cpp`
**Notes:** All eight steps in the spec are faithfully reflected in the implementation:
find by subject DN (`find_cert` without key ID), find by subject DN + key ID, look up private key by
cert, look up certs for key, remove cert, verify removal, remove key, verify key removal.
The test is run against all 6 test certificate/key pairs.

---

### CERTSTOR-REV-1
**Status:** 🔄 PROPOSED CHANGES
**Source:** `test_certstor_sqlite3_crl_test()` in `test_certstor.cpp`
**Notes:** The spec only mentions revoking Certs[3] once. The actual code calls
`store.revoke_cert(certsandkeys[3].certificate(), ...)` **twice** before the affirmation step.
This double-revocation is deliberate (idempotency check). The spec should document this.
**Proposed Changes:**
- In the Steps table, change the second occurrence of step 2 to:
  *"Revoke Certs[3] with reason CA Compromise a second time (idempotency check)"*

---

### CERTSTOR-SDN-1
**Status:** ✅ CONFIRMED
**Source:** `test_certstor_sqlite3_all_subjects_test()` in `test_certstor.cpp`
**Notes:** Spec accurately describes: insert all 6 certs, call `all_subjects()`, verify the returned
list has exactly 6 entries, and cross-check each returned DN against the known set.

---

### CERTSTOR-FAC-1
**Status:** 🔄 PROPOSED CHANGES
**Source:** `test_certstor_sqlite3_find_all_certs_test()` in `test_certstor.cpp`
**Notes:** The spec only documents the single-match lookup per certificate. The actual test also
inserts two certificates with **identical subject DNs** (from BSI test corpus:
`x509/bsi/common_14/common_14_sub_ca.ca.pem.crt` and `common_14_wrong_sub_ca.ca.pem.crt`) and
verifies that `find_all_certs(dn, {})` returns exactly 2 entries. This duplicate-DN branch is
unspecified.
**Proposed Changes:**
- Add a second scenario to the Steps:
  *"Insert two certificates sharing the same subject DN. Query by that DN with empty key ID.
  Check that exactly two certificates are returned."*

---

### CERTSTOR-SCH-1
**Status:** 🔄 PROPOSED CHANGES
**Source:** `test_certstor_all_finders()` in `test_certstor.cpp`
**Notes:** The spec describes only the SHA-256-hashed subject DN lookup
(`find_cert_by_raw_subject_dn_sha256`). The same function also tests
`find_cert_by_issuer_dn_and_serial_number`, which is not mentioned in the spec. Additionally, the
function tests the negative case: looking up with a 32-byte all-zero dummy hash returns no result.
This test uses `Certificate_Store_In_Memory`, not SQLite, so it covers the in-memory backend.
**Proposed Changes:**
- Expand Steps to include:
  1. *(existing)* Find each cert by SHA-256 hash of its raw subject DN.
  2. *(new)* Find each cert by its issuer DN and serial number.
  3. *(new)* Confirm that a lookup with a known-invalid 32-byte dummy hash returns no result.
- Clarify the Description to note that the store under test is `Certificate_Store_In_Memory`.

---

### CERTSTOR-SYSTEM-1
**Status:** ✅ CONFIRMED
**Source:** `find_certificate_by_pubkey_sha1()` and
`find_certificate_by_pubkey_sha1_with_unmatching_key_id()` in `test_certstor_system.cpp`
**Notes:** The spec correctly documents both sub-cases:
(a) the "typical" root certificate where the Subject Key Identifier equals the SHA-1 of the public
key, and (b) "SecureTrust CA" whose Subject Key Identifier deliberately differs from the SHA-1 of
the public key (regression test for GH #2779). Both functions are invoked from `Certstor_System_Tests::run()`.

---

### CERTSTOR-SYSTEM-2
**Status:** 🔄 PROPOSED CHANGES
**Source:** `find_cert_by_subject_dn()`, `find_cert_by_utf8_subject_dn()`, and
`find_all_certs_by_subject_dn()` in `test_certstor_system.cpp`
**Notes:** **The Steps table contains a copy-paste error:** Step 1 reads *"Query certificates by
their public key's SHA-1"*, which is the description for CERTSTOR-SYSTEM-1, not SYSTEM-2.
The actual test functions query by Subject Distinguished Name (PrintableString encoding and
UTF-8 encoding). The "find all certs by subject DN" variant also verifies that no duplicate
certificates are returned. The D-TRUST certificate note states it is disabled on Windows CI, which
is correctly captured in the Preconditions.
**Proposed Changes:**
- Fix Step 1: change *"Query certificates by their public key's SHA-1"* to
  *"Query certificates by their Subject Distinguished Name"*.
- Add to Steps: confirm that no duplicate certificates are returned (tested via sort+unique check
  in `find_all_certs_by_subject_dn`).

---

### CERTSTOR-SYSTEM-3
**Status:** ✅ CONFIRMED
**Source:** `find_cert_by_subject_dn_and_key_id()` and `find_certs_by_subject_dn_and_key_id()` in
`test_certstor_system.cpp`
**Notes:** Both the singular (`find_cert`) and plural (`find_all_certs`) variants of the DN+key ID
lookup are exercised. The plural variant additionally calls `certstore.contains()` to confirm the
returned certificate is recognised by the store. The spec captures the intent accurately.

---

### CERTSTOR-SYSTEM-4
**Status:** ✅ CONFIRMED
**Source:** `find_all_subjects()` in `test_certstor_system.cpp`
**Notes:** Spec accurately describes: call `all_subjects()`, confirm the list is non-empty, and
check that the DN of "ISRG Root X1" is present in the result.

---

### CERTSTOR-SYSTEM-5
**Status:** ✅ CONFIRMED
**Source:** `no_certificate_matches()` in `test_certstor_system.cpp`
**Notes:** Three distinct queries are issued with dummy data
(`find_all_certs`, `find_cert`, `find_cert_by_pubkey_sha1`) and all must return empty/null.
The spec's two sub-queries ((a) DN + key ID and (b) SHA-1 of public key) correspond exactly to
the three calls in the code. Minor: the code also passes a dummy key ID as the SHA-1 hash, which
is technically the same 28-byte value used for the DN query – this is an implementation detail
that does not affect spec accuracy.

---

---

## New Tests Found in Botan 3.12.0 Not in Spec

| Function | Registration | First Added |
|---|---|---|
| `test_certstor_load_allcert()` | `"certstor"` (via `Certstor_Tests::run()`) | Feb 2018 (commit 0e47fb6) |

### Timeline Context

**test_certstor_load_allcert** — Added in February 2018 (commit 0e47fb6), well before the 3.7.1 baseline. This test verifies that `Certificate_Store_In_Memory` correctly handles bundled PEM files (multiple certificates concatenated in a single file). The test loads a directory containing a two-certificate bundle and confirms that the store loads both certificates, while `X509_Certificate` only loads the first (expected single-certificate behavior). This test predates the 3.7.1 baseline and was missed during previous documentation.

**Conclusion:** This certificate store test was missed during previous documentation cycles. It existed before the 3.7.1 baseline and should have been documented earlier.
