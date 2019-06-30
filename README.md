# Jacl Specification 0.1

Jacl is a straightforward configuration language with the following features:

* Indentation does NOT matter.
* All values in the configuration have a non-ambiguous type.
* Separators between values are optional.
* NO magic, NO cheap tricks.

Goals:

* The specification should be trivial to understand and easy to implement.
* A configuration file should be easy to edit and easy to understand.
* Trivial mistakes shouldn't change the *meaning* of the configuration file drastically.

Non-goals:

* Anything not necessary in a configuration file.


## Change Log

### v0.1.2 (2019-06-26)

* Single line comments starts with `//` instead of `#`.
* Numbers may optionally include underscores to increase their readability, e.g., `12_345_678_900 == 12345678900`.

### v0.1.1 (2019-06-23)

* Added extended specification.

### v0.1.0 (2019-06-22)

* Initial release.

## Sample

```
// This is a Jacl file. Boom.

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
    ports: [8001 8002 8003]
    connection_max: 5_000
    enabled: true
}

// Some code in the config. Indentation matters only when it should:
source: trim"""
    def main():
        if True:
            print("OK, fine")
        else:
            print("Not fine")
    """

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
        [1 2]
    ]
}
```

## Table Of Contents

1. [Base Specification](#base-specification)
    1. [Grammar](#grammar)
    2. [Definitions](#definitions)
    3. [Empty File](#empty-file)
    4. [Comments](#comments)
    5. [Properties](#properties)
        1. [Property name](#property-name)
        2. [Property value](#property-value)
    6. [Data Types](#data-types)
        1. [String](#string)
        2. [Unsigned integer](#unsigned-integer)
        2. [Signed integer](#signed-integer)
        3. [Float](#float)
        4. [Boolean](#boolean)
        5. [Array](#array)
        6. [Map](#map)
2. [Extended Specification](#extended-specification)
    1. [Raw String Functions](#raw-string-functions)
        1. [trim](#trim)
        2. [pin](#pin)

## Base Specification

### Grammar

The Jacl grammar is written using [ANTLR4](https://www.antlr.org/index.html). You can find the grammar [here](Jacl.g4).

### Definitions

Host language: The programming language that implements this specification.

### Empty File

An empty file is a valid Jacl file.

### Comments

Single line comments start with `//`:

    // This is a single line comment.

Multiline comments start with `/*` and end with `*/`:

    /*
        This is a multiline
        comment
    */

### Properties

Properties are defined at the top level of a file, and they have the following structure:

    [NAME][:] [VALUE]

#### Property name

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

#### Property value

Property value can be one of the following types:

* [String](#string)
* [Unsigned integer](#unsigned-integer)
* [Signed integer](#signed-integer)
* [Float](#float)
* [Boolean](#boolean)
* [Array](#array)
* [Map](#map)

### Data Types

#### String

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

#### Unsigned integer

Unsigned integers always have the biggest size the host language supports natively (usually 64 bits these days).

If a number has one of the following prefixes below, then it is an unsigned integer:

* `0b`: binary, e.g., `0b10101`.
* `0o`: octal, e.g., `0o744`.
* `0d`: decimal, eg., `0d987123`.
* `0x`. hexadecimal, e.g., `0xBEEF`.

Underscore (`_`) is allowed between digits:

* `0b1_0_1_0_1 == 0b10101`
* `0d13_490_567 == 0d13490567`
* `0o12_567 == 0o12567`
* `0xAB_CD_EF12 == 0xABCDEF12`

#### Signed integer

Signed integers always have the biggest size the host language supports natively (usually 64 bits these days).

Signed integers cannot have `0` as a prefix, unless the number itself is `0`. So, this is invalid: `0145`.

Signed integers may have the sign: `-` or `+`.

```
    -123
    123
    +123
```

Underscore (`_`) is allowed between digits: `12_345_678 == 12345678`.

#### Float

Floats always have the biggest size the host language supports natively (usually 64 bits these days).

A float should always have a decimal point; that's how floats are distinguished from integers.

Underscore (`_`) is allowed between digits: `12_345_678.910_405 == 12345678.910405`.

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

## Extended Specification

### Raw String Functions

#### trim

Sets the column of the first non-space character of ther first non-empty line as the pin point. Removes empty lines at the beginning and at the end. Removes space characters from each line so the pin point is the first column.

Example:

```
some_text: trim"""

    This is line 1.
        This is line 2.

    This is line 3.
    """
```

In the example above, pin point is the 5th column. 4 spaces are removed from each line. So `some_text` property is set to:

```
This is line 1.
    This is line 2.

This is line 3.
```

It is an error to have a line with less spaces than the pin point. Parsing the following text returns an error. The pin point is set to 5th column, but that causes loss of characters in the 2nd line:

```
invalid_text: trim"""

    This is line 1.
This is line2.
"""
```

See the [pin](#pin) function to set the pin point manually.

`trim` is especially useful for indentation-sensitive text, like Python code:

    source: trim"""
        import sys

        def main():
            args = sys.argv[1:]
            if args:
                print(f"{len(args)} arguments passed.")
                for i, arg in enumerate(args):
                    print(f"  {i}. {arg}")
            else:
                print("No arguments passed.")

        if __name__ == "__main__":
            main()
        """


#### pin

This function allows to set the pin point manually using the caret (`^`). The pin line and the empty lines before it are removed, but not the empty lines after it.

Example:

```
some_text: pin"""

    ^
    This is line 1.
        This is line 2.

    This is line 3.

    """
```

In the example above, pin point is the 5th column. 4 spaces are removed from each line. So `some_text` property is set to:

```
This is line 1.
    This is line 2.

This is line 3.

```

Note that the blank line at the end of the text was not removed.

The same text using a different pin point:

```
some_text: pin"""

  ^
    This is line 1.
        This is line 2.

    This is line 3.

    """
```

This time, pin point is the 3rd column. 2 spaces are removed from each line. So `some_text` property is set to:

```
  This is line 1.
      This is line 2.

  This is line 3.

```

Just like `trim`, it is an error if the pin point choice results in loss of non-space characters. The following returns an error:

```
invalid_text: pin"""
  ^
    This is line 1.
This is line2.
"""
```

Moving the pin point to the 1st column makes that configuration valid:

```
valid_text: pin"""
^
    This is line 1.
This is line2.
"""
```

The pin must be the first non-space character in the text, otherwise an error is returned:

```
invalid_text: pin"""
    Here, some text.
    ^
    Rest of the text.
    """
```

It is an error to not have the pin:

```
invalid_text: pin"""
    Here, some text.
    Rest of the text.
    """
```

`pin` preserves the space on empty lines in contrast with `trim` which removes them.

## Thanks

* Alan Bernstein for the name of the language.
* Matt Jaffee for early feedback.

## License

Copyright (c) 2019 Yuce Tekol. Licensed under [MIT](LICENSE).