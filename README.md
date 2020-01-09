# Intel(R) Multi-Buffer Crypto for IPsec Library

Contents
========
1. Overview
2. Processor Extensions
3. Recommendations
4. Package Content
5. Compilation
6. Security Considerations & Options for Increased Security
7. Installation
8. Disclaimer (ZUC, KASUMI, SNOW3G)
9. Legal Disclaimer

1\. Overview
============
Intel Multi-Buffer Crypto for IPsec Library is highly-optimized software
implementations of the core cryptographic processing for IPsec,
which provides industry-leading performance on a range of Intel(R) Processors.

For information on how to build and use this library, see the
Intel White Paper:
"Fast Multi-buffer IPsec Implementations on Intel Architecture Processors".
Jim Guilford, Sean Gulley, et. al.

The easiest way to find it is to search the Internet for the title and
Intel White Paper.

Table 1. List of supported cipher algorithms and their implementations.
```
+---------------------------------------------------------------------+
|               |                   Implementation                    |
| Encryption    +-----------------------------------------------------|
|               | x86_64 | SSE    | AVX    | AVX2   | AVX512 | VAES(5)|
|---------------+--------+--------+--------+--------+--------+--------|
| AES128-GCM    | N      | Y  by8 | Y  by8 | Y  by8 | Y  by8 | Y by48 |
| AES192-GCM    | N      | Y  by8 | Y  by8 | Y  by8 | Y  by8 | Y by48 |
| AES256-GCM    | N      | Y  by8 | Y  by8 | Y  by8 | Y  by8 | Y by48 |
| AES128-CCM    | N      | Y  by4 | Y  by8 | N      | N      | N      |
| AES128-CBC    | N      | Y(1)   | Y(3)   | N      | N      | Y(6)   |
| AES192-CBC    | N      | Y(1)   | Y(3)   | N      | N      | Y(6)   |
| AES256-CBC    | N      | Y(1)   | Y(3)   | N      | N      | Y(6)   |
| AES128-CTR    | N      | Y  by4 | Y  by8 | N      | N      | Y by16 |
| AES192-CTR    | N      | Y  by4 | Y  by8 | N      | N      | Y by16 |
| AES256-CTR    | N      | Y  by4 | Y  by8 | N      | N      | Y by16 |
| AES128-ECB    | N      | Y  by4 | Y  by4 | N      | N      | N      |
| AES192-ECB    | N      | Y  by4 | Y  by4 | N      | N      | N      |
| AES256-ECB    | N      | Y  by4 | Y  by4 | N      | N      | N      |
| NULL          | Y      | N      | N      | N      | N      | N      |
| AES128-DOCSIS | N      | Y(2)   | Y(4)   | N      | Y(7)   | N      |
| DES-DOCSIS    | Y      | N      | N      | N      | Y  x16 | N      |
| 3DES          | Y      | N      | N      | N      | Y  x16 | N      |
| DES           | Y      | N      | N      | N      | Y  x16 | N      |
| KASUMI-F8     | Y      | N      | N      | N      | N      | N      |
| ZUC-EEA3      | N      | Y  x4  | Y  x4  | N      | N      | N      |
| SNOW3G-UEA2   | N      | Y      | Y      | Y      | N      | N      |
+---------------------------------------------------------------------+
```
Notes:  
(1,2) - decryption is by4 and encryption is x4  
(3,4) - decryption is by8 and encryption is x8  
(5)   - AVX512 plus VAES and VPCLMULQDQ extensions  
(6)   - decryption is by16 and encryption is x16  
(7)   - same as AES128-CBC for AVX, combines cipher and CRC32  

Legend:  
` byY` - single buffer Y blocks at a time  
`  xY` - Y buffers at a time

As an example of how to read table 1 and 2, if one uses AVX512 interface
to perform AES128-CBC encryption then there is no native AVX512
implementation for this cipher. In such case, the library uses best
available implementation which is AVX for AES128-CBC.


