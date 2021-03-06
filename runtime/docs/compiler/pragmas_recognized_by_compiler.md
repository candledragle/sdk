# Pragma annotations recognized by the compiler

## Annotations for functions and methods

### Changing whether a function or method is inlined

The user can change whether the VM attempts to inline a given function or method
with the following pragmas.

#### Requesting a function be inlined

```dart
@pragma("vm:prefer-inline")
```

Here, the VM will inline the annotated function when possible. However, other
factors can prevent inlining and thus this pragma may not always be respected.

#### Requesting a function never be inlined

```dart
@pragma("vm:never-inline")
```

Here, the VM will not inline the annotated function. In this case, the pragma
is always respected.

## Annotations for return types and field types

The VM is not able to see across method calls (apart from inlining) and
therefore does not know anything about the return'ed values of calls, except for
the interface type of the signature.

To improve this we have two types of additional information sources the VM
utilizes to gain knowledge about return types:

- inferred types (stored in kernel metadata): these are computed by global
  transformations (e.g. TFA) and are only available in AOT mode

- `@pragma` annotations: these are recognized in JIT and AOT mode

This return type information is mainly used in the VM's type propagator.

Since those annotations side-step the normal type system, they are unsafe and we
therefore restrict those annotations to only have an affect inside dart:
libraries.

### Providing an exact result type

```dart
@pragma("vm:exact-result-type", <type>)
```

Tells the VM about the exact result type (i.e. the exact class-id) of a function
or a field load.

There are two limitations on this pragma:

- The Dart object returned by the method at runtime must have **exactly** the
  type specified in the annotation (not a subtype).

- The exact return type declared in the pragma must be a subtype of the
  interface type declared in the method signature.

  **Note:** This limitation is not enforced automatically by the compiler.

If those limitations are violated, undefined behavior may result.
Note that since `null` is an instance of the `Null` type, which is a subtype of
any other, exactness of the annotated result type implies that the result must
be non-null.

#### Examples for exact result types

```dart
class A {}
class B extends A {}

// Reference to type via type literal
@pragma("vm:exact-result-type", B)
A foo() native "foo_impl";

// Reference to type via path
@pragma("vm:exact-result-type", "dart:core#_Smi");
int foo() native "foo_impl";

class C {
  // Reference to type via type literal
  @pragma('vm:exact-result-type', B)
  final B bValue;

  // Reference to type via path
  @pragma('vm:exact-result-type', "dart:core#_Smi")
  final int intValue;
}
```

### Declaring a result type non-nullable

```dart
@pragma("vm:non-nullable-result-type")
```

Tells the VM that the method/field cannot return `null`.

There is one limitation on this pragma:

- The Dart object returned by the method at runtime **must not** return `null`.

If this limitation is violated, undefined behavior may result.

#### Examples for non-nullable result types

```dart
@pragma("vm:non-nullable-result-type")
A foo() native "foo_impl";

class C {
  @pragma('vm:non-nullable-result-type");
  final int value;
}
```
