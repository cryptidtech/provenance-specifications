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

This document refers to `CID` content identifiers. The normative reference for
which can be found in the [multiformats cid specification][6]

This document refers to `nonce` values. The normative reference for which can 
be found in the [multiformats nonce specification][7]

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
is identified by the `0x07` sigil followed by a `nonce` and a `cid`.

```
vlad sigil    cid value
 |              /
 v             v
0x07 <nonce> <cid>
        ^
        |
    nonce value
```

In some cases, the `varbytes` in the `nonce` are a `multisig` encoded digital
signature over the `cid` value in the VLAD. The following diagram illustrates a
signed VLAD.

```
vlad sigil    cid value
 |              /
 v             v
0x07 <nonce> <cid>
        ^
        |
    nonce value

<nonce> ::= 0x3b <varbytes>
             ^
            / 
 nonce sigil

<varbytes> ::= <varuint> <multisig octets>
                   ^             ^
                  /              |
          count of          variable number
            octets         of multisig octets
```

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
All we have to do is construct a tuple identifer from a large random value--
commonly called a `nonce`--and a cryptographic commitment to a validation
function. By combinding content addressable storage and WASM as universally
executable code, any WASM code that validates data using cryptography may be
hashed to create a content address that is both an immutable identifier for
retrieving the WASM code and also a cryptographic verification method to ensure
that the WASM code has not been modified and retains its original form down to
the bit.

Combining the nonce and the content address (e.g. `cid`) of a WASM validation
function gives us an identifier that is both unique and a cryptographic
commitment to a validation function; the same set of primary properties as
public keys. However this new identifer is not based off of key material and is
not subject to compromise. Any change in the WASM code is detectable and
produces a different identifier. Any change in the nonce also creates a
different identifier. The only way for one of these new identifiers to remain
relevant over time is to remain unchanged.

This new identifer is called a "Verifiable Long-lived ADdresses" or "VLAD". The
security of the VLAD can be further enhanced by replacing the 256-bit nonce
value with a digital signature--preferably an ECDSA/EdDSA signature for
compactness--over the WASM content address. The resulting signature has the
same amount of entropy as a random nonce but has the added benefit of allowing
the creator of the VLAD to prove they created it. More on this later when we
discuss VLADs as identifiers for provenance logs.

To reduce the tight binding and fragility of VLADs, they are encoded using the
multiformats standard. A VLAD therefore begins with the multicodec sigil
identifying the data as a VLAD (e.g. `0x07`) followed by two multiformat
encoded values, a nonce (e.g. `0x3b`) or a multisig (e.g. `0x39`) followed by a
content addres CID (e.g. `0x01` v1, `0x02` v2, or `0x03` v3).

### [VLAD Creation](#vlad-creation)

1. Select the WASM validation function they would like to use for validating
   PUT operations in the global DHT for updating the mutable forward pointer.
   There is likely to be a small well-known set of WASM validation functions
   used. The CID of the chosen WASM is the value used in provenance log
   creation.
2. Generate an ephemeral cryptographic public key pair.
3. Digitally sign the CID of the WASM code to create the random value for the
   VLAD.
4. Encode the digital signature in the multisig multiformat, concatenate the
   CID to the multisig and add the VLAD sigil to the front to create a
   multiformat encoded VLAD value.

[0]: https://cryptid.tech 
[1]: https://github.com/cryptidtech/provenance-specifications/
[2]: https://github.com/multiformats/multiformats
[3]: https://github.com/CommunitySpecification/1.0
[4]: https://github.com/multiformats/unsigned-varint/blob/master/README.md
[5]: https://github.com/multiformats/multicodecs/blob/master/table.csv
[6]: https://github.com/multiformats/cid/blob/master/README.md
[7]: https://github.com/cryptidtech/provenance-specifications/blob/main/specifications/nonce.md
[8]: https://github.com/cryptidtech/provenance-specifications/blob/main/specifications/multisig.md