Table 2. List of supported integrity algorithms and their implementations.
```
+-------------------------------------------------------------------------+
|                   |                   Implementation                    |
| Integrity         +-----------------------------------------------------|
|                   | x86_64 | SSE    | AVX    | AVX2   | AVX512 | VAES(3)|
|-------------------+--------+--------+--------+--------+--------+--------|
| AES-XCBC-96       | N      | Y   x4 | Y   x8 | N      | N      | N      |
| HMAC-MD5-96       | Y(1)   | Y x4x2 | Y x4x2 | Y x8x2 | N      | N      |
| HMAC-SHA1-96      | N      | Y(2)x4 | Y   x4 | Y   x8 | Y  x16 | N      |
| HMAC-SHA2-224_112 | N      | Y(2)x4 | Y   x4 | Y   x8 | Y  x16 | N      |
| HMAC-SHA2-256_128 | N      | Y(2)x4 | Y   x4 | Y   x8 | Y  x16 | N      |
| HMAC-SHA2-384_192 | N      | Y   x2 | Y   x2 | Y   x4 | Y   x8 | N      |
| HMAC-SHA2-512_256 | N      | Y   x2 | Y   x2 | Y   x4 | Y   x8 | N      |
| AES128-GMAC       | N      | Y  by8 | Y  by8 | Y  by8 | Y  by8 | Y by48 |
| AES192-GMAC       | N      | Y  by8 | Y  by8 | Y  by8 | Y  by8 | Y by48 |
| AES256-GMAC       | N      | Y  by8 | Y  by8 | Y  by8 | Y  by8 | Y by48 |
| NULL              | N      | N      | N      | N      | N      | N      |
| AES128-CCM        | N      | Y   x4 | Y   x8 | N      | N      | N      |
| AES128-CMAC-96    | Y      | Y   x4 | Y   x8 | N      | N      | N      |
| KASUMI-F9         | Y      | N      | N      | N      | N      | N      |
| ZUC-EIA3          | N      | Y  x4  | Y  x4  | N      | N      | N      |
| SNOW3G-UIA2       | N      | Y      | Y      | Y      | N      | N      |
| DOCSIS-CRC32(4)   | N      | Y      | Y      | N      | Y      | N      |
+-------------------------------------------------------------------------+
```
Notes:  
(1) - MD5 over one block implemented in C  
(2) - Implementation using SHANI extentions is x2  
(3) - AVX512 plus VAES and VPCLMULQDQ extensions  
(4) - used only with AES128-DOCSIS cipher

Legend:  
` byY`- single buffer Y blocks at a time  
`  xY`- Y buffers at a time  

Table 3. Encryption and integrity algorithm combinations
```
+---------------------------------------------------------------------+
| Encryption    | Allowed Integrity Algorithms                        |
|---------------+-----------------------------------------------------|
| AES128-GCM    | AES128-GMAC                                         |
|---------------+-----------------------------------------------------|
| AES192-GCM    | AES192-GMAC                                         |
|---------------+-----------------------------------------------------|
| AES256-GCM    | AES256-GMAC                                         |
|---------------+-----------------------------------------------------|
| AES128-CCM    | AES128-CCM                                          |
|---------------+-----------------------------------------------------|
| AES128-CBC,   | AES-XCBC-96,                                        |
| AES192-CBC,   | HMAC-SHA1-96, HMAC-SHA2-224_112, HMAC-SHA2-256_128, |
| AES256-CBC,   | HMAC-SHA2-384_192, HMAC-SHA2-512_256,               |
| AES128-CTR,   | AES128-CMAC-96,                                     |
| AES192-CTR,   | NULL                                                |
| AES256-CTR,   |                                                     |
| AES128-ECB,   |                                                     |
| AES192-ECB,   |                                                     |
| AES256-ECB,   |                                                     |
| NULL,         |                                                     |
| AES128-DOCSIS,|                                                     |
| DES-DOCSIS,   |                                                     |
| 3DES,         |                                                     |
| DES,          |                                                     |
|---------------+-----------------------------------------------------|
| AES128-DOCSIS | DOCSIS-CRC32                                        |
|---------------+-----------------------------------------------------|
| KASUMI-F8     | KASUMI-F9                                           |
|---------------+-----------------------------------------------------|
| ZUC-EEA3      | ZUC-EIA3                                            |
|---------------+-----------------------------------------------------|
| SNOW3G-UEA3   | SNOW3G-UIA3                                         |
+---------------+-----------------------------------------------------+
```

