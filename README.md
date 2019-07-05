mic
===

mic (Mod Inspired Config) is a config file format inspired by the format used for [go.mod files][go.mod]. Unlike JSON, YAML, and TOML, it is _not_ a serialization format, and is not designed for storing arbitrary data. It is designed only for use as a format for storing configuration information. While it may often be suitable for the storage of arbitrary data, unlike these other data storage formats, that is not its goal.

Goals
-----

mic files are designed to be:

* Easy to read.
* Easy to parse.
* Easy to edit.
* Capable of storing general config info useful to a large variety of types of pieces of software.

To this end, a number of shortcomings of similar languages have been identified to be avoided:

* JSON does not support comments. This is an obvious problem for a config file.
* JSON does not support trailing commas. This adds a level of completely unnecessary tediousness to editing it.
* YAML is indentation based. This ensures well-formatted files, but also allows easy accidental introduction of subtle mistakes.
* TOML has subtle distinctions between some types that can be quite odd to deal with. For example, the difference between an object definition and an array of objects is rather strange.

Spec
----

### Structure

A mic file is made up of a series of directives. A directive is a single line that takes one of two basic forms:

```mic
name value metadata
```

or

```mic
name key metadata => value metadata
```

In the first form, the directive is a definition of a value to be assigned to a name. A name must fit the regular expression `[a-zA-Z][a-zA-Z0-9_]*`. A value may be any of the types defined below. The metadata field is similar to the value field, but it is optional and is usage-specific. For more information, see the section on metadata below.

These directives are ordered definitions of the values to be assigned to a name. In other words, if multiple directives with the same name appear multiple times in the same file, these are considered to be similar to an array definition. For example,

```mic
example one
example two
```

would be similar to saying `config.example = ['one', 'two']` in JavaScript.

In the second form, the directive means that a key-value mapping is established in the config entry so named. A key must have fit the same regular expression that a config entry name does, as specified above. Like with the first form, this may be specified multiple times. In other words,

```mic
example k1 => one
example k2 => two
```

would be similar to the JavaScript statement `config.example = {k1: 'one', k2: 'two'}`. If the same key is used multiple times, then, like specifying the same directive name multiple times, it means that an array is to be stored in that object.

In all of these cases, for simplicity, an implementation may opt to consider a single directive or key to be equivalent to an array with a single entry, rather than a stand-alone value.

To avoid repetition, directives may be grouped via a parenthesized block. For example,

```mic
example (
	one
	two
)
```

is exactly equivalent to

```mic
example one
example two
```

and

```mic
example (
	k1 => one
	k2 => two
)
```

is exactly equivalent to

```mic
example k1 => one
example k2 => two
```

### Value Types

A value may be one of three types: A string, a number, or a boolean. Which type the value is can be determined from the raw value, but there are some ambiguities that can be settled differently depending on implementation and usage. For more information, see the section on schemas below. This section describes the default way in which ambiguities are determined.

#### Strings

A string may be defined in one of three ways:

* Any series of non-whitespace characters that is not ambiguous with the other types.
* A series of characters surrounded by double quotes. These may not contain newlines, but may contain any valid Unicode character. They also support the following escape sequences:
	* `\n`: newline character
	* `\t`: tab character
	* `\"`: double quote
* A series of any number of backquotes followed by an arbitrary number of valid Unicode characters, including newlines, followed by the same number of backquotes as the ones that started the string. This is a raw string: Any characters in between the sets of backquotes are considered to be part of the string simply as they are. This is also the only way in which a single directive can be placed across multiple lines.

#### Numbers

Numbers may be defined in one of two ways:

* An optional `-` followed by any number of numeric characters, optionally followed by a `.` and more numeric characters. If the period and following numeric characters are present, than the first set of numeric characters are optional. In other words, a number may be of either the form `-?[0-9]+(\.[0-9]+)?` or the form `-?\.[0-9]+`. This form specifies a floating point number in base-10.
* An integer in the range [2,36] followed by the character `x` and followed by any number of numeric characters and possibly alphabet characters, depending on the number preceding the `x`. This form specifies an integer the base preceding the `x`. In other words, `2x101` would be the number 5, specified in binary, while `20xG` would be 17, specified in base-20.

#### Booleans

Boolean values are much simpler than the others, as there are only two possible ones: `true` and `false`.

### Hierarchy

TODO: Explain how hierarchical directives work.

### Metadata

TODO: Explain what metadata is for and how implementations should use it.

Schemas
-------

TODO: This is only a suggestion for implementations. None of this is a requirement. It's essentially a section on how to specify the expected layout and types of a config file.

[go.mod]: https://github.com/golang/go/wiki/Modules#gomod
