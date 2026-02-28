# XDV-051 — Domain Type System Extensions  
## K / Q / Φ Type Markers and Compile-Time Domain Safety  

**Specification ID:** XDV-051  
**Version:** 0.1-draft  
**Status:** Normative  
**Depends On:** XDV-001 through XDV-050  

---

# 1. Purpose

This document defines Domain Type System Extensions (DTSE) for XDV.

It specifies:

- K / Q / Φ type markers  
- Compile-time domain safety  
- Domain contract verification  

DTSE extends the Dust Programming Language (DPL) type system to enforce domain correctness at compile time.

Domain violations SHALL be impossible to compile.

---

# 2. Design Goals

The Domain Type System SHALL:

1. Encode domain identity into types.
2. Prevent illegal cross-domain mutation.
3. Enforce no-cloning (Q) at type level.
4. Enforce Φ integrity boundaries.
5. Validate domain contracts statically.
6. Integrate with capability and resource models.

Type safety SHALL precede runtime enforcement.

---

# 3. K / Q / Φ Type Markers

## 3.1 Domain Type Annotation

All domain-bound types SHALL carry a domain marker:

```
@domain(K)
struct KValue<T> { ... }

@domain(Q)
struct QState { ... }

@domain(Φ)
struct PhiEnvelope { ... }
```

Domain marker SHALL become part of type identity.

---

## 3.2 Domain-Tagged Types

General form:

```
DomainType<D, T>
```

Where:

- D ∈ { K, Q, Φ }
- T = underlying data or descriptor

Examples:

```
DomainType<K, u64>
DomainType<Q, CircuitDescriptor>
DomainType<Φ, TransformDescriptor>
```

Types from different domains SHALL NOT be implicitly convertible.

---

## 3.3 No Implicit Domain Coercion

The following SHALL be rejected at compile time:

```
let x: DomainType<Q, _> = some_k_value;
```

Explicit projection functions MUST be used.

---

# 4. Compile-Time Domain Safety

## 4.1 Cross-Domain Function Constraints

Functions SHALL declare domain constraints:

```
fn submit_q_job(job: DomainType<Q, JobSpec>) -> DomainType<K, MeasurementHandle>
```

Compiler SHALL verify:

- Correct domain input.
- Correct domain output.
- No illegal domain mutation.

---

## 4.2 Domain Separation Rule

For any types A and B:

If Domain(A) ≠ Domain(B)

Then:

- No direct assignment permitted.
- No implicit reference sharing permitted.
- No shared mutable alias permitted.

Violation SHALL be compile-time error.

---

## 4.3 Q No-Cloning Enforcement

Q types SHALL be marked non-copyable:

```
@domain(Q)
@noncopy
struct QState { ... }
```

Compiler SHALL enforce:

- No implicit copy.
- No clone trait implementation.
- Move semantics only.

Attempted duplication SHALL fail compilation.

---

## 4.4 Φ Integrity Type Constraints

Φ types SHALL:

- Disallow mutable aliasing.
- Restrict mutation to transform contracts.
- Require explicit coherence scope annotation.

Example:

```
fn apply_transform(
    env: &mut DomainType<Φ, PhiEnvelope>,
    t: TransformContract
)
```

Mutation outside allowed contract SHALL be compile-time error.

---

# 5. Domain Contract Verification

## 5.1 Contract-Parameterized Types

Domain contracts SHALL be encoded in type system:

```
struct QContainer<C: QContract> { ... }
struct PhiEnvelope<C: PhiContract> { ... }
```

Contract traits SHALL define:

- Allowed operations
- Scope limits
- Capability requirements

Compiler SHALL verify contract compliance.

---

## 5.2 Capability-Aware Types

Capability tokens MAY be encoded as type parameters:

```
struct Capability<D, Scope>
```

Functions requiring capability SHALL require capability type:

```
fn measure(
    state: DomainType<Q, QState>,
    cap: Capability<Q, MeasureScope>
) -> DomainType<K, MeasurementResult>
```

Missing capability SHALL produce compile-time error.

---

## 5.3 Resource Contract Binding

Resource contracts MAY be embedded in types:

```
struct BoundQState<C: QResourceContract> { ... }
```

Operations SHALL require valid resource-bound types.

Compiler SHALL reject operations on unbound Q or Φ types.

---

# 6. Deterministic Type-Level Guarantees

The type system SHALL ensure:

1. No domain boundary crossing without explicit API.
2. No Q-state cloning.
3. No Φ-state aliasing.
4. Capability presence at compile time.
5. Deterministic function signatures.

Type checking SHALL be deterministic and reproducible.

---

# 7. Integration with DPL Kernel Subset

DTSE SHALL be mandatory in:

- Kernel code
- Domain managers
- Scheduler logic
- Security-critical components

User-space MAY use relaxed profiles but MUST comply when invoking kernel APIs.

---

# 8. Static Analysis Requirements

Compiler SHALL verify:

- Domain marker consistency.
- Absence of cross-domain mutable aliasing.
- Proper capability propagation.
- Contract trait compliance.
- No unsafe escape of Q or Φ internal representation.

Unsafe escape SHALL require explicit `unsafe` annotation and audit.

---

# 9. Replay and Audit Compatibility

Domain annotations SHALL:

- Be visible in debug metadata.
- Support trace-level introspection.
- Allow verification that no illegal cross-domain call occurred.

Type information MAY be included in audit trace metadata.

---

# 10. Compliance Requirements

An implementation is compliant if:

1. Domain markers are enforced at type level.
2. Q-state is non-copyable at compile time.
3. Φ mutation requires contract-bound types.
4. Capability requirements are encoded in function signatures.
5. Cross-domain coercion is impossible without explicit projection.
6. Type checking is deterministic.

---

# 11. Closing Statement

Runtime enforcement protects execution.

Type enforcement prevents illegal execution from existing at all.

In XDV:

Domains are not only runtime concepts —
They are type-level facts.

K, Q, and Φ are encoded in the type system.

No silent crossing.
No hidden duplication.
No accidental corruption.

Hybrid safety begins at compile time.