2\. Processor Extensions
========================

Table 4. Processor extensions used in the library
```
+-------------------------------------------------------------------------+
| Algorithm         | Interface | Extensions                              |
|-------------------+-----------+-----------------------------------------|
| HMAC-SHA1-96,     | AVX512    | AVX512F, AVX512BW, AVX512VL             |
| HMAC-SHA2-224_112,|           |                                         |
| HMAC-SHA2-256_128,|           |                                         |
| HMAC-SHA2-384_192,|           |                                         |
| HMAC-SHA2-512_256 |           |                                         |
|-------------------+-----------+-----------------------------------------|
| DES, 3DES,        | AVX512    | AVX512F, AVX512BW                       |
| DOCSIS-DES        |           |                                         |
|-------------------+-----------+-----------------------------------------|
| HMAC-SHA1-96,     | SSE       | SHANI                                   |
| HMAC-SHA2-224_112,|           | - presence is autodetected and library  |
| HMAC-SHA2-256_128,|           |   falls back to SSE implementation      |
| HMAC-SHA2-384_192,|           |   if not present                        |
| HMAC-SHA2-512_256 |           |                                         |
+-------------------+-----------+-----------------------------------------+
```

3\. Recommendations
===================

Legacy or to be avoided algorithms listed in the table below are implemented
in the library in order to support legacy applications. Please use corresponding
alternative algorithms instead.
```
+-------------------------------------------------------------+
| # | Algorithm          | Recommendation | Alternative       |
|---+--------------------+----------------+-------------------|
| 1 | DES encryption     | Avoid          | AES encryption    |
|---+--------------------+----------------+-------------------|
| 2 | 3DES encryption    | Avoid          | AES encryption    |
|---+--------------------+----------------+-------------------|
| 3 | HMAC-MD5 integrity | Legacy         | HMAC-SHA1         |
|---+--------------------+----------------+-------------------|
| 3 | AES-ECB encryption | Avoid          | AES-CBC, AES-CNTR |
+-------------------------------------------------------------+
```
Intel(R) Multi-Buffer Crypto for IPsec Library depends on C library and
it is recommended to use its latest version.

Applications using the Intel(R) Multi-Buffer Crypto for IPsec Library rely on
Operating System to provide process isolation.
As the result, it is recommended to use latest Operating System patches and
security updates.

4\. Package Content
===================

- LibTestApp - Library test applications
- LibPerfApp - Library performance application
- sse - Intel(R) SSE optimized routines
- avx - Intel(R) AVX optimized routines
- avx2 - Intel(R) AVX2 optimized routines
- avx512 - Intel(R) AVX512 optimized routines
- no-aesni - Non-AESNI accelerated routines

**Note:**   
There is just one branch used in the project. All development is done on the master branch.  
Code taken from the tip of the master branch should not be considered fit for production.  

Refer to the releases tab for stable code versions:  
https://github.com/intel/intel-ipsec-mb/releases


5\. Compilation
===============

Linux (64-bit only)
-------------------

Required tools:  
- GNU make  
- NASM version 2.13.03 (or newer)  
- gcc (GCC) 4.8.3 (or newer)  

Shared library:  
`> make`

Static library:  
`> make SHARED=n`

Clean the build:  
`> make clean`  
or  
`> make clean SHARED=n`

Build with debugging information:  
`> make DEBUG=y`

**Note:** Building with debugging information is not advised for production use.

For more build options and their explanation run:   
`> make help`

