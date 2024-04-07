# Wasm Cryptographic Constructs (WACC) VM API Specification
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

This specification describes the API specification for Wasm VM implementations
designed for executing cryptographic operations such as data validation and
proof verification. It takes its inspiration from the [Bitcoin script
specification][4] in that there is an argument stack and the scripts execute 
cryptographic functions that operate on the stack.

### [Current Status](#current-status)

The WACC API is fully defined by this specification and meets all of the needs
of the current set of use cases. If there are new functions needed to meet new
use cases please file a proposal as an issue in this repository.

### [Normative References](#normative-references)

This document refers to `nonce` values. The normative reference for which can
be found in the [multiformats nonce specification][5]

This document refers to `multisig` encoded digital signatures. The normative
reference for which can be found in the [multiformats multisig
specification][6].

This document refers to `multikey` encoded cryptography keys. The normative
reference for which can be found in the [multiformats multikey
specification][7].

### [Terms and Definitions](#terms-and-definitions)



[0]: https://cryptid.tech 
[1]: https://github.com/cryptidtech/provenance-specifications/
[2]: https://github.com/multiformats/multiformats
[3]: https://github.com/CommunitySpecification/1.0
[4]: https://github.com/bitcoin/bips/blob/master/bip-0342.mediawiki
[5]: https://github.com/cryptidtech/provenance-specifications/blob/main/specifications/nonce.md
[6]: https://github.com/cryptidtech/provenance-specifications/blob/main/specifications/multisig.md
[7]: https://github.com/cryptidtech/provenance-specifications/blob/main/specifications/multikey.md
