## Formalization

We base our formalization on the memory model described in *The Java™ Language Specification* (JLS), Section 17.4 [@jls17], which defines the possible behaviors of a program.

### Definition: Action

**Action** \( a \) is described by a tuple \( \langle k, v, u \rangle \), where:

- \( k \): The kind of action (e.g., read, write, lock, unlock, etc.).
- \( v \): The variable or monitor involved in the action.
- \( u \): The value associated with the action.

*Note: This definition adapts the action formalism from JLS 17.4.2 [@jls17] and omits the thread \( t \) component, as threads are not directly relevant to this work.*

### Definition: Program Trace

A **program trace**, denoted as \( T \), is a global sequence of actions \( \langle a_1, a_2, \dots, a_n \rangle \) that represents the execution of a program during a single run.

### Definition: Read Action

A **read action** occurs when a variable \( v \) is accessed, and a value \( u \) is transmitted from the main memory to the working memory of the thread executing the program.

### Definition: Best Annotation Rule

The type of the program element \( v \) at its declaration should be annotated as `@Nullable` if:

$$
\exists \, a \in T, \ \text{Kind}(a) = \text{Read} \wedge \text{Variable}(a) = v \wedge \text{Value}(a) = \text{null},
$$

where:

- \( \text{Kind}(a) \): The kind of action \( a \) (e.g., read, write, lock, etc.).
- \( \text{Variable}(a) \): The program element \( v \) involved in action \( a \).
- \( \text{Value}(a) \): The value associated with the action \( a \).

### Definition: Dereference Chain

The **dereference chain** \( \text{dereferences}() \) is a partial order over actions in the program trace \( T \) [@jls17]. Formally, for actions \( r \) and \( a \), \( \text{dereferences}(r, a) \) holds if and only if:

- \( r \) is a read action that observes the reference of an object \( o \), and
- \( a \) is a subsequent action that accesses \( o \), such as:
  - reading or writing a field of \( o \) (e.g., \( o.field \)),
  - invoking a method on \( o \) (e.g., \( o.method() \)), or
  - accessing an array element of \( o \) (e.g., \( o[index] \)).

This partial order reflects the causal relationship between the actions \( r \) and \( a \), where \( r \) provides the reference required by \( a \).

### Definition: Dereference(x)

For a program element \( x \), a **dereference(x)** is the set of all actions \( a \) in the program trace \( T \) that access \( x \). Formally:

$$
\text{Dereference}(x) = \{ a \in T \mid \exists r \in T, \ \text{dereferences}(r, a) \wedge v(a) = x \},
$$

### Definition: Null Safety

A program \( P \) is **null-safe** if for every program element \( x \) and for every action \( a \in \text{Dereference}(x) \), all read actions \( r \) in the dereference chain for \( a \) provide a non-`null` reference. Formally:

$$
\forall x \in E_P, \ \forall a \in \text{Dereference}(x), \ \forall r \in T, \ \text{dereferences}(r, a) \implies \text{Value}(r) \neq \text{null},
$$

where:

- \( E_P \): The set of all program elements in the program \( P \), including variables, fields, formal parameters, and method return values.

## Proof Sketch for Null Safety

We aim to prove that if a program is annotated according to the _Best Annotation Rule_ (Definition: Best Annotation Rule) and a sound nullability type checker reports no errors, then the program satisfies the definition of null safety (Definition: Null Safety).

### Case Analysis:

1. **Case 1**: At least one read action \( r \) in the dereference chain retrieves a `null` reference.
    - Let \( a \) be an action accessing \( x \), and let \( r \) be a read action such that \( \text{dereferences}(r, a) \wedge v(a) = x \).
    - If \( \exists r \in T, \ \text{Value}(r) = \text{null} \), then by the _Best Annotation Rule_, \( x \) must be annotated as `@Nullable`.
    - The nullability type checker ensures that `@Nullable` elements cannot be dereferenced without a null check. If there is a null check, the checker performs a local flow-sensitive refinement, treating the element as `@NonNull` within the scope of the null check. If \( a \) is not properly guarded, the checker raises a compilation error, preventing program execution.
    - Therefore, any unsafe dereference of \( x \) is statically rejected, and the program satisfies null safety.

2. **Case 2**: All read actions \( r \) in the dereference chain retrieve non-`null` references.
    - If \( \forall r \in T, \ \text{dereferences}(r, a) \implies \text{Value}(r) \neq \text{null} \), then \( x \) is not annotated as `@Nullable`.
    - By Definition: Null Safety, null safety is satisfied because no dereference chain retrieves a `null` value.

*Note: This proof assumes the presence of a sound nullability checker that rejects programs with unsafe dereferences.*