Windows (x64 only)
------------------

Required tools:  
- Microsoft (R) Visual Studio 2015:  
  - NMAKE: Microsoft (R) Program Maintenance Utility Version 14.00.24210.0  
  - CL: Microsoft (R) C/C++ Optimizing Compiler Version 19.00.24215.1 for x64  
  - LIB: Microsoft (R) Library Manager Version 14.00.24215.1  
  - LINK: Microsoft (R) Incremental Linker Version 14.00.24215.1  
  - Note: Building on later versions should work but is not verified  
- NASM version 2.13.03 (or newer)  

Shared library (DLL):  
`> nmake /f win_x64.mak`

Static library:  
`> nmake /f win_x64.mak SHARED=n`

Clean the build:   
`> nmake /f win_x64.mak clean`   
or   
`> nmake /f win_x64.mak clean SHARED=n`

Build with additional safety features:  
- SAFE_DATA clears sensitive information stored in stack/registers  
- SAFE_PARAM adds extra checks on input parameters  
- SAFE_LOOKUP uses constant-time lookups (enabled by default)  

`> nmake /f win_x64.mak SAFE_DATA=y SAFE_PARAM=y`

Build with debugging information:   
`> nmake /f win_x64.mak DEBUG=y`

**Note:** Building with debugging information is not advised for production use.

For more build options and their explanation run:   
`> nmake /f win_x64.mak help`

FreeBSD (64-bit only)
-------------------

Required tools:  
- GNU make  
- NASM version 2.13.03 (or newer)  
- gcc (GCC) 4.8.3 (or newer) / clang 5.0 (or newer)  

Shared library:   
`> gmake`

Static library:   
`> gmake SHARED=n`

Clean the build:   
`> gmake clean`   
or   
`> gmake clean SHARED=n`

Build with debugging information:   
`> gmake DEBUG=y`

**Note:** Building with debugging information is not advised for production use.

For more build options and their explanation run:   
`> gmake help`

6\. Security Considerations & Options for Increased Security
============================================================

### Security Considerations
The security of a system that uses cryptography depends on the strength of
the cryptographic algorithms as well as the strength of the keys.
Cryptographic key strength is dependent on several factors, with some of the
most important factors including the length of the key, the entropy of the key
bits, and maintaining the secrecy of the key.

The selection of an appropriate algorithm and mode of operation critically
affects the security of a system. Appropriate selection criteria is beyond the
scope of this document and should be determined based upon usage, appropriate
standards and consultation with a cryptographic expert. This library includes some
algorithms, which are considered cryptographically weak and are included only
for legacy and interoperability reasons. See the "Recommendations" section for
more details.

Secure creation of key material is not a part of this library. This library
assumes that cryptographic keys have been created using approved methods with
an appropriate and secure entropy source. Users of this library are
referred to NIST SP800-133 Revision 1, Recommendation for Cryptographic Key
Generation, found at https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-133r1.pdf

Even with the use of strong cryptographic algorithms and robustly generated
keys, software implementations of cryptographic algorithms may be attacked
at the implementation through cache-timing attacks, buffer-over-reads, and
other software vulnerabilities. Counter-measures against these types of
attacks are possible but require additional processing cycles. Whether a
particular system should provide such counter-measures depends on the threats
to that system, and cannot be determined by a general library such as this
one. In order to provide the most flexible implementation, this library allows
certain counter-measures to be enabled or disabled at compile time. These
options are listed below as the "Options for Increased Security" and are
enabled through various build flags.

### Options for Increased Security
There are three build options that can be enabled to increase safety in the
code and help protect external functions from incorrect input data.
SAFE_DATA and SAFE_PARAM options are disabled by default, due to
the potential performance impact associated to the extra code added.
SAFE_LOOKUP option is enabled by default, and can be disabled
by setting the parameter equal to "n" (e.g. make SAFE_LOOKUP=n).

