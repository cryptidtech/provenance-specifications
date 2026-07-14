# Multi-Key Specification
**Version: 0.0.1** \
**Status: Pre-draft** \
© 2024 Cryptid Technologies, Inc.

[![](https://img.shields.io/badge/made%20by-Cryptid%20Technologies-gold.svg?style=flat-square)][CRYPTID]
[![](https://img.shields.io/badge/project-provenance-purple.svg?style=flat-square)][PROVSPEC]
[![](https://img.shields.io/badge/project-multiformats-blue.svg?style=flat-square)][MULTIFORMATS]

This specification is subject to the [Community Specification License 1.0][COMMUNITYSPEC].

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
    2. [A Secp256k1 Public Key](#a-secp256k1-public-key)

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
[specifications repository][PROVSPEC].

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

This version of the Multi-Key specification defines attributes to support
unencrypted public keys and both encrypted and unencrypted secret keys. The
attributes also support storing the encrypted key cipher and key derivation
functions so that the encryption key can be recreated from a secret. There are
also attributes used to support threshold signing key splitting and
recombining.

### [Normative References](#normative-references)

This document refers to `varuint` encoded values throughout. The normative 
reference for which can be found in the [multiformats unsigned-varint
specification][MULTIFORMATSREADME].

This document also refers to `sigils` the identify a codec or data type. The 
normative reference for the list of sigils can be found in the [multiformats 
multicodecs table][MULTIFORMATSTABLE].

### [Terms and Definitions](#terms-and-definitions)

**Varuint**
: An unsigned integer variable encoded in a variable number bytes.

**Sigil**
: A constant value that maps to a codec or data type.

**Comment**
: A Multi-Key object contains a string that describes the key. This is
application specific.

**Attributes**
: Each Multi-Key object consists of a set of attributes identified by a
name/numerical value with an associated binary value. This is how the Multi-Key
supports all the different codec-specific data and use cases.

## [Specification](#specification)

The following diagram shows the overall structure of a Multi-Key. A Multi-Key
is identified by the `0x123a` sigil followed by a codec sigil, comment, and a
variable number of codec-specific attributes. The Multi-Key sigil is encoded as
a varuint `0xba24`.

```
multi-key
sigil         key comment
|                  |
v                  v
0xba24 <varuint> <comment> <attributes>
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

The Multi-Key format is designed to support any kind of arbitrarily complex key
data in such a way that tools are able to know exactly how many octets are in
the Multi-Key object so that it can skip over it if they don't support the key
codec.

### [Key Comment](#key-comment)

By convention, every key has a comment that is easy to extract from the
Multi-Key data structure by any tool, even if they do not support a specific key
codec.

### [Attributes](#attributes)

Each Multi-Key consists of a set of attributes that are codec-specific and only
meaningful through codec-specific "views" on the attribute table. Currently the
list of defined attributes supports all of the use cases identified so far. The
design of the attribute store leaves room for new attributes in the future.
Please file an issue if you have a use case that requires a new attribute type.

**KeyIsEncrypted (0x00)**
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

**DkgOwnerId (0x0c)**
: DKG: the VLAD bytes of the session owner.

**DkgParticipants (0x0d)**
: DKG: the CBOR-encoded list of participant VLADs.

**DkgThreshold (0x0e)**
: DKG: the t-of-n threshold value (u16 little-endian).

**DkgLimit (0x0f)**
: DKG: the t-of-n limit value (u16 little-endian).

**DkgGroupPublicKey (0x10)**
: DKG: the group public key bytes.

**DkgIdentifier (0x11)**
: DKG: this participant's frost-dkg identifier bytes.

**DkgSessionId (0x12)**
: DKG: the session UUID bytes.

**ShareVerifiers (0x13)**
: Feldman verifier commitment set for a verifiable key share (CBOR list of
group-element byte strings). Present on Feldman shares of split keys.

**ShareDual (0x14)**
: Secondary dual share for seed-based keys (Ed25519/X25519): the CBOR
encoding of the Feldman scalar share carried alongside the gf256 seed
share. Lets these keys mirror the ECC shares without losing exact restore.

**ThresholdGroupPublicKey (0x15)**
: Split (Shamir) threshold key: the group/combined BLS public key bytes.
This is the split-key analogue of `DkgGroupPublicKey`; it is stamped on a
controller Multi-Key so that the combined threshold signature can be
verified against the group public key.

**ThresholdParticipants (0x16)**
: Split (Shamir) threshold key: a CBOR-encoded map from share-identifier
bytes (the Shamir x-coordinate, exactly as stored in `ShareIdentifier`) to
a participant record carrying that participant's VLAD and its BLS public
key share v_i bytes.

**ThresholdMarkerSig (0x17)**
: Split (Shamir) threshold key: a Multi-Sig (signature) over the canonical
encoding of the marker bundle (group public key ‖ participants ‖ threshold
‖ limit), produced by the controller's signing key. Lets a verifier reject
a tampered marker before trusting it (defeats TSIG-1 marker forgery).

### [Secret Keys](#secret-keys)

Secret keys are sensitive and should always be kept encrypted when at rest.
The attributes in the Multi-Key specify whether the key is encrypted and which
encryption method was used as well as the key derivation method and parameters.

This applies to both symmetric keys as well as the secret key part of a public
key pair.

## [Examples](#examples)

### [An Encrypted EdDSA Secret Key](#an-encrypted-eddsa-secret-key)

This example shows how a Multi-Key stores an EdDSA private key encrypted using
the ChaCha20-Poly1305 AEAD symmetric encryption algorithm using a key derived
using the Bcrypt PBKDF function with 10 rounds and a 32-byte salt value:

```
ba 24               -- varuint, multi-key sigil (0x123a)
80 26               -- varuint, Ed25519 private key codec (0x1300)
08                  -- varuint, length of comment 
  "test key"        -- 8 octets of UTF-8 comment data
08                  -- varuint, 8 attributes
  00                -- varuint, AttrId::KeyIsEncrypted
    01              -- varuint, attribute length
      01            -- true, it is encrypted!
  01                -- varuint, AttrId::KeyData
    30              -- varuint, attribute length (48 octets)
      [48 octets]   -- ciphertext
  02                -- varuint, AttrId::CipherCodec
    03              -- varuint, attribute length
      80 c0 02      -- varuint, ChaCha20-Poly1305 codec (0xa000)
  03                -- varuint, AttrId::CipherKeyLen
    01              -- varuint, attribute length
      20            -- 32 byte key length
  04                -- varuint, AttrId::CipherNonce
    0c              -- varuint, attribute length
      [12 octets]   -- nonce
  05                -- varuint, AttrId::KdfCodec
    03              -- varuint, attribute length
      8d a0 03      -- varuint, Bcrypt KDF codec
  06                -- varuint, AttrId::KdfSalt
    20              -- varuint, attribute length (32 octets)
      [32 octets]   -- salt
  07                -- varuint, AttrId::KdfRounds
    01              -- varuint, attribute length
      0a            -- varuint, 10 kdf rounds
```

In this example the encoding starts off with the Multi-Key sigil (`0xba24`)
followed by the varuint codec value for an EdDSA private key (`0x8026`).
Following that there is the key comment encoded as varbytes with a varuint 
length value followed by that number of octets of UTF-8 string data; in this 
case `"test key"`. The rest of the encoded Multi-Key is the list of attributes.
All attributes are optional. Each attribute has an attribute ID followed by the
varuint encoded length of the attribute data followed by the octets of
attribute data. In this case of an encrypted EdDSA secret key, there are
attributes for the flag showing it is encrypted, the encrypted key bytes, the
encryption codec, the encryption key length, the nonce length, the nonce data,
the kdf codec, the kdf salt length, the kdf salt, and the kdf rounds.

### [A Secp256k1 Public Key](#a-secp256k1-public-key)

This example shows how a Multi-Key encodes an secp256k1 public key.

```
ba 24               -- varuint, multi-key sigil  (0x123a)
e7 01               -- varuint, secp256k1 public key codec (0xe7)
08                  -- varuint, length of comment
  "test key"        -- 8 octets of UTF-8 comment data
01                  -- varuint, 1 attribute
  01                -- varuint, AttrId::KeyData
    21              -- varuint, attribute length (33 octets)
      [33 octets]   -- key data
```

In this example the encoding starts off with the Multi-Key sigil (`0xba24`)
followed by the varuint encoded codec value for an secp256k1 public key
(`0xe701`). Following that is 8 octets of UTF-8 comment data and the varuint 
encoded number of attributes in the attributes table. In this case there is 
only one attrbute, the key data itself which is encoded as the varuint 
encoded `KeyData` attribute ID, followed by the varuint encoded attribute
length (33 octets), and the 33 octets of key data.

[CRYPTID]: https://cryptid.tech/
[PROVSPEC]: https://github.com/cryptidtech/provenance-specifications/
[MULTIFORMATS]: https://github.com/multiformats/multiformats/
[COMMUNITYSPEC]: https://github.com/CommunitySpecification/1.0
[MULTIFORMATSREADME]: https://github.com/multiformats/unsigned-varint/blob/master/README.md
[MULTIFORMATSTABLE]: https://github.com/multiformats/multicodecs/blob/master/table.csv
