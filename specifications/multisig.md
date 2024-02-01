# Multisig Specification
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
    1. [Signature Message](#signature-message)
    2. [Attributes](#attributes)

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
[specifications repository][4].

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

This specification outlines how to encode arbitrary digital signatures data in 
a codec-agnostic way while also supporting codec-specific attribute read-write 
and codec-specific cryptographic operations. Digital signatures are created 
from Multikey values and data to be signed.

### [Current Status](#current-status)

This versino of the Multisig specification 

### [Normative References](#normative-references)

This document refers to `varuint` encoded values throughout. The normative 
reference for which can be found in the [multiformats unsigned-varint
specification][5].

This document also refers to `sigils` the identify a codec or data type. The 
normative reference for the list of sigils can be found in the [multiformats 
multicodecs table][6].

### [Terms and Definitions](#terms-and-definitions)

**Varuint**
: An unsigned integer variable encoded in a variable number bytes.

**Sigil**
: A constant value that maps to a codec or data type.

**Attributes**
: Each Multisig object consists of a set of attributes identified by a
name/numerical value with an associated binary value. This is how the Multikey
supports all of the different codec-specific data and use cases.

## [Specification](#specification)

The following diagram explains the overall structure of a Multisig. A Multisig
is identified by the `0x39` sigil followed by a key codec sigil, message, and a 
variable number of codec-specific attributes.

```
signing codec sigil     signature attributes
         |                     |
         v                     v
0x39 <varuint> <message> <attributes>
^                  ^
|                  |
multisig    optional combined
sigil       signature message

<message> ::= <varbytes>

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

The Multisig format allows tools that don't recognize the signing codec--or any
of this format other than varuint and varbytes--to know how many octets are in
the Multisig data object and skip over it. This format is also designed to
support any kind of digital signature, even signatures with multiple signature
payloads such as threshold signatures. This also supports building combined
signatures that contain the signed data in the `message` part of the signature.

### [Signature Message](#signature-message)

In a Multisig object, the message is optional. When present, the Multisig is 
called a "combined" signature. When it isn't present, this is a "detached" 
signature. Either way the message that is signed, as well as the verifying
Multikey, are required to verify the validity of a Multisig signature.

### [Attributes](#attributes)

**SigData (0x00)**
: The signature data.

**PayloadEncoding (0x01)**
: The sigil specifying the encoding of the signed message.

**Scheme (0x02)**
: The threshold signing scheme.

**Threshold (0x03)**
: The minumum number of signature shares required to reconstruct the signature.

**Limit (0x04)**
: The total number of shares for a threshold signature.

**ShareIdentifier (0x05)**
: The identifier for the signature share.

**ThresholdData (0x06)**
: Codec-speicific threshold signature data. This is typically used to 
accumulate threshold signature shares.

[0]: https://cryptid.tech
[1]: https://github.com/cryptidtech/provenance-specifications/
[2]: https://github.com/multiformats/multiformats
[3]: https://github.com/CommunitySpecification/1.0
[4]: https://github.com/cryptidtech/provenance-specifications
[5]: https://github.com/multiformats/unsigned-varint
[6]: https://github.com/multiformats/multicodecs/blob/master/table.csv
