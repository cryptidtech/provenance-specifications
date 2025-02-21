# Nonce Specification
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
4. [Examples](#examples)
    1. [A Nonce with Random Data](#a-nonce-with-random-data)
    2. [A Nonce with Multisig Data](#a-nonce-with-multisig-data)

## [Foreword](#foreword)

Attention is drawn to the possibility that some of the elements of this
document may be the subject of patent rights. No party shall not be held
responsible for identifying any or all such patent rights.

Any trade name used in this document is information given for the convenience
of users and does not constitute an endorsement.

This document was prepared by Cryptid Technologies, Inc.

Known patent licensing exclusions are available in the specification’s
repository’s Notices.md file.

Any feedback or questions on this document should be directed to
[specifications repository][1].

THESE MATERIALS ARE PROVIDED “AS IS.” The Contributors and Licensees expressly
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

This specification outlines how to encode a nonce value (i.e. Number used ONCE)
in a multiformats compatible way. Nonces are commonly used in a variety of 
cryptographic operations.

### [Current Status](#current-status)

This version of the Nonce specification defines a minimal multiformats
compatible encoding. It meets the needs of all use cases so far, however feel 
free to file an issue if you have a use case this does not cover.

### [Normative References](#normative-references)

This document refers to `varuint` encoded values throughout. The normative 
reference for which can be found in the [multiformats unsigned-varint
specification][4].

This document also refers to `sigils` the identify a codec or data type. The 
normative reference for the list of sigils can be found in the [multiformats 
multicodecs table][5].

### [Terms and Definitions](#terms-and-definitions)

**Varuint**
: An unsigned integer variable encoded in a variable number bytes.

**Sigil**
: A constant value that maps to a codec or data type.

## [Specification](#specification)

The following diagram shows the overall structure of a Nonce. A Nonce is
identified by the `0x123b` sigil followed by a `varbytes` encoded array of
octets. The sigil is encoded as a varuint `0xbb24`.

```
nonce sigil
 |
 v
0xbb24 <varbytes>
         ^
         |
    nonce octets

<varbytes> ::= <varuint> N(OCTET)
                   ^        ^
                  /          \
          count of            variable number
            octets            of octets
```
The Nonce format is designed in such a way that tools only need support for 
`varuint` and `varbytes` processing to know exactly how many octets are in the 
Nonce object so that it can skip over it if needed.

## [Examples](#examples)

### [A Nonce with Random Data](#a-nonce-with-random-data)

This example shows how a Nonce containing random data is encoded:

```
bb 24               -- varuint, nonce sigil
  20                -- varuint, length of nonce data
    [32 octets]     -- 32 octets of random data
```

### [A Nonce with Multisig Data](#a-nonce-with-multisig-data)

This example shows how a Nonce containing an EdDSA [Multisig][6]—as used in some
[VLADs][7]—is encoded:

```
bb 24               -- varuint, nonce sigil (0x123b)
  49                -- varuint, length of nonce data
    [               -- 73 octets of the Multisig
      b9 24         -- varuint, Multisig sigil (0x1239)
      ed            -- varuint, EdDSA sigil (0xed)
      [70 octets]   -- 68 octets of Multisig data
    ]
```

In this example the data inside of the Nonce is a Multisig. The Multisig 
consists of 72 octets that begins with the `0xb924` Multisig sigil followed by
the `0xed` Ed25519 codec sigil signifying that this is an EdDSA signature. The 
remaining 70 octets are the Multisig attributes of the signature.

[0]: https://cryptid.tech 
[1]: https://github.com/cryptidtech/provenance-specifications/
[2]: https://github.com/multiformats/multiformats
[3]: https://github.com/CommunitySpecification/1.0
[4]: https://github.com/multiformats/unsigned-varint/blob/master/README.md
[5]: https://github.com/multiformats/multicodecs/blob/master/table.csv
[6]: https://github.com/cryptidtech/provenance-specifications/blob/main/specifications/multisig.md
[7]: https://github.com/cryptidtech/provenance-specifications/blob/main/specifications/vlad.md
