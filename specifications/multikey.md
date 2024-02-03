# Multikey Specification
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
    1. [Key Comment](#key-comment)
    2. [Attributes](#attributes)
    3. [Secret Keys](#secret-keys)
4. [Examples](#examples)
    1. [An Encrypted EdDSA Secret Key](#an-encrypted-eddsa-secret-key)
    2. [An Es256K Public Key](#an-es256k-public-key)

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

This specification outlines how to encode arbitrary cryptographic key material 
in a codec-agnostic way while also supporting codec-specific attribute read-
write and codec-specific cryptographic operations (e.g. encrypt, decrypt, key 
generation, key splitting, digital signature signing and verification). The 
serialization format is suitable for network messages as well as storage in 
block devices.

### [Current Status](#current-status)

This version of the Multikey specification defines attributes to support
unencrypted public keys and both encrypted and unencrypted secret keys. The
attributes also support storing the encrypted key cipher and key derivation
functions so that the encryption key can be recreated from a secret. There are
also attributes used to support threshold signing key splitting and
recombining.

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

**Comment**
: A Multikey object contains a string that describes the key. This is
application spedific.

**Attributes**
: Each Multikey object consists of a set of attributes identified by a
name/numerical value with an associated binary value. This is how the Multikey
supports all of the different codec-specific data and use cases.

## [Specification](#specification)

The following diagram shows the overall structure of a Multikey. A Multikey
is identified by the `0x3a` sigil followed by a codec sigil, comment, and a 
variable number of codec-specific attributes.

```
multikey
sigil         key comment
|                  |
v                  v
0x3a <varuint> <comment> <attributes>
         ^                    ^
         |                    |
    key codec sigil       key attributes


<comment> ::= <varbytes>

                         variable number of attributes
                                       |
                            ______________________
                           /                      \
<attributes> ::= <varuint> N(<varuint>, <varbytes>)
                     ^           ^          ^
                    /           /           |
            count of      attribute     attribute
          attributes     identifier       value


<varbytes> ::= <varuint> N(OCTET)
                   ^        ^
                  /          \
          count of            variable number
            octets            of octets
```

The Multikey format is designed to support any kind of arbitrarily complex key
data in such a way that tools are able to know exactly how many octets are in
the Multikey object so that it can skip over it if they don't support the key
codec.

### [Key Comment](#key-comment)

By convention, every key has a comment that is easy to extract from the
Multikey data structure by any tool, even if they do not support a specific key
codec.

### [Attributes](#attributes)

Each multikey consists of a set of attributes that are codec-specific and only
meaningful through codec-specific "views" on the attribute table. Currently the
list of defined attributes supports all of the use cases identified so far. The
design of the attribute store leaves room for new attributes in the future.
Please file an issue if you have a use case that requires a new attribute type.

**KeyIsEncryptied (0x00)**
: The value is a single boolean byte flag; true if the key data is encrypted.

**KeyData (0x01)**
: The value is the key data.

**CipherCodec (0x02)**
: The codec sigil specifying the encryption cipher used to encrypt the key data.

**CipherKeyLen (0x03)**
:  The number of octets in the key encryption key.

**CipherNonce (0x04)**
:  The nonce value for the key encryption cipher.

**KdfCodec (0x05)**
:  The codec sigil specifying the key encryption key derivation function.

**KdfSalt (0x06)**
: The salt value used in the key encryption key derivation function.

**KdfRounds (0x07)**
: The number of rounds used in the key encryption key derivation function.

**Threshold (0x08)**
: The number of threshold signature key shares needed to recreate the key.

**Limit (0x09)**
: The total number of shares in the split threshold singature key.

**ShareIdentifier (0x0a)**
: The identifer for a given threshold key share.

**ThresholdData (0x0b)**
: Threshold signing codec-specific data. This is typically use to store the 
accumulated key shares while gathring enough shares to recreate the key.

### [Secret Keys](#secret-keys)

Secret keys are sensitive and should always be kept encrypted when at rest.
The attributes in the Multikey specify whether the key is encrypted and which
encryption method was used as well as the key derivation method and parameters.

This applies to both symmetric keys as well as the secret key part of a public
key pair.

## [Examples](#examples)

### [An Encrypted EdDSA Secret Key](#an-encrypted-eddsa-secret-key)

This example shows how a Multikey stores an EdDSA private key encrypted using
the ChaCha20-Poly1305 AEAD symmetric encryption algorithm using a key derived
using the Bcrypt PBKDF function with 10 rounds and a 32-byte salt value:

```
3a                  -- varuint, multikey sigil 
[80 26]             -- varuint, Ed25519 private key codec 
08                  -- varuint, length of comment 
  "test key"        -- 8 octets of UTF-8 comment data
0a                  -- varuint, 10 attributes
  00                -- varuint, AttrId::KeyIsEncrypted
    01              -- varuint, attribute length
      [01]          -- true, it is encrypted!
  01                -- varuint, AttrId::KeyData
    30              -- varuint, attribute length (48 octets)
      [48 octets]   -- ciphertext
  02                -- varuint, AttrId::CipherCodec
    02              -- varuint, attribute length
      [a5 01]       -- varuint, ChaCha20-Poly1305 codec
  03                -- varuint, AttrId::CipherKeyLen
    01              -- varuint, attribute length
      [20]          -- 32 byte key length
  04                -- varuint, AttrId::CipherNonce
    08              -- varuint, attribute length
      [8 octets]    -- nonce
  05                -- varuint, AttrId::KdfCodec
    03              -- varuint, attribute length
      [8d a0 03]    -- varuint, Bcrypt KDF codec
  06                -- varuint, AttrId::KdfSalt
    20              -- varuint, attribute length (32 octets)
      [32 octets]   -- salt
  07                -- varuint, AttrId::KdfRounds
    01              -- varuint, attribute length
      [0a]          -- varuint, 10 kdf rounds
```

In this example the encoding starts off with the multikey sigil (`0x3a`)
followed by the varuint codec value for an EdDSA private key (`0x8026`).
Following that there is the key comment encoded as varbytes with a varuint 
length value followed by that number of octets of UTF-8 string data; in this 
case `"test key"`. The rest of the encoded Multikey is the list of attributes.
All attributes are optional. Each attribute has an attribute ID followed by the
varuint encoded length of the attribute data followed by the octets of
attribute data. In this case of an encrypted EdDSA secret key, there are
attributes for the flag showing it is encrypted, the encrypted key bytes, the
encryption codec, the encryption key length, the nonce length, the nonce data,
the kdf codec, the kdf salt length, the kdf salt, and the kdf rounds.

### [An Es256K Public Key](#an-es256k-public-key)

This example shows how a Multikey encodes an Es256K public key.

```
3a                  -- varuint, multikey sigil 
[e7 01]             -- varuint, Secp256k1 public key codec
08                  -- varuint, length of comment
  "test key"        -- 8 octets of UTF-8 comment data
01                  -- varuint, 1 attribute
  01                -- varuint, AttrId::KeyData
    21              -- varuint, attribute length (33 octets)
      [33 octets]   -- key data
```

In this example the encoding starts off with the multikey sigil ('0x3a')
followed by the varuint encoded codec value for an Es256K public key
(`0xe701`). Following that is 8 octets of UTF-8 comment data and the varuint 
encoded number of attributes in the attributes table. In this case there is 
only one attrbute, the key data itself which is encoded as the varuint 
encoded `KeyData` attribute ID, followed by the varuint encoded attribute
length (33 octets), and the 33 octets of key data.

[0]: https://cryptid.tech
[1]: https://github.com/cryptidtech/provenance-specifications/
[2]: https://github.com/multiformats/multiformats
[3]: https://github.com/CommunitySpecification/1.0
[4]: https://github.com/multiformats/unsigned-varint/blob/master/README.md
[5]: https://github.com/multiformats/multicodecs/blob/master/table.csv
