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

Repo
----

This repository provides both the spec for the mic file format and a Go module that serves as a reference implementation of the mic format. The spec can be found in the `doc` subdirectory.

[go.mod]: https://github.com/golang/go/wiki/Modules#gomod
