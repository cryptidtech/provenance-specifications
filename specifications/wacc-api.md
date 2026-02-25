# WACC VM API Specification (Complete)

**Version:** 1.0.5
**Status:** Draft
**Implementation:** bettersign/crates/wacc
© 2024 Cryptid Technologies, Inc.

## Overview

This document provides the complete API specification for the Web Assembly Cryptographic Constructs (WACC) VM. The WACC VM is a stack-based virtual machine designed for executing cryptographic verification scripts in provenance systems.

## Architecture

### Execution Model

The WACC VM uses a dual-stack architecture:
- **Parameter Stack (pstack)**: Arguments for operations
- **Return Stack (rstack)**: Results and status values

### State Model

The VM operates on two key-value stores:
- **Current State**: Immutable, contains committed data
- **Proposed State**: Immutable, contains data to be verified

### Context Path

Operations can be scoped using hierarchical context paths (e.g., `/root/child/grandchild`).

## API Functions

All WACC API functions are imported from the "wacc" module and prefixed with underscore.

### 1. _push

**Signature:**
```wasm
(import "wacc" "_push" (func $push (param i32 i32) (result i32)))
```

**Parameters:**
- `key_ptr` (i32): Pointer to key string in linear memory
- `key_len` (i32): Length of key string

**Returns:**
- i32: 1 on success, 0 on failure

**Operation:**
1. Reads key string from linear memory at `[key_ptr, key_ptr + key_len)`
2. Looks up value in current state using the key
3. Pushes value onto parameter stack
4. Returns success/failure

**Errors:**
- Missing key in current state
- Invalid memory access
- Stack overflow

**Example:**
```wasm
;; Push value at key "/pubkey" onto stack
(call $push
  (i32.const 0)    ;; pointer to "/pubkey" string
  (i32.const 7))   ;; length of "/pubkey"
```

### 2. _pop

**Signature:**
```wasm
(import "wacc" "_pop" (func $pop (result i32)))
```

**Parameters:** None

**Returns:**
- i32: 1 on success, 0 on failure

**Operation:**
1. Removes top value from parameter stack
2. Discards the value
3. Returns success/failure

**Errors:**
- Empty parameter stack

### 3. _check_eq

**Signature:**
```wasm
(import "wacc" "_check_eq" (func $check_eq (param i32 i32) (result i32)))
```

**Parameters:**
- `key_ptr` (i32): Pointer to key string
- `key_len` (i32): Length of key string

**Returns:**
- i32: 1 on success, 0 on failure

**Operation:**
1. Reads key from memory
2. Looks up expected value in current state
3. Peeks at top of parameter stack
4. Compares values for equality (constant-time)
5. If equal: pops stack, increments check_count, returns success
6. If not equal: increments check_count, returns failure

**Errors:**
- Missing key in current state
- Empty parameter stack
- Type mismatch
- Maximum check count exceeded

### 4. _check_preimage

**Signature:**
```wasm
(import "wacc" "_check_preimage" (func $check_preimage (param i32 i32) (result i32)))
```

**Parameters:**
- `key_ptr` (i32): Pointer to key string
- `key_len` (i32): Length of key string

**Returns:**
- i32: 1 on success, 0 on failure

**Operation:**
1. Reads key from memory
2. Looks up multihash from current state at key
3. Validates hash algorithm is on security whitelist
4. Peeks at preimage data from top of parameter stack
5. Computes hash of preimage using algorithm from stored hash
6. Compares computed hash with stored hash
7. If match: pops preimage from stack, returns success
8. If no match: increments check_count, returns failure

**Errors:**
- Missing key in current state
- Empty parameter stack
- Invalid multihash format
- Disallowed hash algorithm
- Hash mismatch
- Maximum check count exceeded

**Security:**
- Only whitelisted hash algorithms are accepted
- Weak algorithms (MD5, SHA-1) are rejected

### 5. _check_signature

**Signature:**
```wasm
(import "wacc" "_check_signature" (func $check_signature (param i32 i32 i32 i32) (result i32)))
```

