[![](https://img.shields.io/badge/made%20by-Cryptid%20Technologies-gold.svg?style=flat-square)][CRYPTID]
[![](https://img.shields.io/badge/project-provenance-purple.svg?style=flat-square)][PROVENANCE]
[![](https://img.shields.io/badge/project-multiformats-blue.svg?style=flat-square)][MULTIFORMATS]
[![](https://img.shields.io/badge/License-Functional_Source_1.1-red)][FSL]
![](https://github.com/cryptidtech/wacc/actions/workflows/rust.yml/badge.svg)

# Web Assembly Cryptographic Constructs (WACC) VM

This Rust crate is an implementation of the [WACC VM specification][SPEC] that
is a part of the [provenance specifications][PROVENANCE]. It extends a standard
web assembly VM with the set of data manipulation and cryptographic operations
found in the WACC VM specification. Please see the specification Appendix for a
full discussion of the motivation and design principles behind the this
implementation.

## Purpose

In general, the WACC VM is designed to be a way to describe data validation 
scripts similar to the way Bitcoin scripts work when validating a Bitcoin 
transaction. WACC VM scripts operate on fields in a virtual key-value store
with strings as keys. No data is manipulated directly by the WACC scripts, they 
simply push and pop keys to the virtual stack and then execute cryptographic 
operations on the values associated with the keys on the stack and then do 
logic tests on the results.

WACC VM scripts are designed to be very simple to write. The `examples/` folder
contains examples of Rust code compiled as WACC VM scripts as well as web 
assembly text (wast) versions of scripts. Just like Bitcoin script, WACC VM 
scripts are designed to be executed in pairs with an "unlock" script executing
before the "lock" script. The "unlock" script places key values on the virtual
stack, setting up the data necessary for the "lock" script to execute the 
cryptographic operations. The goal is to be able to verify updates to the 
key-value store state.

## Use Cases

### Provenance Logs

The primary use for WACC VM scripts is to provide control over who can append
events to a [provenance log][PLOG]. The scripting is necessary because
provenance logs enforce a proof "precedence" whereby verification proofs of one
kind take precendence over proofs of other kinds. This creates the capabilities
necessary to build useful decentralized identity, intellectual property
management, and general data provenance solutions on top.

[CRYPTID]: https://cryptid.tech/
[PROVENANCE]: https://github.com/cryptidtech/provenance-specifications/
[MULTIFORMATS]: https://github.com/multiformats/multiformats/
[FSL]: https://github.com/cryptidtech/provenance-log/blob/main/LICENSE.md
[SPEC]: https://github.com/cryptidtech/provenance-specifications/blob/main/specifications/wacc.md 
[PLOG]: https://github.com/cryptidtech/provenance-log/
