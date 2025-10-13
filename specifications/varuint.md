# Varuint Specification
**Version: 0.0.1** \
**Status: Pre-draft** \
© 2024 Cryptid Technologies, Inc.

[![](https://img.shields.io/badge/made%20by-Cryptid%20Technologies-gold.svg?style=flat-square)][0]
[![](https://img.shields.io/badge/project-provenance-purple.svg?style=flat-square)][1]
[![](https://img.shields.io/badge/project-multiformats-blue.svg?style=flat-square)][2]

This specification is subject to the [Community Specification License 1.0][3].

## Contents
1. [Foreword](#foreword)
2. [Introduction](#introduction)
    1. [Current Status](#current-status)
    2. [Normative References](#normative-references)
    3. [Terms and Definitions](#terms-and-definitions)
3. [Specification](#specification)
    1. [Encoding Algorithm](#encoding-algorithm)
    2. [Decoding Algorithm](#decoding-algorithm)
    3. [Minimal Encoding Requirement](#minimal-encoding-requirement)
4. [Examples](#examples)
    1. [Single-Byte Values](#single-byte-values)
    2. [Multi-Byte Values](#multi-byte-values)
    3. [Sigil Encodings](#sigil-encodings)

## [Foreword](#foreword)

Attention is drawn to the possibility that some of the elements of this
document may be the subject of patent rights. No party shall not be held
responsible for identifying any or all such patent rights.

Any trade name used in this document is information given for the convenience
of users and does not constitute an endorsement.

This document was prepared by Cryptid Technologies, Inc.

Known patent licensing exclusions are available in the specification's
repository's Notices.md file.

Any feedback or questions on this document should be directed to
[specifications repository][1].

THESE MATERIALS ARE PROVIDED "AS IS." The Contributors and Licensees expressly
disclaim any warranties (express, implied, or otherwise), including implied
warranties of merchantability, non-infringement, fitness for a particular
purpose, or title, related to the materials.  The entire risk as to
implementing or otherwise using the materials is assumed by the implementer and
user. IN NO EVENT WILL THE CONTRIBUTORS OR LICENSEES BE LIABLE TO ANY OTHER
PARTY FOR LOST PROFITS OR ANY FORM OF INDIRECT, SPECIAL, INCIDENTAL, OR
CONSEQUENTIAL DAMAGES OF ANY CHARACTER FROM ANY CAUSES OF ACTION OF ANY KIND
WITH RESPECT TO THIS DELIVERABLE OR ITS GOVERNING AGREEMENT, WHETHER BASED ON
BREACH OF CONTRACT, TORT (INCLUDING NEGLIGENCE), OR OTHERWISE, AND WHETHER OR
NOT THE OTHER MEMBER HAS BEEN ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

## [Introduction](#introduction)

This specification describes the encoding of unsigned integers as variable-
length byte sequences (varuint) used throughout the multiformats ecosystem.
Varuint encoding provides space-efficient serialization for small numbers while
supporting arbitrarily large values.

### [Current Status](#current-status)

This version of the Varuint specification documents the encoding format used in
all multiformats specifications. It is derived from the [multiformats unsigned-
varint specification][4] and provides a normative reference for all provenance
specifications that use varuint encoding.

### [Normative References](#normative-references)

This specification is based on the [multiformats unsigned-varint
specification][4], which is itself based on unsigned LEB128 (Little Endian Base
128) encoding with additional restrictions for security and consistency.

This document also refers to `sigils` that identify a codec or data type. The
normative reference for the list of sigils can be found in the [multiformats
multicodecs table][5].

### [Terms and Definitions](#terms-and-definitions)

**Varuint**
: An unsigned integer encoded in a variable number of bytes using the encoding
specified in this document.

**Sigil**
: A constant value that maps to a codec or data type, encoded as a varuint.

**Continuation Bit**
: The most significant bit (MSB) of each byte in a varuint encoding. When set
to 1, it indicates that additional bytes follow. When set to 0, it indicates
the current byte is the final byte.

**LEB128**
: Little Endian Base 128, a variable-length encoding for unsigned integers.

## [Specification](#specification)

Varuint encoding serializes unsigned integers in a space-efficient manner by
encoding 7 bits of data per byte, with the 8th bit (MSB) serving as a
continuation indicator.

### [Encoding Algorithm](#encoding-algorithm)

To encode an unsigned integer as a varuint:

1. Start with the least significant 7 bits of the integer
2. If the remaining value (after removing those 7 bits) is non-zero:
   - Set the MSB (bit 7) of the current byte to 1 (continuation bit)
   - Proceed to encode the next 7 bits
3. If the remaining value is zero:
   - Set the MSB (bit 7) of the current byte to 0 (termination bit)
   - Complete the encoding

The encoding proceeds from least significant to most significant bits, with
each byte containing 7 bits of data and 1 continuation bit.

### [Decoding Algorithm](#decoding-algorithm)

To decode a varuint:

1. Read bytes sequentially
2. For each byte:
   - Extract the lower 7 bits as data
   - Check the MSB:
     - If MSB is 1, continue reading the next byte
     - If MSB is 0, this is the final byte
3. Reconstruct the integer by combining the 7-bit segments:
   - First byte contributes bits 0-6
   - Second byte contributes bits 7-13 (multiply by 128)
   - Third byte contributes bits 14-20 (multiply by 16384)
   - And so on...

### [Minimal Encoding Requirement](#minimal-encoding-requirement)

Varuint encodings MUST be minimal. This means:

1. **No leading zeros**: The encoding must use the minimum number of bytes
   required to represent the value
2. **Only zero encodes as 0x00**: The value 0 is the only value that can be
   encoded as a single byte 0x00
3. **Rejection of non-minimal encodings**: Decoders MUST reject varuints with
   unnecessary leading zero bytes. For example, encoding the value 1 as
   `0x81 0x00` is invalid and must be rejected

This requirement ensures canonical encoding and prevents potential security
vulnerabilities related to alternative encodings of the same value.

**Maximum Size**: Implementations MUST restrict varuints to a maximum of 9
bytes (63 bits) to prevent denial-of-service attacks involving arbitrarily
large encoded integers.

## [Examples](#examples)

### [Single-Byte Values](#single-byte-values)

Values from 0 to 127 can be encoded in a single byte:

```
Value 0 (0x00):
  00000000 => 0x00
  MSB=0 (final byte), data bits = 0000000

Value 1 (0x01):
  00000001 => 0x01
  MSB=0 (final byte), data bits = 0000001

Value 127 (0x7f):
  01111111 => 0x7f
  MSB=0 (final byte), data bits = 1111111
  Maximum single-byte value
```

### [Multi-Byte Values](#multi-byte-values)

Values 128 and above require multiple bytes:

```
Value 128 (0x80):
  10000000 00000001 => 0x80 0x01
  Byte 0: MSB=1 (continuation), data = 0000000 (0)
  Byte 1: MSB=0 (final), data = 0000001 (1)
  Reconstruction: 0 + (1 × 128) = 128

Value 255 (0xff):
  11111111 00000001 => 0xff 0x01
  Byte 0: MSB=1 (continuation), data = 1111111 (127)
  Byte 1: MSB=0 (final), data = 0000001 (1)
  Reconstruction: 127 + (1 × 128) = 255

Value 300 (0x012c):
  10101100 00000010 => 0xac 0x02
  Byte 0: MSB=1 (continuation), data = 0101100 (44)
  Byte 1: MSB=0 (final), data = 0000010 (2)
  Reconstruction: 44 + (2 × 128) = 300

Value 16384 (0x4000):
  10000000 10000000 00000001 => 0x80 0x80 0x01
  Byte 0: MSB=1, data = 0000000 (0)
  Byte 1: MSB=1, data = 0000000 (0)
  Byte 2: MSB=0, data = 0000001 (1)
  Reconstruction: 0 + (0 × 128) + (1 × 16384) = 16384
```

### [Sigil Encodings](#sigil-encodings)

The following table shows how common sigils used in multiformats specifications
are encoded as varuints:

```
Sigil Value    Sigil Name           Varuint Encoding
-----------    ---------------      ----------------
0xed           EdDSA                0xed
0x123a         Multikey             0xba 0x24
0x123b         Nonce                0xbb 0x24
0x1239         Multisig             0xb9 0x24
0x1207         VLAD                 0x87 0x24
0x1300         Ed25519 priv key     0x80 0x26
0x1301         secp256k1 pub key    0x81 0x26
0xd01304       Bls12381G1SigShare   0x84 0xa6 0xc0 0x06

Example: Encoding sigil 0x123b (Nonce = 4667 decimal):
  4667 = 0x123b
  Step 1: 4667 mod 128 = 59 (0x3b), 4667 ÷ 128 = 36
    Byte 0: 10111011 = 0xbb (MSB=1, data=59)
  Step 2: 36 mod 128 = 36 (0x24), 36 ÷ 128 = 0
    Byte 1: 00100100 = 0x24 (MSB=0, data=36)
  Result: 0xbb 0x24
```

[0]: https://cryptid.tech
[1]: https://github.com/cryptidtech/provenance-specifications/
[2]: https://github.com/multiformats/multiformats
[3]: https://github.com/CommunitySpecification/1.0
[4]: https://github.com/multiformats/unsigned-varint/blob/master/README.md
[5]: https://github.com/multiformats/multicodecs/blob/master/table.csv
