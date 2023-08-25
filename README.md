<nav>
* TOC should appear here
{:toc}
</nav>

# WIT Quick Reference

This reference is derived from the [Component Model design docs](https://github.com/WebAssembly/component-model/tree/main/design/mvp), especially the [Explainer](https://github.com/WebAssembly/component-model/blob/main/design/mvp/Explainer.md) and [WIT design](https://github.com/WebAssembly/component-model/blob/main/design/mvp/WIT.md). Any contradictions between this reference and those documents are bugs _here_. [Update PRs welcome](https://github.com/lann/wit-quick-ref)!

## Types

[Reference](https://github.com/WebAssembly/component-model/blob/main/design/mvp/WIT.md#types)

### Basic Types

- Booleans: `bool`
- Unsigned integers: `u8`, `u16`, `u32`, `u64`
- Signed integers: `s8`, `s16`, `s32`, `s64`
- Floating-point numbers: `float32`, `float64`
- [Unicode Scalar Values](https://unicode.org/glossary/#unicode_scalar_value): `char`
- Unicode strings: `string`

### Records

A [`record`](https://github.com/WebAssembly/component-model/blob/main/design/mvp/WIT.md#item-record-bag-of-named-fields) is a set of named fields, similar to a `struct` in many other languages.

```wit
record pair {
    x: u32,
    y: u32,
}
```

#### Tuples

A tuple is a specialized `record` with implicit numeric field names.

The Component Model treats this the same as an equivalent `record` but code generators may produce different code for some languages.

```wit
type entry = tuple<string, u64>
```

### Flags

A [`flags`](https://github.com/WebAssembly/component-model/blob/main/design/mvp/WIT.md#item-flags-bag-of-bools) type represents a set of named booleans. It may be represented more efficiently than a `record` of `bool` fields.

```wit
flags permissions {
    read,
    write,
    delete,
}
```

### Lists

A `list<T>` is a variable-length sequence of values, all of the same type `T`.

```wit
// Lists are used for byte buffers:
type bytes = list<u8>

// Lists can contain compound types as well:
type pairs = list<pair>
```

### Variants

A [`variant`](https://github.com/WebAssembly/component-model/blob/main/design/mvp/WIT.md#item-variant-one-of-a-set-of-types) represents exactly one of several named cases, similar to a "tagged union" or "sum type" in other languages.

```wit
variant filter {
    all,
    none,
    some(list<string>),
}
```

The following types are specialized kinds of `variant`. The Component Model treats them the same as an equivalent `variant` but code generators may produce different code for them for some languages.

#### Enums

An [`enum`](https://github.com/WebAssembly/component-model/blob/main/design/mvp/WIT.md#item-enum-variant-but-with-no-payload) is a specialized `variant` where none of the cases have a payload.

```wit
enum directions {
    north,
    east,
    south,
    west,
}

```

#### Options

An `option<T>` is a specialized `variant` with `none` and `some(T)` cases.

```wit
type maybe-named = option<string>
```

#### Results

A `result<T, E>` is a specialized `variant` with `ok(T)` and `err(E)` cases.

```wit
type error-msg = string
type bytes-result = result<list<u8>, error-msg>
```

### Resources

> TODO: To avoid confusion, resources (and handles, `borrow`s) are omitted until tooling is ready.

## Scopes and name resolution

### Identifiers

[Reference](https://github.com/WebAssembly/component-model/blob/main/design/mvp/WIT.md#identifiers)

WIT identifiers are used for naming items (packages, worlds, interfaces, types), and must follow the Component Model `name` rules:

- Identifiers use `kebab-case`, with ASCII-only words separated by a single `-`. Each word must be either all `lowercase` or all `UPPERCASE`.
- Identifiers can be prefixed with a `%` to allow using [WIT keywords](https://github.com/WebAssembly/component-model/blob/main/design/mvp/WIT.md#keywords) as identifiers, e.g. `%interface`.

**Allowed**: "`singleton`", "`multiple-words`", "`acronyms-TLA`", "`%interface`", "`%unnecessary-prefix`"

**Not allowed**: "`Mixed-Case`", "`double--dashes`", "`-outer-dashes-`", "`interface`"

> TODO: explain [name resolution](https://github.com/WebAssembly/component-model/blob/main/design/mvp/WIT.md#name-resolution)

### Type aliases

> TODO

### Renaming with `use`

> TODO: explain different forms of `use`

## Comments

```wit
// Line comment

/*
Block comment /* nesting allowed! */
*/

/// Doc comment
very-well-documented: func()

/**
Block-doc comment
*/
interface extremely-well-documented {}
```

## Functions

[Reference](https://github.com/WebAssembly/component-model/blob/main/design/mvp/WIT.md#wit-functions)

> Functions are defined in an `interface` or are listed as an `import` or `export` from a `world`. Parameters to a function must all be named and have unique names:

```wit
interface quick-ref-funcs {
    set: func(key: string, val: u64) -> option<u64>

    // Return values are optional
    do-nothing: func()
}
```

## Interfaces

[Reference](https://github.com/WebAssembly/component-model/blob/main/design/mvp/WIT.md#wit-interfaces)

> An interface can be thought of as an instance in the WebAssembly Component Model, for example a unit of functionality imported from the host or implemented by a component for consumption on a host.

```wit
interface line-drawer {
    // Interfaces can define types to be used within the same interface or by
    // another interface.
    record point {
        x: float32,
        y: float32,
    }

    draw: func(start: point, end: point)
}
```

## Worlds

[Reference](https://github.com/WebAssembly/component-model/blob/main/design/mvp/WIT.md#wit-worlds)

> A world is a complete description of both imports and exports of a component. A world can be thought of as an equivalent of a `component` type in the component model.

```wit
world my-world {
    // Worlds can `include` another world, which merges that world's
    // imports and exports into this one:
    include other-package:other-world

    // Exports and imports can refer to separately-defined interfaces:
    export wasi:http/incoming-handler
    import wasi:http/outgoing-handle

    // A world can also define interface or function imports/exports in-line:
    export my-interface : interface {
        ping: func()
    }
    import get-config: func() -> string
}
```

## Packages

> A WIT package is a collection of WIT `interfaces` and `worlds` defined in files in the same directory that that all use the file extension wit, for example foo.wit. Files are encoded as valid utf-8 bytes. Types can be imported between interfaces within a package and additionally from other packages through IDs.

`main.wit`

```wit
// Package identifiers have a namespace, name, and optional SemVer version.
package quick-ref:example@0.0.1-alpha.1

world quick-ref-world {
    // Top-level items from other files are in scope.
    import config
}
```

`config.wit`

```wit
// Only one file needs to specify a package identifier.

interface config {
    get-config: func() -> string
}
```
