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
3. [Specification](#specification)
4. [Examples](#examples)
   1. [WACC Logging](#wacc-logging)
   2. [WACC Signature Verification](#wacc-signature-verification)
   3. [WACC Locking Signature Verification](#wacc-locking-signature-verification)
   4. [WACC Unlock Script](#wacc-unlock-script)

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

**Unlock**
: The script that is executed to unlock an operation. Unlock operations
push values onto the WACC VM stack that are satisfy conditions by the lock script.

**Lock**
: The script that is executed to lock an operation. Lock operations
verify values pushed onto the WACC VM stack by the unlock script.
If these values do not satisfy the conditions the operation is not allowed.

## [Specification](#specification)

## [Examples](#examples)

WACC VM is a simple stack-based virtual machine that executes cryptographic
operations in a push - pop manner. Push operations are the unlock script values.
Pop operations are the lock script conditions that must be satisfied using the pushed
values.

### [WACC Logging](#wacc-logging)

This example shows how to use the WACC API that provides a logging function to log messages

[Rust](https://github.com/cryptidtech/wacc/blob/main/examples/log/src/lib.rs)

### [WACC Signature Verification](#wacc-signature-verification)

This example shows how to use the WACC API to verify a signature first

[Rust](https://github.com/cryptidtech/wacc/blob/main/examples/signature_first/src/lib.rs)

### [WACC Locking Signature Verification](#wacc-locking-signature-verification)

This example shows how to use the WACC API to verify a signature using a lock script

[Rust](https://github.com/cryptidtech/wacc/blob/main/examples/signature_lock/src/lib.rs)

### [WACC Unlock Script](#wacc-unlock-script)

This example shows how to use the WACC API to use the unlock script

[Rust](https://github.com/cryptidtech/wacc/blob/main/examples/unlock/src/lib.rs)


[0]: https://cryptid.tech 
[1]: https://github.com/cryptidtech/provenance-specifications/
[2]: https://github.com/multiformats/multiformats
[3]: https://github.com/CommunitySpecification/1.0
[4]: https://github.com/bitcoin/bips/blob/master/bip-0342.mediawiki
[5]: https://github.com/cryptidtech/provenance-specifications/blob/main/specifications/nonce.md
[6]: https://github.com/cryptidtech/provenance-specifications/blob/main/specifications/multisig.md
[7]: https://github.com/cryptidtech/provenance-specifications/blob/main/specifications/multikey.md
