
## A Tour

This project is currently in the works. I will be posting a Trello on Trinity very soon, and I'm looking forward for anyone out there to contribute.

If you spot any errors and/or you have an idea how to make this tutorial better, fork this repo, and pull your changes to [this repository](https://github.com/tehfynlnyt/trinity-language).

### Overview

This document provides an overview of the syntax, operations, and semantics in the Trinity language, as well as a comprehensive guide to the modules in the Standard Library. Other parts of Trinity, like scoping rules or runtime semantics, will be described informally.

### File Types

Trinity only encodes text in UTF-8; other encodings are not supported. Any of the standard line termination sequences can be used, depending on the platform: `\r`, `\n` or `\r\n`.

Trinity has only three file types: module (`*.mir`), script (`*.mirs`), config (`*.mco`) and markup (`*.mml`).

Module files are the most commonly used as they can be imported and exported through packages. The entry point of a Trinity module is defined in the `main` function.

```dart
/// @file .mir
fun main(*args: []Str): Void { /*...*/ }
```

The type annotations or the spread `*args` declaration can be left out, so it can be `fun main {}` instead.

Script files do not have a `main` function, but they can import other modules and files.

Trinity markup is a special syntax of Trinity derived from Markdown, HTML and CSS which you can use to declaratively build user interfaces. Trinity config files are like YAML and JSON, though they can be interspersed with arbitrary Trinity expressions.

### Tokens and whitespace

The characters used in Trinity fall into four groups:

- White space characters
- Alphanumerics: letters, digits, combining punctuation and underscores
- Operator characters (other printable characters excluding below)
- Punctuation: `` (){}[],;\'"` ``

Each token consists of a sequence of consecutive characters from just one of those groups, excluding whitespace. Whitespace is ignored except they separate tokens.

A sequence of alphanumeric characters, with no additional non-alphanumeric characters, is a single token. White-space must be used to separate two such tokens in a program. The same thing goes for operators.

### Comments

Comments start anywhere outside a string or character literal with two slashes, and runs until the end of the line. The end of line characters belong to the piece.

If the next line only consists of a comment piece with no other tokens between it and the preceding one, it does not start a new comment:

```dart
1 // This is a single comment over multiple lines.
// The scanner merges these two pieces.
// The comment continues here.
```

Documentation comments are comments that start with three slashes `///` rather than two. Documentation comments are tokens; they are only allowed at certain places in the input file as they belong to the syntax tree!

```dart
1 /// This is a documentation comment
```

Trinity supports two types of multi-line comments beginning with `/*` and ending in `*/`.

```dart
/*  Comment here.
    Multiple lines
    are not a problem. */
```

`/+ +/` allow nesting.

```dart
/+
/+ Multiline comment in already commented out code. +/
+/
```

Multiline documentation comments also exist and support nesting too. They begin with two asterisks or plus signs (`/**` `/++`) instead of one, and end in only one of each type.

```dart
/** this is a multi-line documentation comment */
/++ and this is its nested cousin +/
```

#### Keywords

The following regular expression denotes all the keywords of the language, including those used for declarations, such as `var`.

Keywords are grouped into three:

- expression keywords, which are keywords used as operators;
- declaration keywords, which declare program entities such as variables, classes and functions,
- modifier keywords which modify such declarations,
- general keywords which command and control program flow and execution.
- pre-defined, or dynamic constants and variables.

<!--  -->

    in of as is new to til thru by unset

    var val let const decl def fun type
    class enum mod pack struct inter space
    proc proto macro given style elem field
    ext pred data trait lemma iter sub

    if un elif elun else then
    for each loop while until when
    with do from
    try throw catch fix
    switch match case fail
    unite queue spawn kill lock
    break skip redo retry return await label yield goto pass
    import export using
    desc debug check assert assume

    true false null void nan infin
    it this that super self target
    params ctor proto pro

#### Identifiers

In [The Unicode Standard 14.0](https://www.unicode.org/versions/Unicode14.0.0/UnicodeStandard-14.0.pdf), **Table 4-4** defines a set of character categories for all Unicode characters.

Trinity treats the entire Unicode `L` super-category as Unicode letters, `M` as combining marks, `Pc` as "underscores", `Pd` as dashes and `Nd` as digits.

An identifier is any sequence of letters, digits, underscores and diacritics, but do not start with diacritics or combining marks. JSX tags can include dashes, but must not end with any amount of trailing dashes.

Naming conventions follow Java or JavaScript. There are four types of identifiers which Trinity recognizes and highlights accordingly:

- `SHOUT_CASE`, used for constants,
- `PascalCase` used for classes, modules, namespaces, and types.
- `camelCase` or `snake_case` used for variables, parameters, functions and methods.
- `_leading` underscores for special methods and keywords.

Variables are compared using their first character, then comparing further characters case-insensitively and ignoring all delimiters. This makes it easier for developers to use varying conventions without having to worry about the variables' exact spelling.

```dart
proc cmpIdent(a: Str, b: Str): Bool =>
  a[0] == b[0] &&
  a.sub(`[^\pL\d]+`g, "").lower() == b.sub(`[^\pL\d]+`g, "").lower();
```

The above rule does not apply to keywords, as all keywords are all-lowercase. Because of this rule, to strop keywords, add one or more trailing underscores.

Keywords lose meaning and become ordinary identifiers when they are part of the inner members of a qualified name, such as a function or method.

```dart
type Type = {
  def: Func,
};

val object_ = new Type({def: |x| x = 10});
assert object_ is Type;
assert object_.def == 9;

var var_ = 42;
val val_ = 8;
assert var_ + let_ == 50;

val assert_ = true;
assert assert_;
```

### Numbers

Trinity supports integers and floating-point numbers. Floats compile to regular JavaScript `number`s, [IEEE-754 double-precision floating-point][double] while integers compile to `bigint` (arbitrary-precision integers). Floats are typically distinguished between integers with a dot.

[double]: https://en.wikipedia.org/wiki/Double-precision_floating-point_format

```dart
val integer: Int = 123;
val floating: Float = 12.345;
```

[double]: https://en.wikipedia.org/wiki/Double-precision_floating-point_format

Numbers are case-insensitive including its type suffix, and can contain leading zeroes and underscores for readability. Integer and floating-point literals can be written in base 2, 4, 6, 8, 10, 12 or 16:

| Base | Name        | Prefix    | Digits                       |
| ---- | ----------- | --------- | ---------------------------- |
| 2    | Binary      | `0b`      | `0` and `1`                  |
| 4    | Quaternary  | `0q`      | `0` to `3`                   |
| 6    | Senary      | `0s`      | `0` to `5`                   |
| 8    | Octal       | `0o`      | `0` to `7`                   |
| 10   | Decimal     | no prefix | `0` to `9`                   |
| 12   | Duodecimal  | `0z`      | `0` to `9`, then `a` and `b` |
| 16   | Hexadecimal | `0x`      | `0` to `9` then `a` to `f`   |

```dart
val base2 = 0b101010111100000100100011;
val base4 = 0q320210213202;
val base6 = 0s125423;
val base8 = 0o52740443;
val base10 = 0011256099;
val base12 = 0z10a37b547ab97;
val base16 = 0xabcdef123;
```

Floating-point numbers can allow different kinds of delimiters and separators,

Repeating fractional blocks are separated with a tilde `~`, so `0.3~33` or simply `0.~3` is equal to `0.33333333333333...`. Fractional literals separate their numerator and denominator with a slash `/`.

```dart
0.3~33 == 0.~3 == 1/3
```

Exponents are relative to the base, but are written in base 10. Therefore `1 * 16^10` is equal to `0x1^10`. If you want a custom base, use the notation `coefficient*base^power`, where the power is signed.

```dart
1 * 16^10  == 0x1^10
```

Precision is delimited using `=n` where `n` is the number of places after the "decimal" point. `!` counts significant figures rather than mantisa digits, while `-` or `+` toggles whether to always round up or down as opposed to automatically.

```dart
10=10
```

There is a literal for every numerical type defined. Suffixes beginning with a backslash is called a _type suffix_. The backslash denoting the type suffix cannot be left out.

| Suffix  | Resultant Type | Equivalent C#/D Type |
| ------- | -------------- | -------------------- |
| `:i8`   | `I8`           | `sbyte`              |
| `:i16`  | `I16`          | `short`              |
| `:i32`  | `I32`          | `int`                |
| `:i64`  | `I64`          | `long`               |
| `:i128` | `I128`         | `cent`               |
| `:u8`   | `U8`           | `byte`               |
| `:u16`  | `U16`          | `ushort`             |
| `:u32`  | `U32`          | `uint`               |
| `:u64`  | `U64`          | `ulong`              |
| `:u128` | `U128`         | `ucent`              |
| `:f32`  | `F32`          | `float`              |
| `:f64`  | `F64`          | `double`             |
| `:f128` | `F128`         | `decimal`            |

Arbitrary bases can be used, beginning with `nb` where `n` is a positive integer greater than 1. The digits are usually decimal, though

### Booleans, Null and Void

`Null` and `void` compile to JavaScript `null` and `undefined`. In Trinity, null is equal to void by value, but not by reference.

```dart
null; void;
assert null == void
assert null !== void
```

A boolean data type can only have two values: `true` or `false`. Booleans are mainly used for control flow, and there are a lot of operators that return boolean values.

```dart
true; false;
```

All values default to an empty value, which means they yield `false` when converted into booleans. All other values, including non-primitive objects, yield true.

Boolean values also come as a result of comparisons, or other logical operations.

```dart
val isGreater = 4 > 1 // true
```

### Strings

Strings function the same way as in JavaScript, and are delimited by matching quotes. Only double-quoted strings contain escape sequences which all begin with a backslash. Single-quoted strings are raw, which means that escape sequences are not transformed.

```dart
var s1 = 'Single quotes work well for string literals.';
var s2 = "Double quotes work just as well.";
```

Single-quoted raw strings the escape sequences for double-quoted strings mentioned above are not escaped. To escape a single quote, double it.

```dart
var daughterOfTheVoid = 'Kai''Sa';
```

Double quoted string literals can contain the following escape sequences, and can contain the following escape sequences:

| Escape Sequence | Meaning                                        |
| --------------- | ---------------------------------------------- |
| `\p`            | platform specific newline (`\r\n`, `\n`, `\r`) |
| `\r`            | carriage return (`\x9`)                        |
| `\n`            | line feed (or newline) (`\xA`)                 |
| `\f`            | form feed (`\xC`)                              |
| `\t`            | horizontal tabulator (`\x9`)                   |
| `\v`            | vertical tabulator (`\xB`)                     |
| `\a`            | alert (`\x7`)                                  |
| `\b`            | backspace (`\x8`)                              |
| `\e`            | escape (`\xB`)                                 |
| `\s`            | space (`\x20`)                                 |

Trinity also supports escapes in even bases up to 16, excluding 14.

| Escape Sequence      | Meaning                                        |
| -------------------- | ---------------------------------------------- |
| `\b` (beside 0 or 1) | _Base 2_ - from `0` to `100001111111111111111` |
| `\q`                 | _Base 4_ - from `0` to `10033333333`           |
| `\s` (beside 0 to 5) | _Base 6_ - from `0` to `35513531`              |
| `\o`                 | _Base 8_ - from `0` to `4177777`               |
| `\d` or `\`          | _Base 10_ - from `0` to `1114111`              |
| `\z`                 | _Base 12_ - from `0` to `4588A7`               |
| `\x`                 | _Base 16_ - from `0` to `10FFFF`               |
| `\u`                 | UTF-8, 16 or 32 code units only                |
| `\j`                 | Named Unicode characters (more later)          |

The same escapes with curly brackets allow you to insert many code points inside, with each character or code unit separated by spaces. Only `\j` requires curly brackets.

```dart
// "HELLO"
"\x48\x45\x4c\x4c\x4f" == "\x{48 45 4c 4c 4f}";
"\d{72 69 76 76 69}" == "\72\69\76\76\79";
```

In single quoted strings, to escape single quotes, double them.

```dart
var s3 = 'It''s easy to escape the string delimiter.';
var s4 = "It's even easier to use the other delimiter.";
```

In double-quoted strings, an ending backslash joins the next line _without spaces_.

```dart
assert "hello \
        world" == "hello world";
```

#### Block strings

String literals can also be delimited by at least three single or double quotes, provided they end with _at least_ that many quotes of the same type.

The rules for single- and double-quoted strings also apply.

```dart
'''
  "stringified string"
'''
""" "stringified string""""
```

produces:

    "stringified string"

All newlines and whitespace before the first non-line character and after the last non-line character are discarded.

All indentation is determined based on the first line of text (the first non-whitespace character). All indentation after that column is preserved while those before it are discarded.

Newlines are normalized to `\n`.

```dart
'''
"stringified
  string"
''' ==
"""
  "stringified
    string"
"""
```

Any string that does not obey this rule is a compile-time error.

```dart
"""
  "stringified
string"
"""
```

#### Backslash strings

Strings can also be delimited using an initial backslash, and do not contain `()[]{}<>.,:;`, so those cannot be included at all in the string. However, you can escape them.

Strings cannot begin in `|`, `>`, `<` or `-`.

```dart
\word
func(\word, \word)
func\word
[\word]
{prop: \word}
```

Block strings begin with a backslash followed by either `|` or `>`, both functioning like block strings. `\|` behaves like the single quote and `\>` the double quote.

Block strings also begin with a backslash followed by either `|` or `>`, both functioning like `|` block strings. `\|` behaves like the single quote and `\>` the double quote.

They can also be appended with a "chomping indicator" `+` or `-` to preserve or remove the line feed, or fold the line past how many spaces.

```dart
\|
  this is my very very "very" long-ass string.
  Love, Trinity.
\>
  this is my very very "very" long-\
  ass string.\nLove, Trinity.
```

Trinity comes with several avenues to make manipulating, formatting and serializing strings easier.

#### String Interpolation

All forms of string literals, with exception to inline backslash strings, can enable embedding of arbitrary expressions. Embedded expressions are prefixed with the dollar and surrounded by curly brackets.

If the expression is an identifier or qualified name, then the brackets can be left out. Use the `\$` escape sequence if you wish to express the dollar sign itself.

```dart
"x is $x, in hex $x.toHex, and x+8 is ${x + 8}"
```

is syntax sugar for:

```dart
"x is " + x + ", in hex " + x.toHex + ", and x+8 is " + (x + 8)
```

The hash sign takes several arguments, as placeholders, passed to the `format` method. Arguments can either be named, numbered or keyed.

```dart
'#0%s is #1 meters tall'.format('James', 1.9)
// "James is 1.9 meters tall"
```

#### Macro Strings

Macro strings are used to embed domain-specific languages directly into Trinity, and are functionally the same as JS tagged template literals.

The construct `qualifiedName"string"` denotes a macro call, with a string literal as its only argument. Macro string literals are especially convenient for embedding DSLs directly into Trinity (for example, SQL), then parsing them and doing things with them.

A macro function is defined with the keyword `macro` rather than `fun`, `sub` or `proc`. Macros are functions with up to three arguments. The first argument of a tagged function contains a list of intermediate strings, the second are related to the interpolated values themselves, and the third the formatted result.

```dart
macro template(strings, keys) = |*values| {
  let dict = values[-1] ?? {};
  let values = (from let key in keys
    select if key is Int => values[key]
    else => dict[key]) as List
  return strings.intercalate(keys).join('')
}

let t1Closure = template"${0}${1}${0}!"
assert t1Closure("Y", "A") == "YAY!"
let t2Closure = template"${0} ${"foo"}!"
assert t2Closure("Hello", {foo: "World"}) == "Hello World!"
```

#### Format Directives

Trinity also provides an extensive format specifier mini-language for transforming, converting serializing and translating strings, taking its inspiration from Command Prompt.

They look like this: `%float/sci/pow:32/sf:3`. A command immediately after the percent sign, followed by an optional range of switches `/sw` and their optional values `:val`.

```dart
const prices = { bread: 4.50 }
'I like bread. It costs $prices.bread%f/cur:SGD.'
// "I like bread. It costs $4.50."
```

### Regular expressions

Regular expressions function much like strings, except that they are delimited using backticks ` `` ` as opposed to quotes. They allow free spacing and comments, and therefore, spaces and comments are removed when they compile.

Escaping rules apply, though in between `()` or `[]`, the backtick `` ` `` itself need not be escaped. Interpolation and formatting also applies but the interpolated result is usually escaped (quoted) so to prevent generating invalid regular expressions.

```dart
`\b{wb}(fee|fie|foe|fum)\b{wb}`x
`[ ! @ " $ % ^ & * () = ? <> ' : {} \[ \] `]`x
`
  \/\* // Match the opening delimiter.
  .*?  // Match a minimal number of characters.
  \*\/ // Match the closing delimiter.
`
```

Multi-quoted and block regular expressions are also supported.

```dart
\< x // global flag
  (?x)\s*
  (\\\|)\s*
  ((?:\w|\\.)+(?:(?:[^\s'"`\\\[\](){}<>]|\\.)*(?:\w|\\.)+)?)?\s*
  (.*$)?
```

If there are two regular expressions side by side, then the one on the right is the replacement string attributed to the pattern on the left.

```dart
val str = 'James Bond'
val newStr = str =< `(\w+)\W+(\w+)` `$2, $1` // 'Bond, James'
val newStr = str =< `(\w+)\W+(\w+)` `My name is $2, $0!`
// 'My name is Bond, James Bond'
```

The following section serves as a summary to the regular expression syntax of Trinity, as well as some of the more unique features that Nova has over other regex flavors.

#### Basic Syntax Elements

| Syntax      | Description                                     |
| ----------- | ----------------------------------------------- |
| `\`         | Escape (disable) a metacharacter                |
| `\|`        | Alternation                                     |
| `/`         | Alternation: try out matches in the given order |
| `(...)`     | Capturing group                                 |
| `[...]`     | Character class (can be nested)                 |
| `${...}`    | Embedded expression                             |
| `{,}`       | Quantifier token (LHS 0, RHS &infin;)           |
| `"..."`     | Raw quoted literal                              |
| `'...'`     | Quoted literal                                  |
| `\0` onward | Numeric back-reference (0-indexed)              |
| `$...%...`  | String interpolation syntax                     |
| `#...`      | String anchor syntax                            |

#### Characters

Most of these characters also appear the same way as in string literals.

| Syntax                         | Description and Use                     |
| ------------------------------ | --------------------------------------- |
| `\a`                           | \*Alert/bell character (inside `[]`)    |
| `\b`                           | \*Backspace character (inside `[]`)     |
| `\e`                           | Escape character (Unicode `U+`)         |
| `\f`                           | Form feed (Unicode `U+`)                |
| `\n`                           | New line (Unicode `U+`)                 |
| `\r`                           | Carriage return (Unicode `U+`)          |
| `\t`                           | Horizontal tab (Unicode `U+`)           |
| `\v`                           | Vertical tab (Unicode `U+`)             |
| `\cA`...`\cZ`<br>`\ca`...`\cz` | Control character from `U+01` to `U+1A` |

The following can only be used inside square brackets.

| Syntax               | Description and Use                            |
| -------------------- | ---------------------------------------------- |
| `\b` (beside 0 or 1) | _Base 2_ - from `0` to `100001111111111111111` |
| `\q`                 | _Base 4_ - from `0` to `10033333333`           |
| `\s` (beside 0 to 5) | _Base 6_ - from `0` to `35513531`              |
| `\o`                 | _Base 8_ - from `0` to `4177777`               |
| `\d` or `\`          | _Base 10_ - from `0` to `1114111`              |
| `\z`                 | _Base 12_ - from `0` to `4588A7`               |
| `\x`                 | _Base 16_ - from `0` to `10FFFF`               |

#### Character Sequences

Character sequences in regular expressions are the same as in their string counterparts, with exception to `\b{}` outside `[]`.

#### Character Classes and Sequences

| Syntax | Inverse | Description                                                       |
| ------ | ------- | ----------------------------------------------------------------- |
| `.`    | None    | Hexadecimal code point (1-8 digits)                               |
| `\w`   | `\W`    | Word character `[\d]`                                             |
| `\d`   | `\D`    | Digit character `[0-9]`                                           |
| `\s`   | `\S`    | Space character `[\t\n\v\f\r\20]`                                 |
| `\h`   | `\H`    | Hexadecimal digit character `[\da-fA-F]`                          |
| `\u`   | `\U`    | Uppercase letter `[A-Z]`                                          |
| `\l`   | `\L`    | Lowercase letter `[a-z]`                                          |
| `\f`   | `\F`    | Form feed `[\f]`                                                  |
| `\t`   | `\T`    | Horizontal tab `[\t]`                                             |
| `\v`   | `\V`    | Form feed `[\v]`                                                  |
| `\n`   | `\N`    | Newline `[\n]`                                                    |
|        | `\O`    | Any character `[^]`                                               |
| `\R`   |         | General line break (CR + LF, etc)                                 |
| `\c`   | `\C`    | First character of identifier; `[\pL\pPc]` by default             |
| `\i`   | `\I`    | Subsequent characters of identifier `[\pL\pPc\pM\pNd]` by default |
| `\x`   | `\X`    | Extended grapheme cluster                                         |

##### Unicode Properties

Properties are case-insensitive. Logical operators such as `&&`, `||`, `^^` and `!`, as well as `==` and `!=`, unary `in` and `!in` , `is` and `!is` can work.

A short form starting with `Is` indicates a script or binary property:

- `is Latin`, &rarr; `Script=Latin`.
- `is Alphabetic`, &rarr; `Alphabetic=Yes`.

A short form starting with `In` indicates a block property:

- `InBasicLatin`, &rarr; `Block=BasicLatin` .
- `\p{in Alphabetic && is Latin}` &rarr; all Latin characters in Unicode

| Syntax                                                                | Description                      |
| --------------------------------------------------------------------- | -------------------------------- |
| `\p{property=value}`<br>`\p{property:value}`<br>`\p{property==value}` | Unicode binary property          |
| `\p{property!=value}`<br>`\P{property:value}`                         | Negated binary property          |
| `\p{in BasicLatin}`<br>`\P{!in BasicLatin}`                           | Block property                   |
| `\p{is Latin}`<br>`\p{script==Latin}`                                 | Script property (shorthand `is`) |
| `\p{value}`                                                           | Short form\*                     |
| `\p{Cc}`                                                              | Unicode character categories^    |

\*Properties are checked in the order: `General_Category`, `Script`, `Block`, binary property:

- `Latin` &rarr; (`Script=Latin`).
- `BasicLatin` &rarr; (`Block=BasicLatin`).
- `Alphabetic` &rarr; (`Alphabetic=Yes`).

##### POSIX Classes

Alternatively, `\p{}` notation can be used instead of `[:]`.

| Syntax      | ASCII                                        | Unicode (`/u` flag) | Description                                              |
| ----------- | -------------------------------------------- | ------------------- | -------------------------------------------------------- |
| `[:alnum]`  | `[a-zA-Z0-9]`                                | `[\pL\pNl}\pNd]`    | Alphanumeric characters                                  |
| `[:alpha]`  | `[a-zA-Z]`                                   | `[\pL\pNl]`         | Alphabetic characters                                    |
| `[:ascii]`  | `[\x00-\x7F]`                                | `[\x00-\xFF]`       | ASCII characters                                         |
| `[:blank]`  | `[\x20\t]`                                   | `[\pZs\t]`          | Space and tab                                            |
| `[:cntrl]`  | `[\x00-\x1F\x7F]`                            | `\pCc`              | Control characters                                       |
| `[:digit]`  | `[0-9]`                                      | `\pNd`              | Digits                                                   |
| `[:graph]`  | `[\x21-\x7E]`                                | `[^\pZ\pC]`         | Visible characters (anything except spaces and controls) |
| `[:lower]`  | `[a-z]`                                      | `\pLl`              | Lowercase letters                                        |
| `[:number]` | `[0-9]`                                      | `\pN`               | Numeric characters                                       |
| `[:print]`  | `[\x20-\x7E] `                               | `\PC`               | Printable characters (anything except controls)          |
| `[:punct]`  | `[!"\#$%&'()\*+,\-./:;<=>?@\[\\\]^\_'{\|}~]` | `\pP`               | Punctuation (and symbols).                               |
| `[:space]`  | `[\pS\t\r\n\v\f]`                            | `[\pZ\t\r\n\v\f]`   | Spacing characters                                       |
| `[:symbol]` | `[\pS&&[:ascii]]`                            | `\pS`               | Symbols                                                  |
| `[:upper]`  | `[A-Z]`                                      | `\pLu`              | Uppercase letters                                        |
| `[:word]`   | `[A-Za-z0-9_]`                               | `[\pL\pNl\pNd\pPc]` | Word characters                                          |
| `[:xdigit]` | `[A-Fa-f0-9] `                               | `[A-Fa-f0-9]`       | Hexadecimal digits                                       |

#### Character Sets

A set `[...]` can include nested sets. The operators below are listed in increasing precedence, meaning they are evaluated first.

| Syntax                 | Description                                                    |
| ---------------------- | -------------------------------------------------------------- |
| `^...`, `~...`, `!...` | Negated (complement) character class                           |
| `x-y`                  | Range (from x to y)                                            |
| `\|\|`                 | Union (`x \|\| y` &rArr; "x or y")                             |
| `&&`                   | Intersection (`x && y` &rArr; "x and y" )                      |
| `^^`                   | Symmetric difference (`x ^^ y` &rArr; "x and y, but not both") |
| `--`                   | Difference (`x ~~ y` &rArr; "x but not y")                     |

#### Anchors

| Syntax | Inverse | Description                                  |
| ------ | ------- | -------------------------------------------- |
| `^`    | None    | Beginning of the string/line                 |
| `$`    | None    | End of the string/line                       |
| `\b`   | `\B`    | Word boundary                                |
| `\a`   | `\A`    | Beginning of the string/line                 |
| `\z`   | `\Z`    | End of the string/before new line            |
|        | `\G`    | Where the current search attempt begins/ends |
|        | `\K`    | Keep start/end position of the result string |
| `\m`   | `\M`    | Line boundary                                |
| `\y`   | `\Y`    | Text segment boundary                        |

#### Quantifiers

| Syntax           | Reluctant `?` (returns shortest match) | Possessive `+` (returns nothing) | Greedy `*` (returns longest match) | Description                             |
| ---------------- | -------------------------------------- | -------------------------------- | ---------------------------------- | --------------------------------------- |
| `?`              | `??`                                   | `?+`                             | `?*`                               | 1 or 0 times                            |
| `+`              | `+?`                                   | `++`                             | `+*`                               | 1 or more times                         |
| `*`, `{,}`, `{}` | `*?`, `{,}?`, `{}?`                    | `*+`, `{,}+`, `{}+`              | `**`, `{,}*`, `{}*`                | 0 or more times                         |
| `{n,m}`          | `{n,m}?`                               | `{n,m}+`                         | `{n,m}*`                           | At least `n` but no more than `m` times |
| `{n,}`           | `{n,}?`                                | `{n,}+`                          | `{n,}*`                            | At least `n` times                      |
| `{,m}`           | `{,m}?`                                | `{,m}+`                          | `{,m}*`                            | Up to `m` times                         |
| `{n}`            | `{n}?`                                 | `{n}+`                           | `{n}*`                             | Exactly `n` times                       |

#### Groups

`(?'')`, `(?"")` notation can also be used.

| Syntax                      | Description                       |
| --------------------------- | --------------------------------- |
| `(?#...)`                   | Comment                           |
| `(?x-y:...)`<br>`(?x-y)...` | Mode modifier                     |
| `(?:...)`                   | Non-capturing (passive) group     |
| `(...)`                     | Capturing group (numbered from 1) |
| `(?<name>...)`              | Named capturing group             |
| `(?<-x>...)`                | Balancing group                   |
| `(?<x-x>...)`               | Balancing group pair              |
| `(?=...)`                   | Positive lookahead                |
| `(?!...)`                   | Negative lookahead                |
| `(?<=...)`                  | Positive lookbehind               |
| `(?<!...)`                  | Negative lookbehind               |
| `(?>...)`                   | Atomic group (no backtracking)    |
| `(?~...)`                   | Sub-expression                    |
| `(?()\|...\|...)`           | Conditional branching             |
| `(?~\|...\|...)`            | Absent expression                 |
| `(?~\|...)`                 | Absent repeater                   |
| `(?~...)`                   | Absent stopper                    |
| `(?~\|)`                    | Range clear                       |

#### Backreferences and Calls

`\k''`, `\k""` can also be used.

| Syntax     | Description                                               |
| ---------- | --------------------------------------------------------- |
| `\1`       | Specific numbered backreference                           |
| `\k<1>`    | Specific numbered backreference                           |
| `\k<-1>`   | Relative numbered backreference (`+` ahead, `-` behind)   |
| `\k<name>` | Specific named backreference                              |
| `\g<1>`    | Specific numbered subroutine call                         |
| `\g<-1>`   | Relative numbered subroutine call (`+` ahead, `-` behind) |
| `\g<name>` | Specific named subroutine call                            |

#### Flags

These flags go after the regex literal. `f`, `m`, `u`, `e` and `x` are enabled by default.

| Flag | Description                                                                  |
| ---- | ---------------------------------------------------------------------------- |
| `a`  | Astral mode - `\p` supports the past the BMP                                 |
| `c`  | Case-sensitive mode.                                                         |
| `d`  | Treat only `\n` as a line break                                              |
| `e`  | Safe mode - escape all interpolations                                        |
| `f`  | First match only                                                             |
| `g`  | Global. Enabled by default                                                   |
| `i`  | Case-insensitive mode                                                        |
| `j`  | Switches definitions of `()` and `(?:)`                                      |
| `k`  | Allows duplicate named groups                                                |
| `l`  | Last match only                                                              |
| `m`  | Multiline - `^`/`$` match at every line                                      |
| `n`  | Named capturing groups only - all unnamed groups become non-capturing        |
| `o`  | Unsafe mode - coerces interpolations into strings                            |
| `p`  | `^` and `$` match at the start/end of line                                   |
| `q`  | Quote all metacharacters                                                     |
| `s`  | "Dot-all" - `.` matches all characters                                       |
| `t`  | Strict spacing mode                                                          |
| `u`  | Unicode mode - POSIX class definitions also expanded                         |
| `w`  | `^` and `$` match at the start/end of string, `.` does not match line breaks |
| `x`  | Free-spacing mode                                                            |
| `y`  | Sticky mode - search begins from specified index on LHS of regex             |

#### Replacement String

This syntax applies to the right hand side of the regex literal in regex operations such as `=<` substitution and `</>` transliteration.

| x         | y                                                                                                                                                      |
| --------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `$$`      | Inserts a literal "$".                                                                                                                                 |
| `$0`      | Inserts the entire matched substring into the output                                                                                                   |
| `$-`      | Inserts the portion of the string that precedes the matched substring.                                                                                 |
| `$+`      | Inserts the portion of the string that follows the matched substring.                                                                                  |
| `$n`      | Where `n` is a positive integer, inserts the `n`th parenthesized submatch string. If `n` refers to an invalid group, the result is inserted literally. |
| `$<name>` | Where name is a capturing group name. If the group is invalid, it is inserted literally.                                                               |

### Operators

Trinity allows user defined operators. An operator is any punctuation `P` or symbol `S` character, except `` ,;\'"`()[]{} ``. These keywords are also operators: `in of as is new to til thru by unset del`.

`.=`, `:`, `=`, `:=`, `? :`, `! :` and `$ :` are not available as general operators; they are used for other notational purposes. `=>` is a special case, as it is syntactic sugar for `then` and introduces a block.

### Other tokens

The following strings denote other tokens: `'` `"` `` ` `` `#(` `(` `)` `#{` `{` `}` `#[` `[` `]` `,` `;` `:` `=>`

## Syntax

This section describes the syntax for Trinity.

### Operators

Trinity allows user defined operators with a combination of two declarative keywords: one which tells the parser the arity of the operator, and the keyword `oper`. Both must be before a declaration keyword, such as `fun`, `def`, `proc` or `sub`.

```dart
infix oper fun x = 10
```

Nim allows user-definable operators. Binary operators have 11 different levels of precedence.

### Associativity

Binary operators whose first character is `@` are right-associative, all other binary operators are left-associative.

```dart
infix fun + (x, y: Float): Float = x / y
// A right-associative division operator
result = x / y
echo(12 @/ 4 @/ 8) // 24.0 (4 / 8 = 0.5, then 12 / 0.5 = 24.0)
echo(12 / 4 / 8) // 0.375 (12 / 4 = 3.0, then 3 / 8 = 0.375)
```

### Precedence

Prefix operators always bind stronger than any binary operator: `$a + b` is `($a) + b` and not `$(a + b)`.

If an Prefix operator's first character is `@` it is a sigil-like operator which binds stronger than a leading identifier: `@x.abc` is parsed as `(@x).abc` whereas `$x.abc` is parsed as `$(x.abc)`.

For binary operators that are not keywords, the precedence is determined by the following rules: Operators ending in either `->`, `~>` or `=>` are called arrow like, and have the lowest precedence of all operators.

If the operator ends with `=` and its first character is none of `<`, `>`, `!`, `=`, `~`, `?`, it is an assignment operator which has the second-lowest precedence.

Otherwise, precedence is determined by the first character.

| Precedence level | Operators                                                                                         | First Character | Terminal Symbol |
| ---------------- | ------------------------------------------------------------------------------------------------- | --------------- | --------------- |
| 10 (highest)     | `~@` `::` `:-` `-~` `~-` `.-` `@@` `$$`                                                           | `$` `@`         | `BinaryOper10`  |
| 9                | `*` `**` `***` `/` `#` `##` `%` `%%` `*>` `<*`                                                    | `%` `*` `/`     | `BinaryOper9`   |
| 8                | `+` `-` `++` `--` `=<` `<>` `</` `/>` `<$` `$>` `<$>` `<+>` `<*>` `</>`                           | `+` `-`         | `BinaryOper8`   |
| 7                | `<:` `:>` `:<` `>:` `<!` `!>` `!<` `>!` `::` `..` `:::` `...` `..<` `>.<` `>..`                   | `.` `:`         | `BinaryOper7`   |
| 6                | `==` `!=` `===` `!==` `~>` `<~` `~~>` `<~~` `<==` `==>` `->` `-->` `<-` `<--` `<~>` `<==>` `<-->` | `=` `!` `>` `<` | `BinaryOper6`   |
| 5                | `&` `^` `\|` `>>` `<<` `>>>` `<<<` `<=>` `<->`                                                    | `?`             | `BinaryOper5`   |
| 4                | `??` `!!` `?:` `!:` `$:` `&` `\|` `^` `~&` `~\|` `~^`                                             | `~`             | `BinaryOper4`   |
| 3                | `&&` `\|\|` `^^` `&~` `\|~` `^~`                                                                  | `#`             | `BinaryOper3`   |
| 2                | `<+` `+>` `<\|` `\|>` `<\|\|` `\|\|>` `<\|\|\|` `\|\|\|>`                                         | `?`             | `BinaryOper2`   |
| 1 (lowest)       | `=` `:=` `+=` `*=` etc; other assignment operators                                                |                 | `BinaryOper1`   |
| 0                | `? :` `! :` `$ :`                                                                                 |                 | `TernaryOper`   |

Whitespace also affects operator parsing. Spacing also determines whether `(a, b)` is parsed as an argument list of a call or whether it is parsed as a tuple constructor.

| Type of operator | Whitespace               | Precedence, Associativity    | Characters |
| ---------------- | ------------------------ | ---------------------------- | ---------- |
| Primary          | None on either end       | Highest; None                | Multiple   |
| Suffix           | Trailing                 | Higher; Left to right        | Single     |
| Prefix           | Leading                  | Lower; Right to left         | Single     |
| Infix            | Spaced out on either end | Lowest; Refer to table above | Multiple   |

```dart
foo |> echo
```
