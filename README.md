# Jacl Specification 0.1

Jacl is a straightforward configuration language with the following features:

* Indentation does NOT matter.
* All values in the configuration have a non-ambiguous type.
* Separators between values are optional.
* NO cheap tricks.

Goals:

* The specification should be trivial to understand and easy to implement.
* A configuration file should be easy to edit and easy to understand.
* Trivial mistakes shouldn't change the *meaning* of the configuration file drastically.

Non-goals:

* Anything not necessary in a configuration file.


## Change Log

### v0.1.0 (2019-06-22)

* Initial release.

## Sample

```
# This is a Jacl document. Boom.

owner: {
    name: "Phillips Redd"
    age: 34
    bio: """
        Coder.
        Loves cats.
        """
}

database: {
    server: "192.168.1.1"
    ports: [8001, 8001, 8002]
    connection_max: 5000
    enabled: true
}

    /*
        You can indent as you please, even if it doesn't make sense.
        Tabs or spaces. Jacl doesn't care.
        Oh, and this is a multiline comment!
    */
    servers: {
        alpha: {
            ip: "10.0.0.1"
            dc: "eqdc10"
        }
        beta: {
            ip: "10.0.0.2"
            dc: "eqdc10"
        }
    }

clients: {
    data: [
        ["gamma", "delta"]
        [1, 2]
    ]
}
```

### Base Specification

Host language: The programming language that implements this specification.

#### Empty File

An empty file is a valid Jacl file.

#### Comments

Single line comments start with `#`:

    # This is a single line comment.

Multiline comments start with `/*` and end with `*/`:

    /*
        This is a multiline
        comment
    */

#### Properties

Properties are defined at the top level of a file, and they have the following structure:

    [NAME][:] [VALUE]

##### Property name

Property name is a string:

    prefix: "fun-"

Optionally with quotes (`"`) around it:

    "new prefix": "more-fun-"

Escape characters are not evaluated:

    "newest\nprefix": "most-fun-"

The name above is evaluated to `newest\nprefix`, NOT:

    newest
    prefix

The maximum length of a property name is 512 characters.

##### Property value

Property value can be one of the following types:

* string
* signed integer
* unsigned integer
* float
* boolean
* array
* map

#### Strings

A string is always quoted with `"`, `'''` or `"""`. Escapes in the former one are expanded, but not in the latter ones. Latter ones can span multiple lines.

A simple string:

    "A simple string"

Simple string with escape:

    "Simple\tstring\nwith escape"

A raw string:

    '''a raw string''''

A multiline raw string:

    '''
    Multiline
    string
    '''

Another multiline raw string:

    """Another
    multiline
    string"""

#### Integers

Integers always have the biggest size the host language supports natively (usually 64 bits these days).

Integers cannot have `0` as a prefix, unless the number itself is `0`. So, this is invalid: `0145`.

If a number has one of the following prefixes below, then it is an unsigned integer:

* `0b`: binary, e.g., `0b10101`.
* `0o`: octal, e.g., `0o744`.
* `0d`: decimal, eg., `0d987123`.
* `0x`. hexadecimal, e.g., `0xBEEF`.

Otherwise, it is a signed integer. Signed integers may have `-` or `+` prefixes.

```
    -123
    123
    +123
```

#### Floats

Floats always have the biggest size the host language supports natively (usually 64 bits these days).

A float should always have a decimal point; that's how floats are distinguished from integers.

`NaN` and `Infinity` are not supported in the base specification.

#### Boolean

One of `true` or `false`.

#### Array

An array may contain items of arbitrary types. Items in the array are written between brackets `[ ... ]`. Commas between items are optional.

Empty array is `[]`.

This array:

    ["green" true [1 2 3] {name: "John" age: 30}]

is equivalent to:

    ["green", true, [1, 2, 3] {name: "John", age: 30}]

which in turn is equivalent to:

    [
        "green"
        true
        [1, 2, 3]
        {name: "John", age: 30}
    ]

#### Map

A map contains keys and their mapping values. The keys can only be strings and they have the same restrictions as a property name. Keys and values of a map are written between curly braces: `{ ... }`. A key and a value is separated by colons `:`. Commas are optional.

Empty map is `{}`.

This map:

    {name: "John" age: 30 groups: ["sales" "management"]}

is equivalent to:

    {name: "John", age: 30, groups: ["sales", "management"]}

which in turn is equivalent to:

    {
        name: "John"
        age: 30
        groups: ["sales", "management"]
    }

Of course, a map can be many levels deep:

    {
        title: "An important document"
        attributes: {
            permissions: {
                read: true
                write: false
            }
        }
    }


## License

Copyright (c) 2019 Yuce Tekol. Licensed under [MIT](LICENSE).