**Parameters:**
- `key_ptr` (i32): Pointer to public key identifier string
- `key_len` (i32): Length of public key identifier
- `msg_ptr` (i32): Pointer to message identifier string
- `msg_len` (i32): Length of message identifier

**Returns:**
- i32: 1 on success, 0 on failure

**Operation:**
1. Reads public key identifier from memory
2. Looks up multikey from current state
3. Reads message identifier from memory
4. Looks up message from proposed state
5. Peeks at multisig from top of parameter stack
6. Verifies signature using public key and message
7. If valid: pops signature from stack, returns success
8. If invalid: increments check_count, returns failure

**Errors:**
- Missing public key in current state
- Missing message in proposed state
- Empty parameter stack
- Invalid multikey format
- Invalid multisig format
- Signature verification failed
- Maximum check count exceeded

**Security:**
- Only whitelisted signature algorithms accepted
- Supports Ed25519, secp256k1, P-256, BLS12-381

### 6. _branch

**Signature:**
```wasm
(import "wacc" "_branch" (func $branch (param i32 i32) (result i32 i32)))
```

**Parameters:**
- `key_ptr` (i32): Pointer to branch segment string
- `key_len` (i32): Length of branch segment

**Returns:**
- i32: Pointer to resulting path string in linear memory
- i32: Length of resulting path string

**Operation:**
1. Reads branch segment from memory
2. Concatenates current context path with segment
3. Writes result to linear memory (top-down allocation)
4. Returns pointer and length

**Errors:**
- Invalid memory access
- Path too long (>1024 chars)
- Path too deep (>32 levels)

**Usage:**
Used to create hierarchical key paths like `/root/child/key`.

### 7. _log

**Signature:**
```wasm
(import "wacc" "_log" (func $log (param i32 i32) (result i32)))
```

**Parameters:**
- `msg_ptr` (i32): Pointer to log message string
- `msg_len` (i32): Length of message

**Returns:**
- i32: 1 on success, 0 on failure

**Operation:**
1. Reads message string from memory
2. Appends message to log buffer with newline
3. Returns success/failure

**Errors:**
- Invalid memory access
- Log buffer full (>64 KB)

**Usage:**
Debug and audit logging from WASM scripts.

### 8. _push_value

**Signature:**
```wasm
(import "wacc" "_push_value" (func $push_value (param i32 i32) (result i32)))
```

**Parameters:**
- `offset` (i32): Pointer to raw bytes in linear memory
- `len` (i32): Length of the byte data

**Returns:**
- i32: 1 on success, 0 on failure

**Operation:**
1. Reads raw bytes from linear memory at `[offset, offset + len)`
2. Pushes the bytes as a binary value onto the parameter stack (with an empty hint)
3. Returns success/failure

**Errors:**
- Invalid memory access
- Stack overflow

**Usage:**
Used by unlock scripts to push raw byte data (such as a public key or preimage) directly from WASM memory onto the parameter stack, without requiring a key-value store lookup. This is the counterpart to `_push` which reads from the current state store.

**Example:**
```wasm
;; Push 32 raw bytes from memory offset 0 onto the parameter stack
(call $push_value
  (i32.const 0)    ;; pointer to bytes in memory
  (i32.const 32))  ;; length of bytes
```

### 9. _check_preimage_value

**Signature:**
```wasm
(import "wacc" "_check_preimage_value" (func $check_preimage_value (result i32)))
```

**Parameters:** None

**Returns:**
- i32: 1 on success, 0 on failure

**Operation:**
1. Pops the expected hash (as a multihash) from the top of the parameter stack
2. Validates the hash algorithm is on the security whitelist
3. Pops the preimage data from the parameter stack
4. Computes the hash of the preimage using the algorithm from the expected hash
5. Compares the computed hash with the expected hash
6. If match: returns success
7. If no match: increments check_count, returns failure

**Errors:**
- Fewer than 2 values on parameter stack
- Top of stack is not a valid multihash
- Disallowed hash algorithm
- Hash mismatch
- Maximum check count exceeded

**Security:**
- Only whitelisted hash algorithms are accepted (same whitelist as `_check_preimage`)
- Weak algorithms (MD5, SHA-1) are rejected

