# Very Long-Lived Address (VLAD) Specification
**Version: 1.0.0** \
**Status: Stable** \
© 2024–2026 Cryptid Technologies, Inc.

[![](https://img.shields.io/badge/made%20by-Cryptid%20Technologies-gold.svg?style=flat-square)][0]
[![](https://img.shields.io/badge/project-provenance-purple.svg?style=flat-square)][1]
[![](https://img.shields.io/badge/project-multiformats-blue.svg?style=flat-square)][2]

This specification is subject to the [Community Specification License 1.0][3]

## Contents
1. [Foreword](#foreword)
2. [Introduction](#introduction)
    1. [Current Status](#current-status)
    2. [Normative References](#normative-references)
    3. [Terms and Definitions](#terms-and-definitions)
3. [Specification](#specification)
    1. [Structure](#structure)
    2. [Combined Signature Requirement](#combined-signature-requirement)
    3. [WASM Validation](#wasm-validation)
    4. [Encoding and Decoding](#encoding-and-decoding)
    5. [Verification](#verification)
4. [Annex](#annex)
    1. [Do Not Use Public Keys as Identifiers](#do-not-use-public-keys-as-identifiers)
    2. [A Better Identifier](#a-better-identifier)
    3. [VLAD Creation](#vlad-creation)
    4. [VLAD-to-Genesis Linkage](#vlad-to-genesis-linkage)
    5. [Relationship to `wacc` and `provenance-log`](#relationship-to-wacc-and-provenance-log)

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

This specification defines the Very Long-Lived Address (VLAD). A VLAD is a
permanent identifier that replaces public keys as identifiers in distributed
systems. It is a multiformat-encoded combined `Multisig` whose message holds
the binary WASM bytecode of a first-lock script and whose signature is over
that bytecode, made by an ephemeral key pair.

The VLAD is content-addressed by the Blake3 hash of its own bytes, making it
impractical to forge and independent of any particular key. The ephemeral
private key is destroyed after it signs the VLAD and the genesis entry of the
provenance log. This makes the VLAD permanent and unforgeable.

### [Current Status](#current-status)

This specification (v1.0.0) defines the `Vlad(Multisig)` construction. It
replaces the earlier v0.0.1 pre-draft, which described a
`0x8724 <nonce> <cid>` tuple structure. The tuple structure is obsolete. The
current structure is a combined `Multisig` prefixed by the VLAD sigil.

### [Normative References](#normative-references)

This document refers to `varuint` encoded values throughout. The normative
reference for which can be found in the [multiformats unsigned-varint
specification][4].

This document refers to `sigils` that identify a codec or data type. The
normative reference for the list of sigils can be found in the [multiformats
multicodecs table][5].

This document refers to `multi-sig` encoded digital signatures. The normative
reference for which can be found in the [multiformats multi-sig
specification][8].

This document refers to `multi-key` encoded cryptographic keys. The normative
reference for which can be found in the [multi-key specification][9].

This document refers to provenance logs (`plog`). The normative reference for
which can be found in the [provenance logs specification][10].

### [Terms and Definitions](#terms-and-definitions)

**Varuint**
: An unsigned integer variable encoded in a variable number of bytes.

**Sigil**
: A constant value that maps to a codec or data type.

**Combined Multisig**
: A `Multisig` whose `message` field is non-empty. The signature is over the
  message. This is distinct from a detached `Multisig`, whose `message` field
  is empty and whose signature is over an externally supplied message.

**First-Lock Script**
: A WASM binary that the provenance log executes as the lock script for the
  genesis (first) entry. The VLAD carries the first-lock script as the
  `message` field of its inner `Multisig`.

**Ephemeral Key Pair**
: A cryptographic key pair generated for the sole purpose of signing the VLAD
  and the genesis entry. The private key is destroyed after signing.

## [Specification](#specification)

### [Structure](#structure)

A VLAD is a multiformat-encoded value. It begins with the VLAD multicodec
sigil (`0x1207`, encoded as the varuint `0x87 0x24`) followed by a combined
`Multisig`.

```
<vlad> ::= 0x8724 <multisig>
             ^           ^
            /             \
    vlad sigil       combined multisig

<multisig> ::= 0xb924 <varbytes>
                 ^
                /
         multisig sigil (0x1239)

<varbytes> ::= <varuint> <multisig octets>
                    ^           ^
                   /             \
           count of          variable number
             octets         of multisig octets
```

The `Multisig` is a combined signature. Its `message` field holds the binary
WASM bytecode of the first-lock script. Its signature is over that bytecode,
made by the ephemeral key pair.

The overall byte layout is:

```
0x87 0x24              VLAD sigil (varuint 0x1207)
0xb9 0x24              Multisig sigil (varuint 0x1239)
<varuint>              length of the multisig body
<multisig body>        the multisig: signature codec, message (WASM), signature
```

### [Combined Signature Requirement](#combined-signature-requirement)

The inner `Multisig` MUST be a combined signature. A combined signature has a
non-empty `message` field. The message is the WASM first-lock script bytecode.
The signature is over the message.

A detached signature (empty `message` field) is invalid for a VLAD. A VLAD
with a detached `Multisig` MUST be rejected by the decoder.

### [WASM Validation](#wasm-validation)

The `message` field of the inner `Multisig` MUST begin with the WASM binary
magic bytes: `0x00 0x61 0x73 0x6d` (the ASCII string `\0asm`). A VLAD whose
message does not begin with these bytes is invalid and MUST be rejected by
the decoder.

The VLAD does not interpret the WASM beyond the magic header. The WASM is
opaque bytes to the VLAD. The `Script` type that executes the WASM lives in
`provenance-log`, not in the VLAD crate. The VLAD only carries the bytes.

A null VLAD (whose inner `Multisig` is `Multisig::null()`) is exempt from
both the combined-signature and WASM-magic validation. The null VLAD is a
sentinel value.

### [Encoding and Decoding](#encoding-and-decoding)

The VLAD is encoded as the VLAD sigil varuint followed by the `Multisig`
encoding. The `From<Vlad> for Vec<u8>` implementation produces this layout.

The VLAD is decoded by reading the sigil, confirming it is `0x1207`, then
reading the `Multisig`. After decoding, the VLAD is validated: the inner
`Multisig` must be combined (non-empty message) and the message must begin
with the WASM magic bytes.

A VLAD may be multibase-encoded for text representation. The preferred base
encoding is `Base32Lower`. The `EncodedVlad` type (from `multi-vlad`) handles
multibase encoding and decoding.

### [Verification](#verification)

A VLAD is verified by checking the signature in the inner `Multisig` against
the ephemeral public key. The `Vlad::verify(&self, mk: &Multikey)` method
performs this check. It calls `mk.verify_view()` and verifies the `Multisig`
signature over the WASM message.

This proves the VLAD was created by someone who held the private half of the
ephemeral key pair.

## [Annex](#annex)

### [Do Not Use Public Keys as Identifiers](#do-not-use-public-keys-as-identifiers)

One of the greatest failings of the current web-of-trust is the use of public
keys as identifiers. Public key pairs are subject to attack and compromise in
a number of ways necessitating regular rotation and occasional recovery to
ensure a high degree of security and resiliency. Using public keys as
identifiers means that whenever a key is rotated or revoked, any external
references using the public key identifier become broken. It is this fragility
that has prevented a global web-of-trust from being an emergent property of our
socially networked lives.

Why do we use public keys as identifiers? The answer is that they are a compact
and convenient value with two primary properties:

1. Public keys have enough entropy that they can be used as identifiers for
   a very large number of things. Enough so that there could be a public key
   identifying every atom in the known universe and you wouldn't have used
   even a quarter of the available numbers.
2. A public key is a cryptographic commitment to a validation function (e.g.
   public key digital signature) that can be used to verify other data and
   bind ownership of that other data to the controller of the public key pair.

It is plain to see that public keys solve the problem of cryptographically
enforced proof-of-control while also being great random identifiers. However
their vulnerability to attack and compromise makes them bad identifiers for
distributed systems that require loose coupling and resiliency over large spans
of time. Key rotation is good security hygiene. It is also the primary reason
why the web-of-trust isn't anti-fragile.

### [A Better Identifier](#a-better-identifier)

Given the two primary properties of public keys listed above, it is conceivable
that another type of identifier can be constructed with those same properties
while also lacking the vulnerabilities and limitations to durability over time.
All we have to do is construct an identifier from a large random value and a
cryptographic commitment to a validation function. By combining content
addressable storage and WASM as universally executable code, any WASM code that
validates data using cryptography may serve as a cryptographic verification
method.

The VLAD replaces the public-key identifier with a combined `Multisig`. The
`Multisig` message field holds the WASM first-lock script bytecode. The
signature is over that bytecode, made by an ephemeral key pair. The signature
bytes provide the random identifier (enough entropy to be unique). The WASM
bytecode provides the cryptographic commitment to a validation function. The
ephemeral private key is destroyed after signing, so the identifier is
permanent and not subject to key compromise or rotation.

The VLAD is content-addressed by the Blake3 hash of its own bytes. This makes
it impractical to forge and independent of any particular key. The VLAD is
encoded using the multiformats standard: it begins with the VLAD multicodec
sigil (`0x1207`) followed by the `Multisig` encoding.

### [VLAD Creation](#vlad-creation)

1. Generate an ephemeral cryptographic key pair. Any signing key type
   supported by `multi-key` may be used (Ed25519, Lamport, etc.). Lamport
   keys are one-time, which is a natural fit for a VLAD (it signs exactly one
   message).
2. Write the WASM first-lock script. The script embeds the ephemeral public
   key and the signing public key as constants. The script logic verifies
   signatures and public key matches when executed by the wacc VM.
3. Sign the WASM bytecode with the ephemeral private key. This creates a
   combined `Multisig` with the WASM as the `message` field and the signature
   over the WASM.
4. Prefix the `Multisig` with the VLAD sigil (`0x1207`, encoded as the
   varuint `0x87 0x24`). This produces the multiformat-encoded VLAD value.
5. Destroy the ephemeral private key (zeroize the memory, drop the key
   material).

The VLAD is now permanent. It cannot be forged without the ephemeral private
key, which is destroyed. The WASM first-lock script is committed to by the
signature and is immutable.

### [VLAD-to-Genesis Linkage](#vlad-to-genesis-linkage)

The VLAD is cryptographically linked to the genesis (first) entry of a
provenance log. The linkage is bidirectional and unforgeable.

The WASM first-lock script (carried as `vlad.message()`) is used as the
`first_lock` of the plog. The plog `Log` builder seeds
`first_lock = Script::Bin(root_key, vlad.message())`. When `Log::verify()`
runs, it executes the WASM first-lock script through the wacc VM to authorize
the genesis entry.

The WASM script verifies two signatures on the genesis entry:

1. The `"signing"` proof — a signature over the entry-without-proofs by the
   signing key.
2. The `"ephemeral"` proof — a signature over the entry-without-proofs by the
   ephemeral key (the same key that signed the VLAD).

The WASM script also verifies that the entry's `/peer/pubkey` and
`/peer/ephemeral_pubkey` match the public keys embedded as constants in the
WASM.

This creates a bidirectional link: the VLAD commits to the WASM, the WASM
commits to the public keys, and the genesis entry commits to the VLAD (the
entry's `vlad` field equals the VLAD). An attacker cannot forge a new VLAD
for an existing plog (the ephemeral key is destroyed), and an attacker who
compromises the signing key cannot forge a new genesis entry (the WASM
requires the ephemeral proof).

### [Relationship to `wacc` and `provenance-log`](#relationship-to-wacc-and-provenance-log)

The VLAD does not depend on `wacc` or `provenance-log`. The VLAD carries the
WASM first-lock script as opaque bytes. It validates the bytes only by the
`\0asm` magic header. It does not execute the WASM.

The `Script` type that executes the WASM lives in `provenance-log`
(`provenance-log/src/script.rs`). It is not in `wacc` and not in `multi-vlad`.

The `wacc` crate provides the WASM VM that `provenance-log` uses to execute
lock and unlock scripts. `wacc` depends on `multi-cid` for the `Blocks` trait
and `Cid` type, but not on `multi-vlad`.

[0]: https://cryptid.tech
[1]: https://github.com/cryptidtech/provenance-specifications/
[2]: https://github.com/multiformats/multiformats
[3]: https://github.com/CommunitySpecification/1.0
[4]: https://github.com/multiformats/unsigned-varint/blob/master/README.md
[5]: https://github.com/multiformats/multicodecs/blob/master/table.csv
[8]: https://github.com/cryptidtech/provenance-specifications/blob/main/specifications/multi-sig.md
[9]: https://github.com/cryptidtech/provenance-specifications/blob/main/specifications/multi-key.md
[10]: https://github.com/cryptidtech/provenance-specifications/blob/main/specifications/provenance-logs.md
