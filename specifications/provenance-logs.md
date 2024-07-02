# Provenance Logs Specification
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
   1. [Features](#features)
   2. [Entry](#entry)
   3. [Virtual Name Space](#virtual-name-space)
   4. [Hierarchical Keys](#hierarchical-keys)
   5. [Values](#provenance-values)
   6. [Mutation Operations](#mutation-operations)
   7. [Unlock and Lock Scripts](#unlock-and-lock-scripts)
      1. [Validating a Proposed Entry](#validating-a-proposed-entry)
      2. [Unlock and Lock Script Functions](#unlock-and-lock-script-functions)
   8. [Forking Provenance Logs](#forking-provenance-logs)
      1. [Example Forking Using Delegation](#example-forking-using-delegation)

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

This specification describes the specification for a hash-linked data structure 
that is a cryptographically verifiable provenance log. 
Provenance logs are constructed as a singly-linked list of entries that contain 
an arbitrarily long, ordered list of mutable operations to a virtual key-value 
store as well as a set of cryptographic puzzles that must be solved by the next entry in the list for it to remain valid. 
This design ensures delegatable, recoverable and revocable write control over the log.

### [Current Status](#current-status)

Provenance Logs are fully defined by this specification and meets all of the needs
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

This document refers to the `wacc` VM API. The normative reference for which can
be found in the [wacc_api_specification][8].

This document refers to `vlad`s. The normative reference for which can be found
in the [vlad specification][9].

This document refers to `cid`s. The normative reference for which can be found
in the [ipfs_content_identifier specification][https://docs.ipfs.tech/concepts/content-addressing/].

### [Terms and Definitions](#terms-and-definitions)

**DAG**
: A directed acyclic graph. 

**LipmaaLinks**
: A cryptographically secure hash of some older 
entry in the log, chosen such that there are 
short paths between any pair of entries.

**CBOR**
: Concise Binary Object Representation that is a self-describing
binary data serialization format.

**Noop**
: No operation

**Seqno**
: A sequence number represented as a positive integer.

## [Specification](#specification)

### [Features](#features)
Provenance Logs support the following list of features:

* Sequence numbering for detecting forks and erasures.
* [Lipmaa][LIPMAA] links for O(log N) path lengths between any two entries in a
  log.
* Forking capability to create child logs with back links to the parent log
  forming a DAG of logs.
* Stable, non-key-material identifier to scope/namespace each branch in the
  DAG of logs.
* [WACC VM][8] verification script for the next entry is stored inline or
  externally in a content-addressable storage.
* Serialization to/from DAG-CBOR for automated retrieval of the entire log.

### [Entry](#entry)

Each entry contains the entry's version, the VLAD identifier,
the CID for the previous entry, the CID of the lipmaa predecessor,
the entry's sequence number, the list of mutation operations, the lock
scripts for validating the next entry, the unlock script used for validating
this entry and the proof data for validating this entry.

### [Virtual Name Space](#virtual-name-space)

Provenance logs are a sequence of entries that each contain an ordered list
of mutation operations applied to a virtual key-value pair store. There are
three different possible operations: `update`, `delete`, and `noop`. `update` 
creates/modifies the value associated with a key-path. `delete`
removes a key-value pair. `noop` does nothing, but does
"touch" the associated key-path and is important for the lock script execution
system. As each entry in the log is verified and processed, the mutations
contained within are applied to the virtual key-value pair storage. These pairs
have no intrinsic meaning. It is expected that they will have meaning in the
context of the [WACC VM][8] lock and unlock scripts as well as in the
context of applications consuming the logs.

### [Hierarchical Keys](#hierarchical-keys)

The keys in the virtual key-value pair system are [UTF-8](https://www.w3schools.com/charsets/ref_html_utf8.asp) strings of printable
characters and are hierarchical in nature. They work similarly to file paths in
Linux in that they use the forward slash `/` character as the separator. All
keys must begin with the `/` character. Multiple `/` characters together are
invalid. Key-paths that end with a `/` are called branch key-paths and
key-paths that do not are called leaf key-paths. It is correct to mentally
think of branches like filesystem folders and leaves like files inside folders.
A branch can contain other branches and/or leaves.

### [Values](#provenance-values)

The values in the key-value pair store are self describing and can take one of
three values: *nil*, *str*, or *data*. *nil* is an empty value. *str*
is a UTF-8 text string. *data* is a binary blob.

### [Mutation Operations](#mutation-operations)

Let's assume the first entry in a provenance log includes the following
mutation ops:

```json
"ops": [
    { "noop": [ "/" ] },
    { "update": [ "/name", { "str": [ "foo" ] } ] },
    { "update": [ "/move", { "str": [ "zig" ] } ] },
    { "delete": [ "/zig" ] },
]
```

After validating the first entry and applying its mutations, the virtual
key-value store contains the following:

```json
{
    "/name": "foo",
    "/move": "zig",
}
```

Then, assume the second entry contains the following mutation ops:

```json
"ops": [
    { "update": [ "/name", { "str": [ "bar" ] } ] },
    { "delete": [ "/answer" ] },
    { "update": [ "/move", { "str": [ "zig" ] } ] },
]
```

After validating the second entry and applying its mutations, the virtual
key-value store contains the following:

```json
{
    "/name": "bar",
    "/move": "zig"
}
```

## [Unlock and Lock Scripts](#unlock-and-lock-scripts)

Each entry contains a list of lock scripts and a single unlock script. The set
of lock scripts define the conditions which must be met by the next entry in
the log for it to be valid. The unlock script is the solution to one of the
lock scripts in the previous entry. All scripts are wasm code that
executes inside a [WACC VM][8]. When a script is executed, it is expected
to reference data in a key-value store when executing. Along with the virtual
key-value store built up from the op (operations) in each event, the scripts may also
reference the fields in the event they are a part of (e.g. vlad, seqno, etc).
Scripts can only be unlocked using data associated with their corresponding event.
Lock scripts may reference all data in the key-value
store as well as the data in the entry which it is a part of.

### [Validating a Proposed Entry](#validating-a-proposed-entry)

It is critical to the security of the log that the validation of each entry be
done following a very specific order of operations:

1. Create two empty key-value stores.
2. Load the proposed entry fields (e.g. seqno, vlad, proof, etc) into one
   key-value store to establish the proposed state. DO NOT apply the `ops`
   mutations from the proposed entry.
3. Load the unlock script from the proposed entry and execute it to initialize
   the stack to be used in the lock script execution.
4. Apply the mutations for each entry in the log, from the foot (first) to the
   head (last), to the other key-value store to establish the current state of
   the log for the lock script execution.
5. Execute each lock script from the current entry in root to leaf order. For
   each lock script, clone stack from step 2 and clone the key-value pair store
   from step 3 and execute it.
6. Check the top value on the return stack for a SUCCESS value, all other
   values indicate a failure.

### [Unlock and Lock Script Functions](#unlock-and-lock-script-functions)

The following functions are available for use in lock and unlock scripts:

**push(<key-path>)**
: Pushes the value associated with the key-path onto the parameter stack.

**pop()**
: Pops the top value off of the parameter stack.

**branch(<key-path>)**
: Concatenates the branch key-path with the provided key-path to create a
key-path argument for other functions. When used in lock scripts, the branch
key-path is the key-path the lock script is associated with. When used in
unlock scripts, the branch key-path is always `/`. This function fails if
used in a lock script associated with a leaf.

**check_eq(<key-path>)**
: Compares the value associated with the key-path with the value on top of the
parameter stack; increments the check counter if it fails. Pops one parameter
off of the parameter stack if it succeeds.

**check_preimage(<key-path>)**
: Compares the hash associated with the key-path with the hash of the value on
top of the parameter stack; increments the check counter if it fails. Pops
one parameter off of the parameter stack if it succeeds. This function
understands the [Multihash][10] format and is able to generate and
verify preimages using any hash function it supports.

**check_signature(<key-path>)**
: Verifies the digital signature and message on the parameter stack using the
public key associated with the key-path; increments the check counter if it
fails. Pops two parameters off of the parameter stack if it succeeds. This
function understands the [Multikey][7] and [Multisig][6]
formats and is able to verify any digital signature they support.

### [Unlock Scripts](#unlock-scripts)

Unlock scripts are code that compiles to wasm and executed in the [WACC
VM][8]. For historical reasons, each unlock script is a wasm module with a
single exported function called "for_great_justice".

The unlock script in a proposed new entry references the proof and the other
entry values to provide a solution to one of the lock scripts in the current
head entry of the provenance log. By convention, when an unlock script executes
the key-value store has the following keys populated with the data from the
proposed entry like so:

```
─┬─"/"
 ╰─┬─ "entry/"
   ╰─┬─ "version"
     ├─ "vlad"
     ├─ "prev"
     ├─ "lipmaa"
     ├─ "seqno"
     ├─ "ops"
     ├─ "locks"
     ├─ "unlock"
     ╰─ "proof"
```

An example unlock script for satisfying a `check_signature` lock script looks
like:

```rust
#[no_mangle]
pub fn for_great_justice() {
    // push the serialized Entry as the message
    push("/entry/");

    // push the proof data
    push("/entry/proof");
}
```

or in web assembly text:

```wat
(module
  ;; the imported _check_signature function
  (import "wacc" "_push" (func $push (param i32 i32) (result i32)))

  ;; the exported "for_great_justice" function to call
  (func $main (export "for_great_justice") (param) (result i32)
    ;; push("/entry/")
    i32.const 0 
    i32.const 7
    call $push

    ;; push("/entry/proof")
    i32.const 7
    i32.const 12
    call $push 

    return
  )

  ;; define the memory to store the string constants in
  (memory (export "memory") 1)

  ;; string constant to pass to check_signature
  (data (i32.const 0) "/entry/")
  (data (i32.const 7) "/entry/proof")
)
```

In the example given above, the unlock script pushes the value associated with
the `"/entry/"` key-path which is the compact serialized form of the proposed
entry with a NULL `"/entry/proof"` value. In essence, the value associated with
`"/entry/"` is the serialized entry object that was digitally signed; it is the
message associated with the digital signature associated with the
`"/entry/proof"` key-path. NOTE: this assumes a detached signature but it is
also possible to use a combined signature that contains the signed data,
eliminating the need to push the `"/entry/"` value however this unnecessarily
duplicates data.

### [Lock Scripts](#lock-scripts)

Lock scripts are code that compiles to wasm and executed in the [WACC
VM][8]. For historical reasons, each lock script is a wasm module with
a single exported function called "move_every_zig".

Every entry has a list of key-paths and the associated lock scripts that are
used to validate events with mutations to those parts of the key-value pair
store. Below is an example of what the list of lock scripts looks like:

```json
"locks": [
  [ "/", "<root lock script>" ],
  [ "/delegated/mike/", "<mike lock script>"],
  [ "/delegated/walker/", "<walker lock script>"],
  [ "/delegated/dave/", "<dave lock script>"],
]
```

The [WACC VM][8] execution environment exposes a number of cryptographic
functions to lock and unlock scripts outlined above. To ensure maximum safety,
no lock script has direct access to any of the data in events or the key-value
pair store. Instead, the functions take key-paths as references to data as the
parameters to functions. You've already seen in the examples above how the
`push` function takes a key-path as its argument and the associated value is
pushed onto the parameter stack.

Each lock script has a key-path that it governs and there may be multiple lock
scripts for each key-path that get executed in the order in which they are
listed in the log event's lock scripts list. If a branch contains
branches/leaves that do not have a lock script associated with them then the
parent branch lock script also governs them and validates any subsequent events
that mutate those parts of the key-value store.

When a lock script is executed the "context" key-path that is available to the
`branch` function is the longest common key-path in the set of mutation `op`
values in the proposed event.

#### Calculating the Context Key-Path

As an example let us assume the proposed event has the following `op` entries:

```json
"ops": [
    { "update": [ "/forks/001/foo", { "str": [ "bar" ] } ] },
    { "update": [ "/forks/001/move", { "str": [ "zig" ] } ] },
]
```

The "context" key-path is the longest common *branch* key-path in the set of
`op` key-paths. In the example above, the longest common branch key-path is
`"/forks/001/"` so in any lock script that runs to validate this entry, the
`branch` function would concatenate `"/forks/001/"` with the key-path passed
in. For instance, a call to `branch("foo")` would result in the construction of
the `"/forks/001/foo"` key-path.

Now, let us assume the set of mutation `op` values in the proposed event is
the following:

```json
"ops": [
    { "update": [ "/forks/001/foo", { "str": [ "bar" ] } ] },
    { "noop": [ "/forks/" ] },
    { "update": [ "/forks/001/move", { "str": [ "zig" ] } ] },
]
```

The longest common branch key-path is `"/forks/"` because of the `noop`
mutation value. This demonstrates the use of the `noop` to "scope" the context
of the proposed events and affect the context key-path for the lock script
execution. NOTE: the `noop` operation is only useful for making the context
key-path closer to the root `"/"` key-path and cannot make it longer. This
effectively elevates the level of proof required to make the proposed event
valid assuming that the closer to the root key-path the context gets, the
closer to the owner—and thus less delegated—the proof capabilities get.

#### [Lock Script Execution Order](#lock-script-execution-order)

Here is another example to clarify which lock scripts get executed and in
which order. Assume you have a key-path structure like the following:

```
─┬─ "/"
 ╰─┬─ "tpubkey"
   ├─ "pubkey"
   ├─ "hash"
   ╰─┬─ "delegated/"
     ├─┬─ "mike/"
     │ ╰─── "endpoint"
     ├─┬─ "walker/"
     │ ╰─── "peerid"
     ╰─┬─  "dave/"
       ╰─── "endpoint"
```

And also assume the current entry has the following list of lock scripts
associated with each key-path:

```json
[
  [ "/", "<root lock script>" ],
  [ "/delegated/mike/", "<mike lock script>"],
  [ "/delegated/walker/", "<walker lock script>"],
  [ "/delegated/dave/", "<dave lock script>"],
]
```

If a proposed entry only contains an `op` to `update("/foo")` then the only
lock script that gets executed is the one associated with the `"/"` key-path
because `"/foo"` is in the `"/"` branch. The context key-path in that case is
just `"/"` because that is the longest common key-path in the `op` set.

If the proposed entry only contains an `op` to
`update("/delegated/mike/endpoint")` then the lock script for `"/"` runs, and if
it fails then the lock script for `"/delegated/mike/"` runs because
`"/delegated/mike/endpoint"` is in the `"/delegated/mike/"` branch. In both cases
the context key-path is `"/delegated/mike/"` since that is the longest common
branch key-path in the `op` set.

Lock scripts associated with branches closer to the root branch execute first
and if they succeed, no further lock scripts are executed. This allows lock
scripts closer to the root branch to override the lock scripts closer to the
leaves; as it should be. If the provenance log owner wishes to override an
update made to a delegated branch/leaf, their proof takes precedence as long as
it causes a lock script closer to the root branch to succeed. So in this
example, the proposed next entry with proof that satisfies the `"/"` branch
lock script will take precedence over a proposed next entry with a proof that
only satisfies the `"/delegated/mike/"` lock script.

#### [Example Lock Script](#example-lock-script)

Assume that applying the mutation ops from foot (i.e. first event) to head
(i.e. most recent) creates the following key-value store state:

```
─┬─ "/"
 ╰─── "pubkey"
```

A lock script may reference the value of the public key like so:

```rust
#[no_mangle]
pub fn move_every_zig() -> bool {
    // check the signature using the data on the stack and the pubkey
    check_signature("/pubkey")
}
```

or in web assembly text:

```wat
(module
  ;; the imported _check_signature function
  (import "wacc" "_check_signature" (func $check_signature (param i32 i32) (result i32)))

  ;; the exported "move_every_zig" function to call
  (func $main (export "move_every_zig") (param) (result i32)
    i32.const 0 
    i32.const 7 
    call $check_signature 
    return
  )

  ;; define the memory to store the string constants in
  (memory (export "memory") 1)

  ;; string constant to pass to check_signature
  (data (i32.const 0) "/pubkey")
)
```

Since lock scripts are associated with a key-path which specifies when/if it is
executed, the above lock script will always reference the public key associated
with the `"/pubkey"` key-path regardless of which key-path it is associated
with because it uses an absolute key-path. The above lock script could also be
rewritten to the following:

```rust
#[no_mangle]
pub fn move_every_zig() -> bool {
    // check the signature using the pubkey in the branch context 
    check_signature(branch("pubkey"))
}
```

This lock script could be associated with the `"/"` key-path or any other
key-path such as `"/foo/"`. In the latter case, the call to `check_signature`
would be passed the key-path `/foo/pubkey`. This is handy for use in delegation
which is explained further down in this document.

The lock script above executes the `check_signature` function passing it the
key-path referencing the public key to use. The function first peeks at the
value on top of the stack to see if there is a [Multisig][6] encoded
digital signature on the top of the stack and checks that it is the same kind
of signature as the public key. If those match, then the function expects the
stack to have the digital signature on top with the message just below that. It
checks the signature, and if it is valid, it pops both the signature and
message off of the parameter stack and it pushes the `SUCCESS` marker onto the
return stack.

Lock scripts may themselves have multiple checks that are combined with logical
"and" and "or" operations. In those cases, precedence is established by the
"check counter". The check counter starts at zero and is incremented every time
a `check_*` function is called and fails. When two competing proposed entries
satisfy the same lock script, the one with the _lowest_ check counter value
takes precedence.

Below is an example of the use of "and" and "or" predicates to construct a lock
script that enforces the proposed entry proof is either a valid threshold
signature or pubkey signature or preimage proof.

```rust
#[no_mangle]
pub fn move_every_zig() -> bool {
    // then check a possible threshold sig...
    check_signature("/tpubkey") ||   // check_count++
    // then check a possible pubkey sig...
    check_signature("/pubkey") || // check_count++
    // then the pre-image proof...
    check_preimage("/hash")
}
```

Here, there are three `check_*` function calls. To enforce precedence in the
checks, every time a `check_*` function is executed and it fails the check
counter is incremented in the WACC VM. If the lock script succeeds, the check
counter value is pushed onto the stack as the payload of the
`SUCCESS(check_counter)` marker.

Because logical operations in Rust short circuit, in the example lock script
above, the only time the `check_preimage` function will execute is if the
`check_signature("/pubkey")` fails. The only time the
`check_signature("/pubkey")` function will execute is if the
`check_signature("/tpubkey")` function fails. If either `check_signature`
functions fail, their only side effect is to increment the check counter. Then
if the `check_preimage` succeeds, the marker left on the stack will be
`SUCCESS(2)`. If the `check_signature("/pubkey")` succeeds, the marker left on
the stack will be `SUCCESS(1)`. If the `check_signature("/tpubkey")` succeeds,
the marker left on the stack will be `SUCCESS(0)`. If there are two competing
entries and one provides a valid signature and the other provides a valid
preimage, the one with the valid signature takes precedence over the one with
the valid preimage because the check counter is lower for the entry with the
valid signature.

In the case where competing entries satisfy the same lock script with the same
check count, the entry with the context key-path closest to the `"/"` branch
takes precedence. The solution for resolving a tie is for entry creators to
generate proof with a lower check count or proof that satisfies a lock script
with higher precedence. Single clause lock scripts are a bad idea for this
reason; they leave the door open for unresolvable ties in precedence.

#### [Delegation](#delegation)

The lock script mechanism is designed to support delegation. By adding lock
scripts associated with branches/leaves that can be satisfied by proofs
generated by 3rd parties, the owner of a log is delegating the management of
those branches/leaves to those people/services. The precedence and check
counter mechanism is designed to resolve any conflicts between competing
entries, giving a hierarchy of who can override whom when updating a log.

In the example from above, let us assume we have the following:

```
─┬─ "/"
 ╰─┬─ "tpubkey"
   ├─ "pubkey"
   ├─ "hash"
   ╰─┬─ "delegated/"
     ├─┬─ "mike/"
     │ ├─── "pubkey"
     │ ╰─── "endpoint"
     ╰─┬─ "walker/"
       ├─── "pubkey"
       ╰─── "peerid"

```

And also assume the current entry has the following list of lock scripts
associated with each key-path:

```json
[
  [ "/", "<root lock script>" ],
  [ "/delegated/", "<delegation lock script>"],
]
```

Taking advantage of the `branch` function, we only need a single lock script to
govern all of the `"/delegated/"` branch:

```rust
#[no_mangle]
fn lock() -> bool {
    check_signature(branch("pubkey"))
}
```

If Mike wished to update the value associated with the
`"/delegated/mike/endpoint"`, he would create a new Entry with a single `op`:

```json
"ops": [
    { "update": [ "/delegated/mike/endpoint", { "str": [ "https://cryptid.tech" ] } ] },
]
```

When validating his proposed entry for my log, first the `"/"` lock script
would run with the branch context of `"/"` and it would fail because Mike does
not possess the capability of creating proof (e.g. a digital signature) over
his entry that validates with the `"/"` lock script. After that fails, the lock
script for the `"/delegated/"` branch executes with the `"/delegated/mike/"`
branch context. If Mike's proposed entry is digitally signed with the secret
key that is associated with the public key value stored under
`"/delegated/mike/pubkey"` then the entry validates and is accepted.

If Walker wanted to update the value associated with
`"/delegated/walker/peerid"` then he would also create a new proposed event and
digitally sign it with the key pair associated with the public key stored under
`"/delegate/walker/pubkey"` and the same `"/delegated/"` lock script works for
him because the context branch for validating his event is
`"/delegated/walker/"`.

#### [Forced recovery using precedence](#forced-recovery-using-precedence)

The purpose of the precedence is to create lock scripts with the hardest to
hack and most secure checks as the top precedences with less secure checks
having lower precedence. Today, the most secure kind of check is a threshold
signature generated through the cooperation of multiple independent
people/services using a quantum-resistant one-time signature schemes such as a
threshold Lamport signature. If the lock scripts have a threshold check that
takes precedence over a public key signature that in turn takes precedence over
a preimage check, then if a password (i.e. preimage) is compromised and the
attacker creates a new entry using a preimage proof, the rightful owner of the
log can then create a competing entry with the same sequence number using a
signature proof. The log "protocol" demands that the higher precedence entry
to be chosen as the next valid entry and not the one created by the attacker
who guessed your password.

Most importantly, if a public key pair is compromised and an attacker attempts
a takeover by creating an entry using a signature proof, the rightful owner can
then contact their friends—or a threshold recovery service—and have them create
a threshold signature over a new entry that takes precedence over the
attacker's entry. The new entry with the threshold signature can then contain a
key rotation and key revocation mutation for the virtual key-value store
marking the compromised key as revoked as well as establishing a new valid
public keypair all without breaking the chain of trust in the log.

You can think of this recovery scenario as a digital "social recovery" where
your friends collaborate to generate the threshold signature to recover your
control over your log. This can be applied in a number of real-world scenarios
such as board control over a corporation's identity log, heirs collaborating
with the deceased's counsel to take over the deceased's logs, or even parents
co-signing with their minor children to update the children's log for some
reason.

One especially useful application is empowering maintainers of a Git repository
with full identity and access management (IAM) capabilities. By requiring that
the `"/"` branch lock script have a `check_signature("/maintainers")` threshold
signature check as the highest precedence, then the threshold group associated
with the "maintainers" threshold public key has full control over every
key-value pair in the provenance log. Maintainers can force rotate to a new
signing key for a contributor to recover to a known-good key. They can also
force rotate a contributor's signing key to NULL and remove their ability to
contribute to the repository. They can delegate branches and leaves in
contributors' provenance logs to enable a repository-wide service to update the
contributors' logs as needed. In short, a design like this gives the
maintainers of the repository full control over the IAM for contributors to the
repository all without a centralized server/service such as Github or Gitlab.

## [Forking Provenance Logs](#forking-provenance-logs)

It is possible to fork provenance logs by into any number of child logs. Child
logs always start with an entry that has a sequence number of `0` and a prev
CID that points at the entry in the parent log from which the child log is
forked. The lock script in the parent log entry is used as the lock script to
validate the first entry in the child log. By convention, along with the prev
CID pointing to the parent, the first entry in the child log must also contain
a mutation `op` that updates the VLAD of the parent to the `"/parent/"`
key-path. The parent VLAD is useful for when the parent log moves from one
content addressable storage to another. As long as a VLAD to CID mapping record
exists somewhere, then the CID can be resolved to the correct entry in the
parent log.

### [Example Forking Using Delegation](#example-forking-using-delegation)

Typically, the parent log maintains information about its child forks under the
`"/forks/"` branch. Using the delegation and branch mechanisms, it is trivial
to manage forking using a separate lock script assigned to the `"/forks/"`
key-path:

```rust
#[no_mangle]
pub fn move_every_zig() -> bool {
    // forking the parent be done by whomever can sign with "/forks/pubkey"
    check_signature(branch("pubkey")) ||

    // check the validity of the first entry of the child log 
    (check_eq(branch("vlad")) && check_signature(branch("pubkey")))
}
```

The process of forking a provenance log takes two steps. The first step is
recording the information about the child log in the parent log and the
second step is publishing the child log. By convention, each child log's
information is stored under its own branch under `"/forks/"`. The child's
branch name is arbitrary but, again by convention, under its branch there is a
`"vlad"` leaf with the child's VLAD, a `"pubkey"` leaf with the child's first
advertised pubkey that was also used to sign its first entry and the CID in its
VLAD.

When creating a child log fork, we create a new entry in the parent log with
the following mutation `op` values:

```json
"ops": [
    { "noop": [ "/forks/" ] },
    { "update": [ "/forks/child1/vlad", { "bin": [ <binary child VLAD data> ] } ] },
    { "update": [ "/forks/child1/pubkey", { "bin": [ <binary child multikey pubkey data> ] } ] },
]
```

This new entry is signed using the key pair associated with `"/forks/pubkey"`.
The `noop` mutation is used to set the context key-path to `"/forks/"` by
including it the `op` mutation set of key-paths. It is the longest common
key-path and thus becomes the context key-path for the `branch` function when
running it to validate the new event in the parent log.

When the lock script associated with `"/forks/"` executes, the first
`check_signature` call passes and the lock script exits with `SUCCESS(0)` on
the return stack.

Then the first entry of the child log is created with the child VLAD as its
VLAD, the prev CID set to the CID of the parent event we just created. It must
include an `op` mutation to record the parent VLAD in the
`"/forks/child1/parent"` leaf of the child's key-value store:

```json
"ops": [
    { "update": [ "/forks/child1/parent", { "bin": [ <binary parent VLAD data> ] } ] },
    { "update": [ "/forks/child1/pubkey", { "bin": [ <binary child VLAD pubkey> ] } ] },
]
```

The additional `"/forks/child1/pubkey"` in the child log records the signing
key in the child in such a way that it doesn't disrupt the delegation and
forking lock script execution but allows for the child log to have its own
lock scripts and validate the next entry in the log. The only rule is that the
child's initial entry can only have `op` mutations under the `"/forks/child1/"`
key-path to ensure that the lock script from the parent executes correctly and
validates the child's first entry.

The unlock script of the first entry in the child log must be like the
following:

```rust
#[no_mangle]
pub fn unlock() {
    // push the entry values
    push("/entry/");

    // push the signature created using the /forks/child1/pubkey keypair
    push("/entry/proof");

    // push the vlad
    push("/entry/vlad");
}
```

When validating the first entry in the first child log, the unlock script
pushes the serialized entry, the ephemeral signature over the entry as well as
the log's VLAD value. This puts the parameter stack into the following state:

```
      ┌──────────────────┐
top → │ <"/entry/vlad">  │ ← child vlad
      ├──────────────────┤
      │ <"/entry/proof"> │ ← digital signature
      ├──────────────────┤
      │ <"/entry/">      │ ← signed message
      ├──────────────────┤
      │        ┆         │
      ┆                  ┆
```

The first entry of the child log must be signed using the key pair associated
with the pubkey recorded in the parent log under the `"/forks/child1/pubkey"`
leaf. When the first entry is validated, the lock script from the parent
associated with the parent's `"/forks/"` key-path will execute because the
first entry in the child log only mutates key-paths under `"/forks/"`. The
context path this time will be `"/forks/child1/"` since that is the longest
common branch key-path in the `op` mutation set in the child's first entry.

When the parent log entry's lock script executes, the following steps are
executed:

1. The `check_signature(branch("pubkey"))` fails because it sees a VLAD on top
   of the stack and not a signature.
2. The `check_eq(branch("vlad"))` succeeds because the `<"/forks/child1/vlad">`
   value in the parent matches the "VLAD" value pushed on the stack by the
   child unlock script. It pops the VLAD parameter off of the parameter stack.
3. The `check_signature(branch("pubkey"))` succeeds because the
   `<"/forks/child1/pubkey">` public key in the parent validates the signature
   `<"/entry/proof">` and message `<"/entry/">` values pushed on the stack by the
   child unlock script. It pops both off of the parameter stack.
4. A `SUCCESS(1)` marker is pushed onto the results stack because the initial
   `check_signature` function failed, incrementing the check counter once.

The initial entry in the child log should also set up it's own lock scripts
for governing its key-value store. The data recorded under the
`"/forks/child1/"` branch in the child's key-value store can be used with lock
scripts in the child to validate the second entry in the child provenance log
which can then remove the `"/forks/child1/"` branch if so desired.

[0]: https://cryptid.tech 
[1]: https://github.com/cryptidtech/provenance-specifications/
[2]: https://github.com/multiformats/multiformats
[3]: https://github.com/CommunitySpecification/1.0
[4]: https://github.com/bitcoin/bips/blob/master/bip-0342.mediawiki
[5]: https://github.com/cryptidtech/provenance-specifications/blob/main/specifications/nonce.md
[6]: https://github.com/cryptidtech/provenance-specifications/blob/main/specifications/multisig.md
[7]: https://github.com/cryptidtech/provenance-specifications/blob/main/specifications/multikey.md
[8]: https://github.com/cryptidtech/provenance-specifications/blob/main/specifications/wacc.md
[9]: https://github.com/cryptidtech/provenance-specifications/blob/main/specifications/vlad.md
[10]: https://github.com/cryptidtech/multihash.git

[LIPMAA]: https://github.com/AljoschaMeyer/bamboo/blob/master/README.md#links-and-entry-verification