**Usage:**
Used by lock scripts to verify a preimage relationship where both the expected hash and the preimage are provided entirely on the stack (pushed by `_push_value` or `_push`), rather than looking up the expected hash from the current state store. This is useful for the first-lock script which verifies that the advertised public key hashes to the value baked into the WASM binary.

**Stack layout before call:**
```
  top:    [expected multihash]   <- popped first
  next:   [preimage data]        <- popped second
```

**Comparison with `_check_preimage`:**
- `_check_preimage(key_ptr, key_len)`: Reads the expected hash from the **current state** by key, preimage from the stack.
- `_check_preimage_value()`: Reads **both** the expected hash and the preimage from the **parameter stack**.

## Stack Values

### Value Types

Values on stacks can be:

1. **Binary (Bin)**
   - Contains: `hint` (String), `data` (Arc<[u8]>)
   - Used for: Keys, signatures, hashes, binary data

2. **String (Str)**
   - Contains: `hint` (String), `data` (Arc<str>)
   - Used for: Text messages, identifiers

3. **Success**
   - Contains: check_count (usize)
   - Indicates successful check operation

4. **Failure**
   - Contains: error message (String)
   - Indicates failed operation

### Stack Behavior

**Parameter Stack:**
- Grows upward (push adds to top)
- Accessed by check operations
- Populated by unlock scripts
- Consumed by lock scripts

**Return Stack:**
- Accumulates operation results
- Contains Success/Failure markers
- Used to determine overall script success

## Security Model

### Resource Limits

| Resource | Default Limit | Strict Limit | Purpose |
|----------|--------------|--------------|---------|
| Fuel | 1,000,000 | 100,000 | Prevent infinite loops |
| Check Count | 256 | 16 | Limit crypto operations |
| Log Size | 64 KB | 4 KB | Prevent log spam |
| Memory | 4 MB | 1 MB | Prevent exhaustion |
| Path Depth | 32 | 32 | Prevent deep nesting |
| Path Length | 1024 | 1024 | Prevent long paths |

### Algorithm Whitelisting

**Allowed Hash Algorithms:**
- SHA-2: sha2-256 (0x12), sha2-512 (0x13), sha2-384 (0x20), sha2-224 (0x1013)
- SHA-3: sha3-256 (0x16), sha3-512 (0x14), sha3-384 (0x15), sha3-224 (0x17)
- Keccak: keccak-256 (0x1b), keccak-512 (0x1d), keccak-384 (0x1c), keccak-224 (0x1a)
- Blake: blake3 (0x1e), blake2b-256 (0xb220), blake2b-512 (0xb240), blake2s-256 (0xb260)

**Blocked Hash Algorithms:**
- SHA-1 (0x11) - collision attacks
- MD4 (0xd4) - broken
- MD5 (0xd5) - broken

**Allowed Signature Algorithms:**
- ed25519-pub (0xed)
- secp256k1-pub (0xe7) - Bitcoin/Ethereum
- p256-pub (0x1200) - NIST P-256
- bls12_381-g1-pub (0xea)

### Input Validation

- **Keys**: Maximum 256 characters, no null bytes
- **Paths**: Maximum 1024 characters, maximum 32 depth
- **Memory sizes**: Maximum 16 MB per operation
- **Pointers**: Overflow checked

## Error Handling

### Error Types

1. **VmError**
   - MissingContext
   - CompilationError
   - InstantiationError
   - ExecutionError

2. **ApiError**
   - MissingExport
   - InvalidParam
   - IncorrectNumberOfParams
   - MemoryDecodeError
   - MemoryAccess
   - NoValue

3. **CryptoError** (domain)
   - InvalidHashAlgorithm
   - HashMismatch
   - SignatureInvalid

### Return Values

**Success (1):**
- Operation completed successfully
- For check operations: Success value pushed to rstack

**Failure (0):**
- Operation failed
- Failure value with error message pushed to rstack

## Memory Model

### Linear Memory

WASM scripts have access to linear memory for string data:
- **Read**: API functions read strings via pointer + length
- **Write**: _branch writes result strings (top-down allocation)
- **Validation**: All accesses are bounds-checked
- **Limit**: Memory size limited by StoreLimits configuration