These options (explained below) can be enabled when building the library,
by setting the parameter equal to "y" (e.g. make SAFE_DATA=y).
No specific code has been added, and no specific validation or security
tests have been performed to help protect against or check for side-channel
attacks.

### SAFE_DATA
Stack and registers containing sensitive information, such as keys or IVs, are
cleared upon completion of a function call.

### SAFE_PARAM
Input parameters are checked, looking generally for NULL pointers or an 
incorrect input length.

### SAFE_LOOKUP
Lookups which depend on sensitive information are implemented with constant
time functions.

Algorithms where these constant time functions are used are the following:  
- AESNI emulation  
- DES: SSE, AVX and AVX2 implementations  
- KASUMI: all architectures  
- SNOW3G: all architectures  
- ZUC: all architectures  

If SAFE_LOOKUP is not enabled in the build (e.g. make SAFE_LOOKUP=n) then the
algorithms listed above may be susceptible to timing attacks which could expose
the cryptographic key.

7\. Installation
================

Linux (64-bit only)
-------------------

First compile the library and then install:   
`> make`  
`> sudo make install`

To uninstall the library run:   
`> sudo make uninstall`

If you want to change install location then define PREFIX:   
`> sudo make install PREFIX=<path>`

If there is no need to run ldconfig at install stage please use NOLDCONFIG=y option:   
`> sudo make install NOLDCONFIG=y`

If library was compiled as an archive (not a default option) then install it using SHARED=n option:   
`> sudo make install SHARED=n`

Windows (x64 only)
------------------

First compile the library and then install from a command prompt in administrator mode:   
`> nmake /f win_x64.mak`  
`> nmake /f win_x64.mak install`

To uninstall the library run:   
`> nmake /f win_x64.mak uninstall`

If you want to change install location then define PREFIX (default C:\Program Files):   
`> nmake /f win_x64.mak install PREFIX=<path>`

If library was compiled as a static library (not a default option) then install it using SHARED=n option:   
`> nmake /f win_x64.mak install SHARED=n`

FreeBSD (64-bit only)
-------------------

First compile the library and then install:   
`> gmake`  
`> sudo gmake install`

To uninstall the library run:   
`> sudo gmake uninstall`

If you want to change install location then define PREFIX:   
`> sudo gmake install PREFIX=<path>`

If there is no need to run ldconfig at install stage please use NOLDCONFIG=y option:   
`> sudo gmake install NOLDCONFIG=y`

If library was compiled as an archive (not a default option) then install it using SHARED=n option:   
`> sudo gmake install SHARED=n`

8\. Disclaimer (ZUC, KASUMI, SNOW3G)
====================================

Please note that cryptographic material, such as ciphering algorithms, may be
subject to national regulations. What is more, use of some algorithms in
real networks and production equipment can be subject to agreement or
licensing by the GSMA and/or the ETSI.

For more details please see:  
- GSMA https://www.gsma.com/security/security-algorithms/  
- ETSI https://www.etsi.org/security-algorithms-and-codes/cellular-algorithm-licences

9\. Legal Disclaimer
====================

THIS SOFTWARE IS PROVIDED BY INTEL"AS IS". NO LICENSE, EXPRESS OR   
IMPLIED, BY ESTOPPEL OR OTHERWISE, TO ANY INTELLECTUAL PROPERTY RIGHTS   
ARE GRANTED THROUGH USE. EXCEPT AS PROVIDED IN INTEL'S TERMS AND   
CONDITIONS OF SALE, INTEL ASSUMES NO LIABILITY WHATSOEVER AND INTEL  
DISCLAIMS ANY EXPRESS OR IMPLIED WARRANTY, RELATING TO SALE AND/OR   
USE OF INTEL PRODUCTS INCLUDING LIABILITY OR WARRANTIES RELATING TO   
FITNESS FOR A PARTICULAR PURPOSE, MERCHANTABILITY, OR INFRINGEMENT   
OF ANY PATENT, COPYRIGHT OR OTHER INTELLECTUAL PROPERTY RIGHT.  
