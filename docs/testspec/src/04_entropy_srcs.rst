Entropy Sources
===============

Entropy sources are tested using a system test that polls the entropy
source for entropy and checks that entropy was added to given random
number generator's entropy pool. Additionally, the entropy returned by
the entropy sources is compressed using different compression algorithms
and the compressed byte size is compared to the number of entropy bytes
returned by the entropy source. All entropy sources in the build-time
configuration variable BOTAN_ENTROPY_DEFAULT_SOURCES are tested. In the
default configuration these are "rdseed", "rdrand", "darwin_secrandom",
"dev_random", "win32_cryptoapi", "proc_walk" and "system_stats". Note
that some entropy sources are not available on all platforms and
therefore tests are skipped on unsupported platforms.

All the tests are implemented in *src/tests/test\_entropy.cpp*.

Entropy sources are tested with the following constraints:

-  Number of test cases: 1
-  Source: -

.. table::
   :class: longtable
   :widths: 20 80

   +-----------------------+--------------------------------------------------------------------------+
   | **Test Case No.:**    | ENTROPY-1                                                                |
   +-----------------------+--------------------------------------------------------------------------+
   | **Type:**             | Positive Test                                                            |
   +-----------------------+--------------------------------------------------------------------------+
   | **Description:**      | Tests whether each enabled entropy source outputs entropy bytes          |
   +-----------------------+--------------------------------------------------------------------------+
   | **Preconditions:**    | None                                                                     |
   +-----------------------+--------------------------------------------------------------------------+
   | **Input Values:**     | None                                                                     |
   +-----------------------+--------------------------------------------------------------------------+
   | **Expected Output:**  | None                                                                     |
   +-----------------------+--------------------------------------------------------------------------+
   | **Steps:**            | #. Create an Entropy_Sources object from all entropy sources in          |
   |                       |    BOTAN_ENTROPY_DEFAULT_SOURCES using Entropy_Sources::global_sources() |
   |                       |                                                                          |
   |                       | #. Get all sources supported by this platform and for each entropy       |
   |                       |    source do:                                                            |
   |                       |                                                                          |
   |                       |    a. Poll the entropy source using a SeedCapturing_RNG test object and  |
   |                       |       check that the number of entropy bytes added to the                |
   |                       |       SeedCapturing_RNG pool is greater or equal to the entropy estimate |
   |                       |       returned by the entropy source                                     |
   |                       |                                                                          |
   |                       |    b. If ``rng.samples() > 0``, check that it added at least one byte   |
   |                       |       and check that the sample count satisfies ``rng.samples() >= 1``  |
   |                       |                                                                          |
   |                       |    c. If ``BOTAN_HAS_COMPRESSION`` is defined and the entropy source     |
   |                       |       produced data, for each of zlib and lzma:                          |
   |                       |                                                                          |
   |                       |       i.  Compress the seed material at compression level 9 and verify   |
   |                       |           that the compressed size * 8 is greater than or equal to the   |
   |                       |           entropy estimate reported by the source                        |
   |                       |                                                                          |
   |                       |       ii. Poll the entropy source a second time, concatenate both seed   |
   |                       |           materials, compress together, and verify that the combined     |
   |                       |           compressed size is strictly larger than the single-poll        |
   |                       |           compressed size, and that the differential compressed size * 8 |
   |                       |           is greater than or equal to the second poll's entropy estimate |
   |                       |                                                                          |
   |                       |       Note: bzip2 is intentionally excluded due to a known macOS issue   |
   |                       |       (GitHub #394) and block-size effects on the differential test      |
   +-----------------------+--------------------------------------------------------------------------+