### Memory Safety

- Pointer overflow detection
- Size validation (<= 16 MB per operation)
- Bounds checking on all reads/writes

## Execution Flow

### Typical Script Pair

**Unlock Script:**
1. Pushes data onto parameter stack
2. May log debug information
3. Returns success

**Lock Script:**
1. Performs check operations on stacked data
2. Verifies cryptographic proofs
3. Returns success only if all checks pass

### Check Counter

The `check_count` tracks cryptographic check operations:
- Incremented by: `check_fail()` when checks fail
- Not incremented by: `succeed()` for successful checks
- Pushed with: Success values to track verification state
- Limited to: 256 operations maximum

## Type Safety

### Newtype Wrappers

The implementation uses newtypes for safety:
- `WasmPtr`, `WasmSize` - Memory addresses/sizes
- `FuelAmount` - Computational limits
- `CheckCount` - Operation counters
- `ContextPath` - Hierarchical paths
- `CurrentKey`, `ProposedKey` - Store keys

## Thread Safety

### Concurrent Execution

- **Instance**: Not thread-safe (`!Send`)
- **Module**: Thread-safe (`Send + Sync`)
- **Engine**: Thread-safe (`Send + Sync`)
- **ModuleCache**: Thread-safe (`Send + Sync`)
- **Value**: Thread-safe with Arc (`Send + Sync`)

### Parallel Patterns

Multiple instances can run in parallel, each with:
- Separate Context
- Separate storage (Pairs, Stack)
- Shared Engine (via Arc)
- Shared compiled modules (via Arc)

## Implementation Requirements

### Mandatory Features

1. **Fuel Tracking**: MUST limit computational resources
2. **Algorithm Whitelisting**: MUST reject weak algorithms
3. **Resource Limits**: MUST enforce all security limits
4. **Input Validation**: MUST validate all user inputs
5. **Memory Safety**: MUST check all memory operations

### Optional Features

1. **Module Caching**: MAY cache compiled modules
2. **Logging**: MAY provide log output
3. **Tracing**: MAY provide debug tracing

## Conformance

A conforming WACC VM implementation MUST:
1. Implement all 9 API functions with specified signatures
2. Enforce security limits (fuel, checks, memory, log)
3. Reject weak cryptographic algorithms
4. Validate all inputs (keys, paths, memory operations)
5. Use dual-stack architecture (parameter + return)
6. Support Success/Failure result values
7. Track check_count for verification

A conforming implementation MAY:
1. Provide additional API functions
2. Use stricter security limits
3. Add additional algorithm support (if secure)
4. Optimize performance (caching, Arc, etc.)

## Security Considerations

### Threat Model

Protected against:
- Resource exhaustion (DoS via infinite loops)
- Memory exhaustion
- Weak cryptography usage
- Path traversal attacks
- Integer overflow
- Buffer overflow

### Defense Mechanisms

1. **Mandatory fuel limits** prevent infinite loops
2. **Check count limits** prevent excessive crypto operations
3. **Algorithm whitelisting** prevents weak crypto
4. **Input validation** prevents malformed data
5. **Memory bounds checking** prevents overflows
6. **Constant-time comparisons** prevent timing attacks

## References

- [Bitcoin Script](https://github.com/bitcoin/bips/blob/master/bip-0342.mediawiki)
- [Multiformats](https://multiformats.io/)
- [Multicodec Table](https://github.com/multiformats/multicodec/blob/master/table.csv)
- [Varuint Encoding](../provenance-specifications/specifications/varuint.md)
- [Multikey Specification](../provenance-specifications/specifications/multikey.md)
- [Multisig Specification](../provenance-specifications/specifications/multisig.md)

## Example Scripts

See the implementation repository for complete examples:
- `crates/wacc/examples/log/` - Logging
- `crates/wacc/examples/unlock/` - Unlock scripts
- `crates/wacc/examples/signature_first/` - Signature verification
- `crates/wacc/examples/signature_lock/` - Signature-based locking
- `crates/wacc/examples/wast/` - WebAssembly Text examples
