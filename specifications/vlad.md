# Very Long-Lived Address (VLAD) Specification
**Version: 0.0.1** \
**Status: Pre-draft** \
© 2024 Cryptid Technologies, Inc.

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
4. [Annex](#annex)
    1. [Do Not Use Public Keys as Identifiers](#do-not-use-public-keys-as-identifiers)
    2. [A Better Identifier](#a-better-identifier)
    3. [VLAD Creation](#vlad-creation)

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

This specification outlines how to encode a very long-lived address (VLAD) in a
multiformat compatible way.

### [Current Status](#current-status)

VLADs are fully defined by this specification and meet all of the needs of the
current set of use cases.

### [Normative References](#normative-references)

This document refers to `varuint` encoded values throughout. The normative
reference for which can be found in the [multiformats unsigned-varint
specification][4].

This document refers to `sigils` the identify a codec or data type. The
normative reference for the list of sigils can be found in the [multiformats
multicodecs table][5].

This document refers to `multisig` encoded digital signatures. The normative
reference for which can be found in the [multiformats multisig
specification][8].

### [Terms and Definitions](#terms-and-definitions)

**Varuint**
: An unsigned integer variable encoded in a variable number bytes.

**Sigil**
: A constant value that maps to a codec or data type.

## [Specification](#specification)

The following diagram shows the overall structure of a VLAD. A VLAD
is identified by the `0x1207` sigil followed by a `multisig`. The sigil is
encoded as a varuint (`0x8724`).

```
vlad sigil   multisig value
 |              /
 v             v
0x8724 <multisig>
```

The `multisig` is a combined digital signature where the signed message is
embedded within the multisig structure. The message MUST be a valid WebAssembly
binary (i.e. it starts with the WASM magic bytes `\0asm`). This WASM binary is
the validation function used to authorize updates to the provenance log
identified by this VLAD.

```
<multisig> ::= <multisig sigil> <signature codec> <message> <attributes>
                      ^                ^               ^          ^
                     /                /               /          |
              multisig          algorithm       varbytes     signature
               sigil             codec        encoded WASM   attributes
                                              binary (the     (includes
                                              combined msg)  sig data)
```

The VLAD is a cryptographic commitment: the digital signature binds the WASM
validation function to the ephemeral signing key used at creation time. Any
modification to the WASM code or the signature invalidates the VLAD.

## [Annex](#annex)

### [Do Not Use Public Keys as Identifiers](#do-not-use-public-key-as-identifiers)

One of the greatest failings of the current web-of-trust is the use of public
keys as identifiers. Public key pairs are subject to attack and compromise in a
number of ways necessitating regular rotation and occassional recovery to
ensure a high degree of security and resiliency. Using public keys as
identifiers means that whenever a key is rotated or revoked, any external
references using the public key identifier becomes broken. It is this fragility
that has prevented a global web-of-trust from being an emergent property of our
socially networked lives.

Why do we use public keys as identifiers? The answer is that they are a compact
and convenient value with two primary properties:

1. Public keys have enough entropy that they can be used as identifiers for
   a very large number of things. Enough so that there could be a public key
   identifying every atom in the known universe and you wouldn't have used
   event a quarter of the available numbers.
2. A public key is a cryptographic commitment to a validation function (e.g.
   public key digital signature) that can be used to verify other data and
   bind ownership of that other data to the controller of the public key pair.

It is plain to see that public keys solve the problem of cryptographically
enforced proof-of-control while also being great random identifiers. However
their vulnerability to attack and compromise makes them bad identifers for
distributed systems that require loose coupling and resiliency over large spans
of time. Key rotation is good security hygiene. It is also the primary reason
why the web-of-trust isn't anti-fragile.

### [A Better Identifer](#a-better-identifier)

Given the two primary properties of public keys listed above, it is conceivable
that another type of identifer can be constructed with those same properties
while also lacking the vulnerabilities and limitations to durability over time.
By combining content addressable storage and WASM as universally executable
code, any WASM code that validates data using cryptography can serve as a
portable, deterministic validation function. A digital signature over that WASM
binary creates a cryptographic commitment that binds the validation function to
a specific creation event without relying on long-lived key material.

A VLAD ("Verifiable Long-lived ADdress") is constructed by digitally signing a
WASM validation function binary with an ephemeral key pair. The resulting
`multisig` structure contains both the signature and the signed message (the
WASM binary) in a combined format. The VLAD sigil is prepended to this multisig
to form the complete identifier.

This construction has the same two primary properties as public keys:

1. The signature provides sufficient entropy to serve as a globally unique
   identifier.
2. The embedded WASM binary is a cryptographic commitment to a validation
   function that can be used to verify provenance log operations.

However, unlike a public key identifier, a VLAD is not based on long-lived key
material and is not subject to compromise. The ephemeral key pair used to sign
the VLAD is discarded after creation. Any modification to the WASM code or the
signature invalidates the VLAD. The only way for a VLAD to remain relevant over
time is to remain unchanged.

To reduce tight binding and fragility, VLADs are encoded using the multiformats
standard. A VLAD begins with the multicodec sigil identifying the data as a
VLAD (e.g. `0x1207`) followed by a multisig (e.g. `0x1239`) encoded digital
signature containing the WASM validation function as the signed message.

### [VLAD Creation](#vlad-creation)

1. Select the WASM validation function to use for validating PUT operations in
   the global DHT for updating the mutable forward pointer. There is likely to
   be a small well-known set of WASM validation functions used.
2. Generate an ephemeral cryptographic key pair (e.g. Ed25519).
3. Digitally sign the WASM binary with the ephemeral key to produce a combined
   multisig (i.e. the WASM binary is embedded as the signed message within the
   multisig structure).
4. Prepend the VLAD sigil to the multisig to create the multiformat encoded
   VLAD value.
5. Discard the ephemeral key pair. It is not needed after VLAD creation.

[0]: https://cryptid.tech
[1]: https://github.com/cryptidtech/provenance-specifications/
[2]: https://github.com/multiformats/multiformats
[3]: https://github.com/CommunitySpecification/1.0
[4]: https://github.com/multiformats/unsigned-varint/blob/master/README.md
[5]: https://github.com/multiformats/multicodecs/blob/master/table.csv
[8]: https://github.com/cryptidtech/provenance-specifications/blob/main/specifications/multisig.md
