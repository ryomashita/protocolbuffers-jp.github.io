+++
title = "Application Note: Field Presence"
weight = 85
linkTitle = "Field Presence"
description = "Explains the various presence-tracking disciplines for protobuf fields. It also explains the behavior of explicit presence-tracking for singular proto3 fields with basic types."
type = "docs"
+++

## Background

*Field presence* is the notion of whether a protobuf field has a value. There
are two different manifestations of presence for protobufs: *no presence*, where
the generated message API stores field values (only), and *explicit presence*,
where the API also stores whether or not a field has been set.

Historically, proto2 has mostly followed *explicit presence*, while proto3
exposes only *no presence* semantics. Singular proto3 fields of basic types
(numeric, string, bytes, and enums) which are defined with the `optional` label
have *explicit presence*, like proto2 (this feature is enabled by default as
release 3.15).

### Presence Disciplines

*Presence disciplines* define the semantics for translating between the *API
representation* and the *serialized representation*. The *no presence*
discipline relies upon the field value itself to make decisions at
(de)serialization time, while the *explicit presence* discipline relies upon the
explicit tracking state instead.

### Presence in *Tag-value stream* (Wire Format) Serialization

The wire format is a stream of tagged, *self-delimiting* values. By definition,
the wire format represents a sequence of *present* values. In other words, every
value found within a serialization represents a *present* field; furthermore,
the serialization contains no information about not-present values.

The generated API for a proto message includes (de)serialization definitions
which translate between API types and a stream of definitionally *present* (tag,
value) pairs. This translation is designed to be forward- and
backward-compatible across changes to the message definition; however, this
compatibility introduces some (perhaps surprising) considerations when
deserializing wire-formatted messages:

-   When serializing, fields with *no presence* are not serialized if they
    contain their default value.
    -   For numeric types, the default is 0.
    -   For enums, the default is the zero-valued enumerator.
    -   For strings, bytes, and repeated fields, the default is the zero-length
        value.
    -   For messages, the default is the language-specific null value.
-   "Empty" length-delimited values (such as empty strings) can be validly
    represented in serialized values: the field is "present," in the sense that
    it appears in the wire format. However, if the generated API does not track
    presence, then these values may not be re-serialized; i.e., the empty field
    may be "not present" after a serialization round-trip.
-   When deserializing, duplicate field values may be handled in different ways
    depending on the field definition.
    -   Duplicate `repeated` fields are typically appended to the field's API
        representation. (Note that serializing a *packed* repeated field
        produces only one, length-delimited value in the tag stream.)
    -   Duplicate `optional` field values follow the rule that "the last one
        wins."
-   `oneof` fields expose the API-level invariant that only one field is set at
    a time. However, the wire format may include multiple (tag, value) pairs
    which notionally belong to the `oneof`. Similar to `optional` fields, the
    generated API follows the "last one wins" rule.
-   Out-of-range values are not returned for enum fields in generated proto2
    APIs. However, out-of-range values may be stored as *unknown fields* in the
    API, even though the wire-format tag was recognized.

### Presence in *Named-field Mapping* Formats

Protobufs can be represented in human-readable, textual forms. Two notable
formats are TextFormat (the output format produced by generated message
`DebugString` methods) and JSON.

These formats have correctness requirements of their own, and are generally
stricter than *tagged-value stream* formats. However, TextFormat more closely
mimics the semantics of the wire format, and does, in certain cases, provide
similar semantics (for example, appending repeated name-value mappings to a
repeated field). In particular, similar to the wire format, TextFormat only
includes fields which are present.

JSON is a much stricter format, however, and cannot validly represent some
semantics of the wire format or TextFormat.

-   Notably, JSON *elements* are semantically unordered, and each member must
    have a unique name. This is different from TextFormat rules for repeated
    fields.
-   JSON may include fields that are "not present," unlike the *no presence*
    discipline for other formats:
    -   JSON defines a `null` value, which may be used to represent a *defined
        but not-present field*.
    -   Repeated field values may be included in the formatted output, even if
        they are equal to the default (an empty list).
-   Because JSON elements are unordered, there is no way to unambiguously
    interpret the "last one wins" rule.
    -   In most cases, this is fine: JSON elements must have unique names:
        repeated field values are not valid JSON, so they do not need to be
        resolved as they are for TextFormat.
    -   However, this means that it may not be possible to interpret `oneof`
        fields unambiguously: if multiple cases are present, they are unordered.

