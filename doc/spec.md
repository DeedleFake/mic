mic Spec
========

This file provides a specification for the mic file format. It also provides suggestions on how implementations should deal with typing, in the form of schemas, when reading from these files.

Spec
----

mic is designed to be a direct superset of the [go.mod file format][go.mod], such that a go.mod file is a valid mic file.

### Structure

A mic file is made up of a series of directives. A directive is a single line that looks similar to one of the following:

```mic
name value
name key value
name key => subkey value
name key => subkey => value
# Etc.
```

Directives can also be grouped via parenthesized blocks. For example,

```mic
name (
	value1
	value2
	value3
)
```

is exactly equivalent to

```mic
name value1
name value2
name value3
```

Directives containing the `=>` operator may also be grouped into nested blocks, such as

```mic
name (
	key => (
		subkey value1
		subkey value2
	)
)
```

One difference to note between mic and various serialization formats is that mic files are not designed to denote the types of their own values. Instead, the expected types are defined by the user and the file is parsed accordingly.

In all cases, directive names must fit the regular expression `[a-zA-Z][a-zA-Z0-9_]*`. Keys, subkeys, and values may be any of the basic types described below, although implementations may allow for custom types.

In it's most basic form, `name value`, it is simply a definition of a value to be assigned to a name. A common use for this form might be the inclusion of values in an array. For example, assuming a JavaScript implementation,

```mic
name (
	value1
	value2
)
```

could result in the equivalent of something like `config.name = ['value1', 'value2']`.

The `name key value` form is for more complex, hierarchical data, such as producing a key-value mapping. For example,

```mic
name (
	key1 one
	key2 two
)
```

could be `config.name = {key1: 'one', key2: 'two'}`.

If these levels of hierarchy are not structural enough, the `=>` operator may be used to produce a deeper hierarchy. Again, this is highly dependent on use. Several examples are as follows:

```mic
name (
	key1 => one
	key2 => (
		two
		three
	)
)
```

is the equivalent of `config.name = [{key1: ['one']}, {key2: ['two', 'three']}]`.

```mic
name (
	key => (
		subkey1 one
		subkey2 two
	)
)
```

is the equivalent of `config.name = {key: {subkey1: 'one', subkey2: 'two'}}`.

Comments begin with either `#` or `//` and continue until the end of the line. A comment that begins with `#` must be separated from any previous non-whitespace characters, while a comment beginning with `//` may start at any point in a line. Neither may begin inside of a string literal.

### Basic Value Types

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

Schemas
-------

TODO: This is only a suggestion for implementations. None of this is a requirement. It's essentially a section on how to specify the expected layout and types of a config file.

[go.mod]: https://github.com/golang/go/wiki/Modules#gomod