In theory, JSON *can* represent presence in a semantic-preserving fashion. In
practice, however, presence correctness can vary depending upon implementation
choices, especially if JSON was chosen as a means to interoperate with clients
not using protobufs.

### Presence in Proto2 APIs

This table outlines whether presence is tracked for fields in proto2 APIs (both
for generated APIs and using dynamic reflection):

Field type                                   | Explicit Presence
-------------------------------------------- | -----------------
Singular numeric (integer or floating point) | ✔️
Singular enum                                | ✔️
Singular string or bytes                     | ✔️
Singular message                             | ✔️
Repeated                                     |
Oneofs                                       | ✔️
Maps                                         |

Singular fields (of all types) track presence explicitly in the generated API.
The generated message interface includes methods to query presence of fields.
For example, the field `foo` has a corresponding `has_foo` method. (The specific
name follows the same language-specific naming convention as the field
accessors.) These methods are sometimes referred to as "hazzers" within the
protobuf implementation.

Similar to singular fields, `oneof` fields explicitly track which one of the
members, if any, contains a value. For example, consider this example `oneof`:

```protobuf
oneof foo {
  int32 a = 1;
  float b = 2;
}
```

Depending on the target language, the generated API would generally include
several methods:

-   A hazzer for the oneof: `has_foo`
-   A *oneof case* method: `foo`
-   Hazzers for the members: `has_a`, `has_b`
-   Getters for the members: `a`, `b`

Repeated fields and maps do not track presence: there is no distinction between
an *empty* and a *not-present* repeated field.

### Presence in Proto3 APIs

This table outlines whether presence is tracked for fields in proto3 APIs (both
for generated APIs and using dynamic reflection):

Field type                                   | `optional` | Explicit Presence
-------------------------------------------- | ---------- | -----------------
Singular numeric (integer or floating point) | No         |
Singular numeric (integer or floating point) | Yes        | ✔️
Singular enum                                | No         |
Singular enum                                | Yes        | ✔️
Singular string or bytes                     | No         |
Singular string or bytes                     | Yes        | ✔️
Singular message                             | No         | ✔️
Singular message                             | Yes        | ✔️
Repeated                                     | N/A        |
Oneofs                                       | N/A        | ✔️
Maps                                         | N/A        |

Similar to proto2 APIs, proto3 does not track presence explicitly for repeated
fields. Without the `optional` label, proto3 APIs do not track presence for
basic types (numeric, string, bytes, and enums), either. Oneof fields
affirmatively expose presence, although the same set of hazzer methods may not
generated as in proto2 APIs.

Under the *no presence* discipline, the default value is synonymous with "not
present" for purposes of serialization. To notionally "clear" a field (so it
won't be serialized), an API user would set it to the default value.

The default value for enum-typed fields under *no presence* is the corresponding
0-valued enumerator. Under proto3 syntax rules, all enum types are required to
have an enumerator value which maps to 0. By convention, this is an `UNKNOWN` or
similarly-named enumerator. If the zero value is notionally outside the domain
of valid values for the application, this behavior can be thought of as
tantamount to *explicit presence*.

## Semantic Differences

The *no presence* serialization discipline results in visible differences from
the *explicit presence* tracking discipline, when the default value is set. For
a singular field with numeric, enum, or string type:

-   *No presence* discipline:
    -   Default values are not serialized.
    -   Default values are *not* merged-from.
    -   To "clear" a field, it is set to its default value.
    -   The default value may mean:
        -   the field was explicitly set to its default value, which is valid in
            the application-specific domain of values;
        -   the field was notionally "cleared" by setting its default; or
        -   the field was never set.
-   *Explicit presence* discipline:
    -   Explicitly set values are always serialized, including default values.
    -   Un-set fields are never merged-from.
    -   Explicitly set fields -- including default values -- *are* merged-from.
    -   A generated `has_foo` method indicates whether or not the field `foo`
        has been set (and not cleared).
    -   A generated `clear_foo` method must be used to clear (i.e., un-set) the
        value.

### Considerations for Merging

Under the *no presence* rules, it is effectively impossible for a target field
to merge-from its default value (using the protobuf's API merging functions).
This is because default values are skipped, similar to the *no presence*
serialization discipline. Merging only updates the target (merged-to) message
using the non-skipped values from the update (merged-from) message.

The difference in merging behavior has further implications for protocols which
rely on partial "patch" updates. If field presence is not tracked, then an
update patch alone cannot represent an update to the default value, because only
non-default values are merged-from.

Updating to set a default value in this case requires some external mechanism,
such as `FieldMask`. However, if presence *is* tracked, then all explicitly-set
values -- even default values -- will be merged into the target.

### Considerations for change-compatibility

Changing a field between *explicit presence* and *no presence* is a
binary-compatible change for serialized values in wire format. However, the
serialized representation of the message may differ, depending on which version
of the message definition was used for serialization. Specifically, when a
"sender" explicitly sets a field to its default value:

-   The serialized value following *no presence* discipline does not contain the
    default value, even though it was explicitly set.
-   The serialized value following *explicit presence* discipline contains every
    "present" field, even if it contains the default value.

This change may or may not be safe, depending on the application's semantics.
For example, consider two clients with different versions of a message
definition.

Client A uses this definition of the message, which follows the *explicit
presence* serialization discipline for field `foo`:

```protobuf
syntax = "proto3";
message Msg {
  optional int32 foo = 1;
}
```

Client B uses a definition of the same message, except that it follows the *no
presence* discipline:

```protobuf
syntax = "proto3";
message Msg {
  int32 foo = 1;
}
```

Now, consider a scenario where client A observes `foo`'s presence as the clients
repeatedly exchange the "same" message by deserializing and reserializing:

```protobuf
// Client A:
Msg m_a;
m_a.set_foo(1);                  // non-default value
assert(m_a.has_foo());           // OK
Send(m_a.SerializeAsString());   // to client B

// Client B:
Msg m_b;
m_b.ParseFromString(Receive());  // from client A
assert(m_b.foo() == 1);          // OK
Send(m_b.SerializeAsString());   // to client A

// Client A:
m_a.ParseFromString(Receive());  // from client B
assert(m_a.foo() == 1);          // OK
assert(m_a.has_foo());           // OK
m_a.set_foo(0);                  // default value
Send(m_a.SerializeAsString());   // to client B

// Client B:
Msg m_b;
m_b.ParseFromString(Receive());  // from client A
assert(m_b.foo() == 0);          // OK
Send(m_b.SerializeAsString());   // to client A

// Client A:
m_a.ParseFromString(Receive());  // from client B
assert(m_a.foo() == 0);          // OK
assert(m_a.has_foo());           // FAIL
```

If client A depends on *explicit presence* for `foo`, then a "round trip"
through client B will be lossy from the perspective of client A. In the example,
this is not a safe change: client A requires (by `assert`) that the field is
present; even without any modifications through the API, that requirement fails
in a value- and peer-dependent case.

## How to Enable *Explicit Presence* in Proto3

These are the general steps to use field tracking support for proto3:

1.  Add an `optional` field to a `.proto` file.
1.  Run `protoc` (at least v3.15, or v3.12 using
    `--experimental_allow_proto3_optional` flag).
1.  Use the generated "hazzer" methods and "clear" methods in application code,
    instead of comparing or setting default values.

### `.proto` File Changes

This is an example of a proto3 message with fields which follow both *no
presence* and *explicit presence* semantics:

```protobuf
syntax = "proto3";
package example;

message MyMessage {
  // No presence:
  int32 not_tracked = 1;

  // Explicit presence:
  optional int32 tracked = 2;
}
```

### `protoc` Invocation

Presence tracking for proto3 messages is enabled by default
[since v3.15.0](https://github.com/protocolbuffers/protobuf/releases/tag/v3.15.0)
release, formerly up until
[v3.12.0](https://github.com/protocolbuffers/protobuf/releases/tag/v3.12.0) the
`--experimental_allow_proto3_optional` flag was required when using presence
tracking with protoc.

### Using the Generated Code

The generated code for proto3 fields with *explicit presence* (the `optional`
label) will be the same as it would be in a proto2 file.

This is the definition used in the "no presence" examples below:

```protobuf
syntax = "proto3";
package example;
message Msg {
  int32 foo = 1;
}
```

This is the definition used in the "explicit presence" examples below:

```protobuf
syntax = "proto3";
package example;
message Msg {
  optional int32 foo = 1;
}
```

In the examples, a function `GetProto` constructs and returns a message of type
`Msg` with unspecified contents.

#### C++ Example

No presence:

```c++
Msg m = GetProto();
if (m.foo() != 0) {
  // "Clear" the field:
  m.set_foo(0);
} else {
  // Default value: field may not have been present.
  m.set_foo(1);
}
```

Explicit presence:

```c++
Msg m = GetProto();
if (m.has_foo()) {
  // Clear the field:
  m.clear_foo();
} else {
  // Field is not present, so set it.
  m.set_foo(1);
}
```

#### C# Example

No presence:

```c#
var m = GetProto();
if (m.Foo != 0) {
  // "Clear" the field:
  m.Foo = 0;
} else {
  // Default value: field may not have been present.
  m.Foo = 1;
}
```

Explicit presence:

```c#
var m = GetProto();
if (m.HasFoo) {
  // Clear the field:
  m.ClearFoo();
} else {
  // Field is not present, so set it.
  m.Foo = 1;
}
```

#### Go Example

No presence:

```go
m := GetProto()
if m.Foo != 0 {
  // "Clear" the field:
  m.Foo = 0
} else {
  // Default value: field may not have been present.
  m.Foo = 1
}
```

Explicit presence:

```go
m := GetProto()
if m.Foo != nil {
  // Clear the field:
  m.Foo = nil
} else {
  // Field is not present, so set it.
  m.Foo = proto.Int32(1)
}
```

#### Java Example

These examples use a `Builder` to demonstrate clearing. Simply checking presence
and getting values from a `Builder` follows the same API as the message type.

No presence:

```java
Msg.Builder m = GetProto().toBuilder();
if (m.getFoo() != 0) {
  // "Clear" the field:
  m.setFoo(0);
} else {
  // Default value: field may not have been present.
  m.setFoo(1);
}
```

Explicit presence:

```java
Msg.Builder m = GetProto().toBuilder();
if (m.hasFoo()) {
  // Clear the field:
  m.clearFoo()
} else {
  // Field is not present, so set it.
  m.setFoo(1);
}
```

#### Python Example

No presence:

```python
m = example.Msg()
if m.foo != 0:
  # "Clear" the field:
  m.foo = 0
else:
  # Default value: field may not have been present.
  m.foo = 1
```

Explicit presence:

```python
m = example.Msg()
if m.HasField('foo'):
  # Clear the field:
  m.ClearField('foo')
else:
  # Field is not present, so set it.
  m.foo = 1
```

#### Ruby Example

No presence:

```ruby
m = Msg.new
if m.foo != 0
  # "Clear" the field:
  m.foo = 0
else
  # Default value: field may not have been present.
  m.foo = 1
end
```

Explicit presence:

```ruby
m = Msg.new
if m.has_foo?
  # Clear the field:
  m.clear_foo
else
  # Field is not present, so set it.
  m.foo = 1
end
```

#### Javascript Example

No presence:

```js
var m = new Msg();
if (m.getFoo() != 0) {
  // "Clear" the field:
  m.setFoo(0);
} else {
  // Default value: field may not have been present.
  m.setFoo(1);
}
```

Explicit presence:

```js
var m = new Msg();
if (m.hasFoo()) {
  // Clear the field:
  m.clearFoo()
} else {
  // Field is not present, so set it.
  m.setFoo(1);
}
```

#### Objective-C Example

No presence:

```objective-c
Msg *m = [[Msg alloc] init];
if (m.foo != 0) {
  // "Clear" the field:
  m.foo = 0;
} else {
  // Default value: field may not have been present.
  m.foo = 1;
}
```

Explicit presence:

```objective-c
Msg *m = [[Msg alloc] init];
if (m.hasFoo()) {
  // Clear the field:
  [m clearFoo];
} else {
  // Field is not present, so set it.
  [m setFoo:1];
}
```

## Cheat sheet {#cheat}

**Proto2:**

Is field presence tracked?

Field type             | Tracked?
---------------------- | --------
Singular field         | yes
Singular message field | yes
Field in a oneof       | yes
Repeated field & map   | no

**Proto3:**

Is field presence tracked?

Field type             | Tracked?
---------------------- | ------------------------
*Other* singular field | if defined as `optional`
Singular message field | yes
Field in a oneof       | yes
Repeated field & map   | no

**Edition 2023:**

Is field presence tracked?

Field type                                         | Tracked?
-------------------------------------------------- | --------
Default                                            | yes
`features.field_presence` set to `LEGACY_REQUIRED` | yes
`features.field_presence` set to `IMPLICIT`        | no
Repeated field & map                               | no
