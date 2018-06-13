# UON Unified Object Notation (UON ™ )

## Draft Version 0.0.1

**This version:** http://github.com/uon-language/specification/blob/spec.md

**Bug tracker**: http://github.com/uon-language/specification/issues

**Editor**: Yves Chevallier <ycr@x0x.ch>

Copyright(C) 2018 Yves Chevallier

## Introduction

In 2018 three major serialization formats are commonly used across Internet: `XML`, `JSON` and `YAML`. Both are great, simple and well specified, but many would argue that: 

* `XML` is great but cumbersome 
* `JSON` is awesome but lack validation by design
* `YAML` is human-friendly but not great for m2m communication

Seen from another perspective:

* `xml` is fully specified with validation schema `XSD` and representation `XSLT`
* `JSON` is lightweight and understood by most developers and programming languages
* `YAML` is very readable and a superset of `JSON`

*UON™* aims to steal all the amazing features of these serialization format into a single format that encompass them.

Also, *UON* would like to go a step further by adding support for validation schema and magnitude units in order to extend interoperability for the Industry 4.0. Internet of Things often have to communicate between small devices with very limited computation power. 

## Scope

*UON*™ pronounced "You On" is a human-machine friendly serialization language designed to serve the purpose of:

* Transmitting data across digital mediums
* Being human readable and human editable
* Being close to international standards and largest consensus
* Serializing/deserializing data across different programming languages
* Unambiguous strict representation by design
* Support references and merge
* Support properties
* Standard API
* Zero-copy operations
* Validation schema supported by design (Inspired from [JSON-Schema](http://json-schema.org/) and [Voluptuous](https://pypi.org/project/voluptuous/))
* Transformation schema supported by design (Inspired from what `XSLT` is to `XML`)
* Platform neutral

## Normative references 

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED",  "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119.

## Terms and definitions

For the purposes of this document, the following terms and definitions apply.

* **RFC** Request for comments
* **UON**  Unified Object Notation, specified in this document
* **ISO** International Standardization Organization
* **YAML** Yet Another Markup Language
* **JSON** JavaScript Object Notation

## Language

 *UON*™ is not a programming language, but a data representation language. It is built upon concepts described by C, Perl, Python, Ruby and JSON. It is meant to be UTF-8 by default, but compatible with other encoding standards. 

![](C:\Users\ycr\Data\specification\assets\complexity-levels.png)

### Related to JSON

Both JSON and UON aim to be human readable data interchange formats. UON extends JSON capabilities and readability with: 

* More datatypes support
* Allow no double quotes on mapping keys since they are promoted to a `keyword` type rather than `string`
* Validation schema by design inspired from JSON-Schema 
* Allow type coercion 
* Add support for comments
* Add support for set, and ordered mapping

### Related to YAML

YAML is very complete, it supports references (recursive references), comments, complex types, but it syntax lack of rigidity. They are multiple way of writing a YAML which can be confusing to some people and hard to validate. UON extends YAML capabilities by adding: 

* More datatype support
* Strict formatting closer to JSON
* Validation schema by design 
* Resolved and unresolved references
* Unambiguous comment position

In YAML it is possible to create a reference to an existing location in the tree. However this always resolved after parsing which mean, the reference information is lost. UON supports unresolved references in which even after parsing the document, the reference is preserved. 

Comments in YAML are not formally part of the object tree, but they are somehow supported by `ruamel.yaml` Python package with the `round_trip_load` method. In *UON*, comments are part of the language, and they appear in the object tree.

![](C:\Users\ycr\Data\specification\assets\supersets.png)

### Related to XML

XML has proven its strength over many years. It is still a very complete language, but it is also more verbose than other serialization languages which make it difficult to use with lightweight infrastructures such as MCU and battery powered telemetry devices. However it provides a full support for validation (XSD) and presentation (XSLT). UON took the tags attributes that is missing from JSON and YAML.

## Related to (Google) Protocol Buffers

Protocol buffers is a relatively new standard developed by Google which allow binary payload generation. So it defined some advanced types. platform neutral serialization language developed by Google.

*UON* got inspired from Protocol Buffers by the following

* `repeated`, Fields can be repeated any number of times
* Fixed size scalar values e.g. `int32`, `uint64`, `sint32`
* Enumerations
* One-of keyword 

### Symbol Pairs

Symbol pairs are syntactic sugars used to enhance readability by associating a specific symbol pair to a particular type of data.  

| Symbol Pairs | Description           | Type     |
| ------------ | --------------------- | -------- |
| `[ ]`        | Ordered Sequence      | `!!seq`   |
| `{ }`        | Mapping (by default unordered) | `!!map`   |
| `" "`        | String                | `!!str`   |
| `/ /`        | Regular expression    | `!!regex` |
| `$ $`        | Mathematic expression | `!!math`  |
| `( )`        | Properties            | -        |
| `< >`        | Structural container  | -        |

Structural container are not meant to be directly used, but they can show how data are imbricated. 

```yaml
!!uon<version: !!version<0.0.1>> 
<
  !!map<comment: <Basic Example>>
  <
    <!!keyword<brand>: !!str<Toyota>>,
    <!!keyword<model>: !!str<Prius>>,
    <!!keyword<year>: !!int<2016>>
  >
>
```

The above example should be written as follow in structured format:

```yaml
# Basic Example
{
  brand: "Toyota",
  model: "Prius",
  year: 2016  
}
```

Or in the minimal version:

```yaml
{brand:"Toyota",model:"Prius",year:"2016"}
```

### Symbols

The only reserved language symbols are the following

| Symbol  | Description                                                  |
| ------- | ------------------------------------------------------------ |
| `!!`    | Standard type (cannot be defined at the user level)          |
| `!`     | User type used to extend *UON* capabilities                  |
| `$`     | Reserved keyword (may be removed on a future *UON* release)  |
| `,`     | Element separator                                            |
| `@`     | Reference                                                    |
| `#`     | Comment                                                      |
| `.inf`  | Infinity (used within `!!number`, `!!float`, or `!!decimal`)    |
| `.nan`  | Not a Number (used within `!!number`, `!!float`, or `!!decimal`) |
| `true`  | Boolean (`!!bool`) value `true`                               |
| `false` | Boolean (`!!bool`) value `false`                              |
| `null`  | Null (`!!null`) value                                         |

## Syntax

![](C:\Users\ycr\Data\specification\assets\syntax.png)

## Example

### General overview

```yaml
# Overview of what UON is capable of
!!uon(version: 0.0.1) {
  # Mapping, a dictionary of key-value pairs with unique keys
  map: {
    key-1: "A value",
    key-2: "Another value"
  },
  # Ordered mapping, but keys are unique
  omap: !!omap {
    foo: 1, 
    bar: 2
  }, 
  # Sequences
  seq: [1, 2, 3],
  # Unordered set of unique values
  set: {"foo", "bar"},
  # Ordered set of unique values
  oset: !!oset {"bar", "foo"},
  # Scalars
  scalar: {
    bool: false,
    string: {
      regular: "A regular string encoded in UTF-8",
      raw: r"A raw string without special chars",
      joined: j"Joined multiline string
      		    Where leading spaces and new lines are removed",
      aligned: m"Multiline string where new lines are preserved"
    },
    number: {
      decimal: -1432879.09,
      int: 1408,
      hex: 0x24321,
      oct: 0o19283,
      bin: 0b10001,
      float: [12.132e6, -.inf, .nan]
      complex: 12+4j,
      quaternion: 12+12i+23j+43k,
      quantity: {
         length: 12m,
         information: 120Gio
      }
    },
    regex: /fo+(bar)?/i
  },
  # Properties
  properties: {
      people(id: 1243): "Robert Ford" 
  },
  # References
  reference: {
  	ref: @(scalar.number.decimal),
  	merge: !!map(merge: @scalar.number) {
       int: 1409 # Override existing value
  	}
  },
  # Rich types
  rich: {
     uuid: !uuid "82584ce5-d086-41ff-978f-57323ebf5b9d",
     jwt: !jwt "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpYXQiOjF9.9GeY7G6YbEh90kFhyhqsL29uFQCbIlk2Y-YVhae61g0",
     base64: !base64 "W55IGNhcm5hbCBwbGVhc3VyZS4="
  }
  # Validation schema
  schema: !!schema("http://schema.org/Book") {
    abridged: !!bool,
    bookEdition: !!str,
    bookFormat: !@("http://schema.org/Person"),
    isbn(optional: true): !@("urn:uon:isbn"),
    numberOfPages: !!uint32(
    	description: "Represent the number of page of a book"
    	min: 1
    ),
    text(dummy: true): !!str(
      description: "This dummy field can be used as a reference by other fields"  
    ),
    about: !@(.dummy),
    accessMode: !@(.dummy)
    genre: !!oneof("Male", "Female"),
    link: !!oneof(!url, !urn, !uri)    
  },
  
  # Schema Type
  !!str: !!schema("http://uon.io/str") !!type(
    description: """
       Base string encoded in UTF-8
    """t,
    properties: {
      min(optional: true): !!uint(default: 0),
      max(optional: true): !!uint(default: .inf), greater-than: @.min),
      pattern(optional: true): !!regex,
    }
  )
}
```

### Other examples

```yaml
# Basic UON example
!!uon(version: 0.0.1) !("urn:car") {
  brand: "Toyota",
  model: "Prius",
  year: 2016
},

# Type description for a !car, and validation schema
!!uon(version: 0.0.1) !!schema("urn:car") {
  brand: !!str(pattern: /[A-Z]\w+/),
  model: !!str(pattern: /[A-Z]\w+/),
  year(required: false): !!dec(min: 1930)
  color(required: false): !!oneof(
     !!str,
     !!number
  )
}

# Coersion is implicit, the parsed result is `12.`
!!uon !!float !!str 12
```

## Comments

Inline comments with `#`are never serialized when transmitted unless explicitly mentioned. Comment with `#` are a syntactic sugar to fill the `comment` property of the related object. A comment is associated to the key-pair value.

```yaml
# Comment global to the document
{
  # Comment associated to the key-pair { brand: "Toyota" }
  brand: "Toyota",
  model: "Prius"
  # Comment associated to the key-pair { model: "Prius" }
  ,
  data: [
     # Comment associated to the value 23
     23, # Comment associated to the value 43
     43
     # Comment associated to the value 43
     ,     
     
     # Comment associated to the ... fuck this is not consistent...
  ]
},
```

If requested, the comment can be transmitted along with the data. They are therefore encoded as a property of the associated element. Thus the following is equivalent: 

```yaml
!!map(comment: "Comment global to the document")
{  
  !!keyword(comment: "Comment associated to the key-pair { brand: "Toyota" }") brand: "Toyota",
}
```

## Formatting

*UON* provide three different format depending on the usage: 

* **minimal** for m2m communication in UTF-8
* **binary** for m2m communication with lightweight architectures
* **structured** for storing, visualizing and editing

### Minimal

In minimal format:

* Spaced and new lines are removed
* Comments are removed unless configured differently 

```yaml
!!uon(version:0.0.1)!car{brand:"Toyota",model:"Prius",year:2016}
```

### Binary

* Payload is encoded in a strict binary form
* LED128 format for numbers
* Padding is preserved as much as possible to allows zero-copy payload read/alter

```yaml
<tbd>
```

### Structured

* Format uses two space indentation, same as JSON and YAML, but strictly imposed by design
* Explicit types are converted into implicit ones if not necessary
* Comments are written using the `#` notation

```yaml
# This is a car...
!!uon(version: 0.0.1) !car {
  brand: "Toyota"
  model: "Prius"
  year: 2016
}
```

## Datatypes

One important feature of *UON* is that any kind of information belongs to a type. Types are organized in different categories: 

### Data Structures (or Structural types)

![](C:\Users\ycr\Data\specification\assets\type-base.svg)

They are primitive data types all derived from the `!!type` master datatype. 

| Type      | Based on | Description                                                |
| --------- | -------- | ---------------------------------------------------------- |
| `!!type`   | -        | Base type                                                  |
| `!!map`    | `!!type`  | Unordered Mapping (also called *HashMap* or *Dictionary* ) |
| `!!omap`   | `!!type`  | Ordered Mapping                                            |
| `!!seq`    | `!!type`  | Ordered Sequence (also called *List* or *Array*)           |
| `!!scalar` | `!!type`  | Scalar value representable with a string                   |
| `!!set`    | `!!map`   | Unordered set of values                                    |
| `!!oset`   | `!!omap`  | Ordered set of values                                      |

*UON* aims to represent static data which are therefore immutable by design. This explain the absence of Records or Tuples. However, when a composite type is used as a key, it will be parsed as an immutable type if the destination language allows it.  

```python
>>> # Example in Python
>>> uon.loads("{[1,2]:[1,2]}")
{(1,2):[1,2]}
```

Unordered mapping (`!!map`, `!!set`) are always sorted naturally when serialized e.g. `[1, 2, 10, 100, a]`

Sets are mapping with null values 

### Scalar

Scalar values are any kind of values that may be represented as a string

| Type       | Based on  | Description                                                  |
| ---------- | --------- | ------------------------------------------------------------ |
| `!!null`    | `!!scalar` | Null type, can be associated to other type e.g. `!!null(!!str)` |
| `!!bool`    | `!!scalar` | Boolean, `true` or `false`                                   |
| `!!str`     | `!!scalar` | String encoded in `UTF-8`                                    |
| `!!number`  | `!!scalar` | Any kind of representable numeric value                      |
| `!!keyword` | `!!str`    | Keyword used for mapping keys and references (`/^[a-z](?:(?:[a-z0-9-])*[a-z0-9])?$/i`) |
| `!!ref`     | `!!str`    | Reference to another type                                    |

#### Numbers Datatypes

The number type is much more complete than JSON, XML or YAML. It aims to serve goals of engineering and scientific areas such as: 

* Physics and Mathematics
  * Quantities with uncertainties and units
  * Complex and Quaternions
* Computer science 
  * Low level representation (hexadecimal, octal and binary)
  * Fractional values for frequency ratio
  * Fixed-Point values in 

| Type         | Based on  | Description                                                  |
| ------------ | --------- | ------------------------------------------------------------ |
| `!!dec`       | `!!number` | Decimal (i.e. Base 10) number e.g. `12345`                   |
| `!!float`     | `!!number` | Floating point `IEEE-754`                                    |
| `!!bin`       | `!!dec`    | Binary representation of decimal number                      |
| `!!oct`       | `!!dec`    | Octal representation of decimal number                       |
| `!!hex`       | `!!dec`    | Hexadecimal representation of decimal number                 |
| `!!complex`   | `!!float`  | Complex value e.g. `42+7j`                                   |
| `!!magnitude` | `!!number` | Physical value with an associated unit                       |
| `!!frac`      | `!!seq`    | Fraction of two integers, to represent repeating such as `0.33333` |
| `!!fixpoint`  | `!!dec`    | Fixed point value                                            |

### Sized Datatypes

When serialized in binary format, the size of the payload has to be provided. If not, the serializing will use a 7-bit encoding where the 8th bit is used as a continuation bit formerly named [LEB128](https://en.wikipedia.org/wiki/LEB128). Variable size numeric values are less efficient at serializing and deserializing.  

| Type         | Based on | Description                                              |
| ------------ | -------- | -------------------------------------------------------- |
| `!!int8`      | `!!dec`   | `8 b` Signed Integer                                     |
| `!!int16`     | `!!dec`   | `16 b` Signed Integer                                    |
| `!!int32`     | `!!dec`   | `32 b` Signed Integer                                    |
| `!!int64`     | `!!dec`   | `64 b` Signed Integer                                    |
| `!!int128`    | `!!dec`   | `128 b` Signed Integer                                   |
| `!!uint8`     | `!!dec`   | `8 b` Unsigned Integer                                   |
| `!!uint16`    | `!!dec`   | `16 b` Unsigned Integer                                  |
| `!!uint32`    | `!!dec`   | `32 b` Unsigned Integer                                  |
| `!!uint64`    | `!!dec`   | `64 b` Unsigned Integer                                  |
| `!!uint128`   | `!!dec`   | `128 b` Unsigned Integer                                 |
| `!!float32`   | `!!float` | `32 b` Single precision floating point number            |
| `!!float64`   | `!!float` | `64 b` Double precision floating point number            |
| `!!float128`  | `!!float` | `128 b` Quadruple precision floating point number        |
| `!!decimal16` | `!!float` | `16 b` Single precision floating point number in base 10 |
| `!!decimal32` | `!!float` | `32 b` Single precision floating point number in base 10 |
| `!!decimal64` | `!!float` | `64 b` Single precision floating point number in base 10 |

### Rich Datatypes

Rich datatypes are always based on the `!!str` but they can be parsed using a regex pattern. Each rich type can be coerced into different format.

All these formats have a strict binary encoding format when the data is transmitted in binary.

| Type        | Based on | Description                                                  | Coercible in                                      |
| ----------- | -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `!!version`  | `!!str`   | [SemVer](https://semver.org/) 2.0.0 version number           | `!!seq`, `!!map` |
| `!!regex`    | `!!str`   | PCRE2 Regular expression                                     |                                     |
| `!!time`     | `!!str`   | Time value e.g. `12:43:01` (`ISO 8601`)                      | `!!seq`, `!!map`       |
| `!!date`     | `!!str`   | Date value e.g. `1982-02-16` ( `ISO 8601`)                   | `!!seq`, `!!map` |
| `!!datetime` | `!!str`   | Full date time e.g. ` 2018-06-08T23:02:51+00:00 ` ( `ISO 8601`) | `!!seq`, `!!map`, `!!epoch` |
| `!!ipv4` | `!!str`   | IP version 4 described in [RFC 791](https://tools.ietf.org/html/rfc791) | `!!seq`, `!!uint32` |
| `!!ipv6` | `!!str`   | IP version 4 described in [RFC 8200](https://tools.ietf.org/html/rfc8200) | `!!seq`, `!!uint128` |
| `!!jwt` | `!!str` | JSON Web Token | `!!seq`, `!!map` |
| `!!uuid` | `!!str` | Unique User Identifier, a 128-bit data |  |
| `!!epoch` | `!!dec` | UNIX timestamp | `!!datetime` |
| `!!math` | `!!str` | Mathematic notation `$\frac{2}{3}$` |  |

### Constraint Datatypes

Constraint datatypes are only used for validation schema

| Type      | Based on | Description                                       |
| --------- | -------- | ------------------------------------------------- |
| `!!oneof`    | `!!map`   | Alternative possibility e.g. `!!oneof( !!str, !!null)` |
| `!!schema` | `!!type`  | *UON* Validation schema                           |

## Properties

Properties can be seen as attributes passed to a constructor of a class. When writing:

```yaml
!!uon(version: 0.0.1) {}
```

You essentially instantiate a `!!uon` class with the attribute `version` set to `0.0.1`. This instance is inherited automatically by the following argument here `{}` which is a `!!map`.  In *UON* only types can be instantiated, but since everything is static on a *UON* document, `!!uon` and `!!uon()` performs the same. It is not allowed to add properties to objects such as:

```yaml
{
  # Incorrect
  {foo: 1, bar: 2}(ordered:true)
  
  # Correct
  !!map(ordered:true) {foo: 1, bar: 2}
}
```

The reason for this is principally for the sake of readability. 

The `UON` parser automatically update the missing properties with correct values for example, this valid *UON* file

```yaml
42
```

```python
>>> import uon
>>> u = uon.Parser()
>>> doc = u.loads('42')
>>> doc
42
>>> type(doc)
uon.integer
>>> doc.binformat
uon.binformats.leb128
>>> doc.to_binary()
b'*'
```

## Type details

###Type

Type is the base type of all other types

| Property Name         | Description                                     | Example                              |
| --------------------- | ----------------------------------------------- | ------------------------------------ |
| `comment: !!str`       | Add a comment to an object                      | `!!dec(comment: "The Answer") 42`     |
| `description: !!str`   | Description for documentation purposes          | `!!dec(description: "The Answer") 42` |
| `binformat: !!keyword` | Encoding format used for binary payload         | `!!dec(binformat: int32) 42`          |
| `optional: !!bool`     | Is the value optional?                          |                                      |
| `content: !!type`      | This is the content value of object (read only) |                                      |

The `content`property is not meant to be used directly, but it appears in the `UON` DOM.

### Null 

TBD: I am not sure whether or not I should keep this type... Perhaps not very useful...

### Bool

Boolean value defined by this schema

```yaml
!!uon(version: 0.0.1) {
  !!bool: !!schema(urn:uon:2018:types:bool) 
    !!oneof(
      set: {
        !!keyword(coercion: {!!number: 0}) false, 
        !!keyword(coercion: {!!number: 1}) true
      },
      properties: {
        alias: !!set {
          !!keyword: !!oneof(set: {false, true})
        }  
      }
    )
}
```

| Property Name | Description                           | Example                       |
| ------------- | ------------------------------------- | ----------------------------- |
| `alias: !!set` | Adds accepted alternative for Boolean | `!!bool(alias: {on: true}) on` |

### String

A string is an arbitrary sequence of characters encoded in [UTF-8](https://www.ietf.org/rfc/rfc3629.txt). Since *UON* is a text based language, any other type can be coerced into a string. 

| Format    | Example                    |
| --------- | -------------------------- |
| Litteral  | `"A literal string"`       |
| Raw       | `r"Here \n is not parsed"` |
| Multiline | `"""A multiline string"""` |
|           |                            |



```yaml
!!uon(version: 0.0.1) {
  !!str: !!schema(urn:uon:2018:types:string) 
    !!type(
      properties: {
      pattern: !!regex,
        min: !!uint,
        max: !!uint(ge: @.min)
      }
    )
}
```

| Property Name     | Description                                          | Example                         |
| ----------------- | ---------------------------------------------------- | ------------------------------- |
| `pattern: !!regex` | Acceptance pattern                                   | `!!str(pattern: /0x[0-9a-f]+/i)` |
| `min: !!uint`      | Minimum length                                       | `!!str(min: 2)`                  |
| `max: !!uint`      | Maximum length (must be greater or equal than `min`) | `!!str(min: 2, max: 18)`         |

### Number

A number can hold any kind of representable number with an absolute precision without restriction of the storage type. It is therefore the base type for any derived number type. 

Number group:
$$
\N\subset\Z\subset\Q\subset\R\subset\C
$$


The following values are valid: 

```yaml
some-numbers: [
  149999932410893322.34334342387324024000485918571097402943e122,
  -.inf,
  -8.0 
]
```

Numbers accepts the [E-notation](https://en.wikipedia.org/wiki/Scientific_notation) with the letter `e` as  "times ten raised to the power of"

| Property Name                | Description                                          | Example                          |
| ---------------------------- | ---------------------------------------------------- | -------------------------------- |
| `magnitude: !!prefix`         | Order of magnitude                                   | `!!number(magnitude: k) 12`       |
| `base: !!uint(min:2, max:62)` | Positional notation (base-2, base-10...)             | `!!number(base: 4) 222 # 42`      |
| `min: !!number`               | Minimum accepted number                              | `!!number(min: 2)`                |
| `max: !!number(min: @.min)`   | Maximum length (must be greater or equal than `min`) | `!!number(min: 2, max: 18)`       |
| `le: @.max`                  | Less than or equal to                                | `!!number(le: 2)`                 |
| `ge: @.min`                  | Greater than or equal to                             | `!!number(ge: 2)`                 |
| `lt: !!number`                | Less than                                            | `!!number(lt: 2)`                 |
| `gt: !!number(min: @.lt)`     | Greater than                                         | `!!number(gt: 2)`                 |
| `uncertainty: !!number`       | Uncertainty value such as in `12 ± 0.01`             | `!!number(uncertainty: 0.1) 12`   |
| `unit: !!unit`                | Associated unit                                      | `number(unit: celsius) 21.3`     |
| `fix(default: 0): !!uint`     | Fixpoint representation, value is divided by `!!uint` | `number(base: 2, fix: 2) 110010` |

Positional notation uses the representable symbols in this order `"0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ"`, which makes the maximum possible representable base 62

### Keyword

A keyword is essentially a string which can be represented without double quotes. It can be either `undefined` or `defined`. An undefined keyword is not defined in the search path, while the defined keyword is documented and defined somewhere in the search path.

```yaml
!!uon(version: 0.0.1) {
  !!keyword: !!schema(urn:uon:2018:types:keyword) !!str(
     pattern: /^[a-z][a-z0-9-]+(?!-)$/i,
     properties: {
       enum: !!dec,
       coercion: {
         !!number: @..enum
       }
     }
  )
}
```

| Property Name | Description       | Example                    |
| ------------- | ----------------- | -------------------------- |
| `enum: !!dec`  | Enumeration value | `!!keyword(enum: 1e3) kilo` |

### References

A reference refers to another field on a *UON* document. A reference can be resolved or not when generating a document. It can be a local reference or a remote reference when the target has to be fetched outside from the document. References can also acts as hyperlinks, this is especially the case when representing a UON document in a Web browser. 

| Explicit                        | Syntactic sugar |
| ------------------------------- | --------------- |
| `!!ref ..foo.bar`                | `@(..foo.bar)`  |
| `!!ref(resolve: true) ..foo.bar` | `@@(.foo.bar)`  |

```yaml
{
  foo: { bar: 2, baz: [10, 20, 30], "qux": 40 },
  sub: {
    local: @@(..foo.bar) # Is replaced by `2` when generating the payload
  },
  local: @@(.foo) # Is replaced by `{ bar: 2}` when generating the payload
  other: [
    @@(..foo.baz[1]), # Replaced by `20` when generating the payload
    @@(..foo."qux"), # Replaced by `40` when generating the payload
  ],
  remote: @@(http://example.com/something.uon) # Replaced by whatever in something.uon
}
```


| Property Name                    | Description                             | Example                        |
| -------------------------------- | --------------------------------------- | ------------------------------ |
| `resolve(default: false): !!bool` | Resolve reference at payload generation | `!!ref(resolve=true) ..foo.bar` |

The schema of a references is expressed as follow:

```yaml
!!uon(version: 0.0.1) {
  !!keyword: !!schema(urn:uon:2018:types:reference) !!oneof {
    !!uri, 
    !!str
  },
  properties: {
    resolve(default: false): !!bool  
  }
}
```

URN references relies on an external UON document that resolve a urn. The resolve document is usually passed as follow `!!uon(urn-resolve: @(http://some-domain.com))`. A set can be passed to `urn-resolve`

A possible value for a this resolver is: 

```yaml
# Resolver for URN
!!uon(version: 0.0.1) {
   ietf: {
     rfc: 
       2648: !!url "https://www.ietf.org/rfc/rfc2648.txt"
   }
}
```

### Binary, Octal and Hexadecimal representation

```yaml

!!uon(version: 0.0.1) {
  !!bin: !!schema(urn:uon:2018:types:binary) !!str(
     pattern: /^0b([01]+)$/,
     coerce: {
       !!number: !!number(base: 2) "$1"
     }
     properties: {
       size: !!uint,
     }
  )
}
```

```yaml
# Value of this is `2.16015625`
!!number !!bin(fix: 8) 0b1000101001
```

### IPv4

The IPv4 type is defined in UON

```yaml
!!ipv4: !!schema("http://uon.io/ipv4") !!str(
  description: "Type representing a Internet Protocol version 4 address defined in RFC 760",
  links: [
    !!url "https://tools.ietf.org/html/rfc760"
  ]
  pattern: /^((?&b))\.((?&b))\.((?&b))\.((?&b))(?(DEFINE)\b(?<b>25[0-5]|2[0-4]\d|[01]?\d\d?))$/,
  coerce: {
    !!seq: [!!int "$1", !!int "$2", !!int "$3", !!int "$4"]
  },
  properties(merge=true): {
    mask(optional: true): !!bool(default: false)
  }
)
```

## Physics

Physical constants



## Units

An important information for telemetry systems and Internet of Things is the unit of magnitudes. When transmitting a temperature, it is essential to know which is the unit used to represent that number. *UON* uses the ISQ system ([ISO/IEC 80000-1:2009](https://www.iso.org/standard/30669.html)). Currently none of the serialization formats listed [here](https://en.wikipedia.org/wiki/Comparison_of_data_serialization_formats) provide native support for units.

### Base Units

The International System of Units lists 7 fundamental units that are used as standard. *UON* does not support imperial units deliberately because amount the 195 countries in the world only three countries do not use the metric system: Liberia, Myanmar and the USA. However *UON* 

| Unit symbol | Unit name | Quantity name                             |
| ----------- | --------- | ----------------------------------------- |
| `m`         | Meter     | Length (`length`)                         |
| `kg`        | Kilogram  | Mass (`mass`)                             |
| `s`         | Second    | Time (`time`)                             |
| `A`         | Ampere    | Electric current (`current`)              |
| `K`         | Kelvin    | Thermodynamic temperature (`temperature`) |
| `mol`       | Mole      | Amount of substance (`substance`)         |
| `cd`        | Candela   | Luminous intensity (`illuminance`)        |

When only the quantity is known, the unit will be assumed to be SI strict.

```yaml
# Temperature is assumed to be given in Kelvin 
!!number(quantity: temperature) 250.24
```

### Derived Units

The International System of Units assigns special names to 22 derived units, which includes two dimensionless derived units, the `radian` and the `steradian`.

| Unit Symbol | Unit Name      | Quantity name                      | Equivalents                                                  |
| ----------- | -------------- | ---------------------------------- | ------------------------------------------------------------ |
| `Hz`        | Hertz          | `frequency`                        | $1/\text{s}$                                                 |
| `rad`       | Radian         | `angle`                            | 1                                                            |
| `sr`        | Steradian      | `solid-angle`                      | 1                                                            |
| `N`         | Newton         | `[force, weight]`                  | $\text{kg}\cdot\text{m}/\text{s}^2$                          |
| `Pa`        | Pascal         | `[pressure, stress]`               | $\text{kg}\cdot\text{m}^{-1}\cdot\text{s}^{-2}$              |
| `J`         | Joule          | `[energy, work, heat]`             | $\text{kg}\cdot\text{m}^2\cdot\text{s}^{-3}$                 |
| `W`         | Watt           | `[power, radian-flux]`             | $\text{kg}\cdot\text{m}^2\cdot\text{s}^{-2}$                 |
| `C`         | Coulomb        | `electric-charge`                  | $\text{s}\cdot\text{A}$                                      |
| `V`         | Volt           | `voltage`                          | $\text{kg}\cdot\text{m}^{2}\cdot\text{s}^{-3}\cdot\text{A}^{-1}$ |
| `F`         | Farad          | `electric-capacitance`             | $\text{kg}^{-1}\cdot\text{m}^{-2}\cdot\text{s}^{4}\cdot\text{A}^{2}$ |
| `Ω `        | Ohm            | `[electric-resistance, impedance]` | $\text{kg}\cdot\text{m}^2\cdot\text{s}^{-3}\cdot\text{A}^{-2}$ |
| `S`         | Siemens        | `electric-conductance`             | $\text{kg}^{-1}\cdot\text{m}^{-2}\cdot\text{s}^{3}\cdot\text{A}^{2}$ |
| `Wb`        | Weber          | `magnetic-flux`                    | $\text{kg}\cdot\text{m}^2\cdot\text{s}^{-2}\cdot\text{A}^{-1}$ |
| `T`         | Tesla          | `magnetic-field`                   | $\text{kg}\cdot\text{s}^{-2}\cdot\text{A}^{-1}$              |
| `H`         | Henry          | `electric-inductance`              | $\text{kg}\cdot\text{m}^2\cdot\text{s}^{-2}\cdot\text{A}^{-2}$ |
| `°C`        | Degree Celsius | `temperature`                      | $\text{K}$                                                   |
| `lm`        | Lumen          | `luminous-flux`                    | $\text{cd}$                                                  |
| `lx`        | Lux            | `illuminance`                      | $\text{m}^{-2}\cdot\text{cd}$                                |
| `Bq`        | Becquerel      | `radioactivity`                    | $\text{s}^{-1}$                                              |
| `Gy`        | Gray           | `absorbed-dose`                    | $\text{m}^2\cdot\text{s}^{-2}$                               |
| `Sv`        | Sievert        | `equivalent-dose`                  | $\text{m}^2\cdot\text{s}^{-2}$                               |
| `kat`       | Katal          | `catalytic-activity`               | $\text{s}^{-1}\cdot\text{mol}$                               |

These derived units can be used either on a payload or on a validation schema:

```yaml
[
  !!number(unit: "°C") 21.13,   # Explicit unit
  21.13 °C                     # Also valid
]
```

### Other accepted units

The SI system lack some useful units as listed below. Millimeter of mercury is part of UON because of its large use in medicine. The electron-volt is widely use in fundamental physics. 

| Unit Symbol | Unit Name             | Quantity name | Equivalents                           |
| ----------- | --------------------- | ------------- | ------------------------------------- |
| `l`         | Liter                 | `volume`      | $ \text{dm}^3 $                       |
| `h`         | Hour                  | `time`        | $3600\cdot\text{s}$                   |
| `min`       | Minute                | `time`        | $60\cdot\text{s}$                     |
| `day`       | Day                   | `time`        | $24\cdot\text{hour}$                  |
| `month`     | Month                 | `time`        | $12\cdot\text{month}$                 |
| `year`      | Year                  | `time`        | $365.25\cdot\text{day}$               |
| `bar`       | Bar                   | `pressure`    | $100\cdot10^3\cdot\text{Pa}$          |
| `mmHg`      | Millimeter of mercury | `pressure`    | $(101325/760)\cdot\text{Pa}$          |
| `eV`        | Electronvolt          | `energy`      | $1.602176620898\cdot10^{-19}\text{J}$ |

### Units of information

Other non-SI units shall also be accepted such as unit of information. 

| Unit symbol | Unit name     | Quantity name |
| ----------- | ------------- | ------------- |
| `b`         | Bit           | Base 2        |
| `B`         | Octet or Byte | Exactly 8 b   |
| `nat`       | Nat           | Base `e`      |
| `dit`       | Dit           | Base 10       |

### Prefix

Prefix can be added in front of a unit 

| Symbol | Name  | Factor                            | Power      |
| ------ | ----- | --------------------------------- | ---------- |
| `Y`    | Yotta | 1 000 000 000 000 000 000 000 000 | $10^{24}$  |
| `Z`    | Zetta | 1 000 000 000 000 000 000 000     | $10^{21}$  |
| `E`    | Exa   | 1 000 000 000 000 000 000         | $10^{18}$  |
| `P`    | Peta  | 1 000 000 000 000 000             | $10^{15}$  |
| `T`    | Tera  | 1 000 000 000 000                 | $10^{12}$  |
| `G`    | Giga  | 1 000 000 000                     | $10^9$     |
| `M`    | Mega  | 1 000 000                         | $10^6$     |
| `k`    | Kilo  | 1 000                             | $10^3$     |
| `h`    | Hecto | 100                               | $10^2$     |
| `da`   | Deca  | 10                                | $10^1$     |
| -      | -     | 1                                 | $10^0$     |
| `d`    | Deci  | 0.1                               | $10^{-1}$  |
| `c`    | Centi | 0.01                              | $10^{-2}$  |
| `m`    | Milli | 0.001                             | $10^{-3}$  |
| ` μ `  | Micro | 0.000 001                         | $10^{-6}$  |
| ` n`   | Nano  | 0.000 000 001                     | $10^{-9}$  |
| `p`    | Pico  | 0.000 000 000 001                 | $10^{-12}$ |
| `f`    | Femto | 0.000 000 000 000 001             | $10^{-15}$ |
| `a`    | Atto  | 0.000 000 000 000 000 001         | $10^{-18}$ |
| `z`    | Zepto | 0.000 000 000 000 000 000 001     | $10^{-21}$ |
| `y`    | Yocto | 0.000 000 000 000 000 000 000 001 | $10^{-24}$ |

In addition to these prefixes *UON* supports the [JEDEC](https://en.wikipedia.org/wiki/JEDEC_memory_standards) memory standards:

| Symbol | Name  | Factor                    | Power    |
| ------ | ----- | ------------------------- | -------- |
| `Ei`   | Exibi | 1 152 921 504 606 846 976 | $2^{60}$ |
| `Pi`   | Pebi  | 1 125 899 906 842 624     | $2^{50}$ |
| `Ti`   | Tebi  | 1 099 511 627 776         | $2^{40}$ |
| `Gi`   | Gibi  | 1 073 741 824             | $2^{30}$ |
| `Mi`   | Mebi  | 1 048 576                 | $2^{20}$ |
| `Ki`   | Kibi  | 1024                      | $2^{10}$ |

### Derived quantities and units

Other derived quantities and units can be built from these basic types.

```yaml
"12 J/(K⋅mol)": {
  magnitude: 12,
  unit: {
    J: 1,
    K: -1,
    mol: -1,
  }
}
```

###Representation

Quantities are represented and written using UTF-8 special chars. It is perfectly allowed to write: 

```yaml
value: 12.35±0.01 J/(m²⋅s)
```

If coerced in `!!map`, it would become:

```yaml
value: !!magnitude {
  magnitude: 12.35,
  uncertainty: 0.01,
  unit: {
    J: 1,
    m: -2,
    s: -1
  }
}
```

Units can be parsed in any order, but they are always serialized in the same manner 

## Validation

### Properties

| Property       | Description                                                  | Example                                             |
| -------------- | ------------------------------------------------------------ | --------------------------------------------------- |
| `optional`     | Type or key is optional, default `false`                     | `foo(optional: false): !!str(min: 10)`               |
| `exclusive`    | Cannot be present along with                                 | `foo-or-bar: { !!exclusive( foo: !!str, bar: !!dec ) } |
| `min`          | Minimum length or value (equivalent to `le`)                 | `age: !!int(min: 0, max: 200)`                       |
| `max`          | Maximum length or value (equivalent to `ge`)                 |                                                     |
| `repr`         | Representation format (coercive)                             | `!!seq(repr: !!complex)`, `[1, 20]` becomes `1+20j`   |
| `quantity`     | Quantity value for physical magnitudes                       | `!!dec(quantity: "temperature")`                     |
| `dummy`        | The value is dummy and does not appear in the payload. Usually used as a reference on other fields |                                                     |
| `default`      | Default value when missing                                   | `enabled: !!bool(default: false)`                    |
| `merge`        | Merge the value with the inherited type, false by default    |                                                     |
| `offset`       | Offset added to the serialized value (used in binary transmissions) |                                                     |
| `scale-factor` | Multiplication factor (used in binary transmissions)         |                                                     |
| `gt`           | The value must be greater than this value                    |                                                     |
| `lt`           | The value must be greater than this value                    |                                                     |
| `ge`           | Greater or equal                                             |                                                     |
| `le`           | Lesser or equal                                              |                                                     |
| `description`  | Description of the key or type                               |                                                     |
| `link`         | List of hyperlinks                                           |                                                     |
| `alias`        | Used to accept values as an alias of another one             | `!!bool(alias: {on: true, off: false})`              |
| `resolve`      | Resolve reference when serializing, default `false`          |                                                     |
| `hash`         | Hash of the associated value computed on the binary format. This is useful to make sure a document as not been modified. The hash value can be an external reference. It is computed in SHA256 |                                                     |
| `signature`    | Any value can be signed using a digital signature            |                                                     |

## Encryption 

A UON document can be encrypted. If so, the content of the document is simply a string value usually in base64. UON MUST support AES CBC (128, 192, 256 bit)

```yaml
!!uon {
  login: "John",
  password: "sequoiaTree"
}
```

Here is what should look the encrypted payload with the passcode `UON`:

```yaml
!!uon(encryption: aes128) !!base64 "XWTrUQIavBWU58SXjpdmZAe/NX4C7Xc1nMQenvK0oKzzyWnwfe8Y5UkmMibT1mwG"
```

The encryption use the binary format as input to AES algorithm

## Capabilities

As UON is made to increase interoperability, the UON standard may not be fully supported, also extensions to the language could be added by third-party people. To ensure two communication nodes can communicate together, they can share minimal UON document with their capabilities such as: 

```yaml
!!capabilities {
  encryption: {
    aes
  }
  formats: {
    minimal: true,
  	binary: true,
  }
}
```

## Destructuring 

With the help of references (`!!ref`), *UON* data can be easily destructured to only access a subset of data. This might be useful when used within a RESTful API. 

```yaml
!!uon(version: 0.0.1) {
    full: {
      name: "Midhuna",
      age: 23,
      place: "New York",
      hobbies: [
        "Singing", 
        "Reading Books"
      ]
      spouse: {
        name: "Akash",
        age: 25
      }
    # This subset use destructuring to compose a different arborescence of data 
    subset: {
      name: @full.name,
      spouse: @full.spouse.name
    }
}
```

Therefore accessing to `subset` would return: 

```yaml
{
  name: "Midhuna",
  spouse: "Akash"
}
```

Notice that references are resolved because they are no longer accessible from the subset. 

### Linked list

The example below represents a Git repository with linked references. When serialized, one can choose to flatten or not the references. 

Editors that supports the UON syntax highlight SHOULD be able to resolve references. 

```yaml
!git-repository {
  head: @(refs.master)
  refs: {
    master: @(b43f2c2f661030e2cd4129787e71052567d4ef5a)
  }
  authors: {
    "jdoe@example.com": "John Doe"
  }
  objects: {
    b43f2c2f661030e2cd4129787e71052567d4ef5a: !commit {
      tree: @(ee313d61f4eb880e14003a28690ea63980673d9c), 
      parent: @(c9825517d028c352af89a3229f011bdf085f7443),
      author: {
        identity: @(authors."jdoe@example.com"),
        date: !!epoch 1528555907
      }        
    },
    c9825517d028c352af89a3229f011bdf085f7443: !commit {
      tree: @(ea41dba10b54a794284e0be009a11f0ff3716a28), 
      parent: @(2770d3625580ecd3bd4cf57cb5bff9751f8cbb3c),
      author: {
        identity: @(authors."jdoe@example.com"),
        date: !!epoch 1528555894
      }        
    },    
    2770d3625580ecd3bd4cf57cb5bff9751f8cbb3c: !commit {
      tree: @(4d5fcadc293a348e88f777dc0920f11e7d71441c), 
      parent: @(2770d3625580ecd3bd4cf57cb5bff9751f8cbb3c),
      author: {
        identity: @(authors."jdoe@example.com"),
        date: !!epoch 1528555881
      }
    },        
    ee313d61f4eb880e14003a28690ea63980673d9c: !tree {
      "bar": {
        permissions: 100644,
        ref: @(e69de29bb2d1d6434b8b29ae775ad8c2e48c5391)
      },
      "foo": {
        permissions: 100644,
        ref: @(d9e80f6f7602c18d035af9659303047b30204182)
      }     
    },
    e69de29bb2d1d6434b8b29ae775ad8c2e48c5391: !blob "",
  }
}
```

Here is the schema of this data structure, allowing validation: 

```yaml
!!uon(version: 0.0.1) {
  !git-repository: !!schema {
    head: !!ref(base: "refs"),
    refs: {
      !!keyword: !!str
    },
    authors: {
      !!str(desc: "email"): !!str
    },
    objects: {
      !!keyword: !!oneof(
        !commit,
        !blob,
        !tree
      ) 
    }
  },
  !ref: !!schema !!ref(base: "objects"),
  !filename: !!schema !!str,
  !blob: !!schema !!oneof([!!blob, !!str]),
  !commit: !!schema {
    tree: !ref,
    parent(optional: true): !ref,
    author: {
      identity: !!ref(base: "authors"),
      date: !!datetime
    }
  },
  !tree: !!schema {
    !filename: {
      permission: !ref,
      ref: !ref
    }
  }   
}
```

## RESTful access

When a UON file is exposed on a RESTful API, a *UON* file MUST be parsed automatically. 

###Example

The following *UON* file is stored on a server and made accessible from `http://example.com/midhuna`

```yaml 
!!uon(version: 0.0.1) {
  name: "Midhuna",
  age: 23,
  place: "New York",
  hobbies: [
    "Singing", 
    "Reading Books"
  ]
  spouse: {
    name: "Akash",
    age: 25
  }
}
```

If accessing `http://example.com/midhuna/age` the response would be:

```
23
```

If accessing `http://example.com/midhuna/spouse` the response would be:

```
{
  name: "Akash",
  age: 25
}
```

## Raw Conversion

*UON* source can be serialized in other format such as YAML, XML or JSON with respect of their capabilities and limitations. 

```yaml
!!uon(version: 0.0.1) {
  # This is a number
  int: !!uint32 12,
  str: "Hello World!"
  ref: @(.int)
}
```

### JSON

Canonical encoding in JSON format is possible to make it easier to share data between systems. 

They are two way of generating a JSON:

* Raw
* Closest

```json
{
  "type": "!!uon",
  "properties": {
    "version": {
      "type": "!!semver",
      "content": "0.0.1"
    }
  },
  "content": {
    "type": "!!map",
    "content":
      {
        "int": {
          "type": "!!uint32",
          "content": 12,
          "properties": {
            "comment": "This is a number"
          }
        }
      },
      "str": { 
        "type": "!!str", 
        "content": "Hello World!"
      }
    }
  }
}
```

### XML

```xml
<?xml encoding='UTF-8'?>
<uon:uon version="0.0.1" xmlns:uon="http://uon-language.io/uon-type">
  <uon:map>
    <int type="!!uint32" comment="This is a number">12</int>
    <str type="!!str">Hello World!</str>
    <ref type="!!ref">.int</ref>
  </uon:map>
</uon:uon>
```

### YAML

```yaml
%YAML 1.2
---
type: uon
properties:
  version: 0.0.1
content:
  int: 
  	type: "!!uint32"
  	comment: This is a number
  	content: 12
  str:
    type: "!!str"    
    content: Hello World!
  ref: 
    type: "!!ref"
    content: .int    
...
```

## File format and extensions

The only allowed extension for *UON* file is `.uon` 

## Unified API

The unified API should be identical on all supported programming languages

- `to_json()`
- `to_xml()`
- `to_yaml()`
- `to_uon(strict=true, format)`
- `to_binary(strict=true)`
- `load(file)`
- `loads(string)`
- `dump(file)`
- `dumps()`
- `validate(schema)`
- `translate(stylesheet)`
- `register_type(type, function)`

## Implementations

This chapter will be removed from this specification when released...

In the context of this draft, no implementation are fully available, however projects in development are available below: 

| Language   | Link                                  | Status |
| ---------- | ------------------------------------- | ------ |
| C          | -                                     | TBD    |
| C++        | -                                     | TBD    |
| C#         | -                                     | TBD    |
| JavaScript | -                                     | TBD    |
| Perl       | -                                     | TBD    |
| Python     | http://github.com/uon-language/py-uon | TBD    |
| PHP        | -                                     | TBD    |

## Syntax Highlight and support in text editors

I believe *UON* will only be used by others when its syntax is fully supported by the major text editors. This chapter will be removed from this specification when released...

| Editor            | Link | Status |
| ----------------- | ---- | ------ |
| Microsoft VS Code | -    | TBD    |
| Vim               | -    | TBD    |
| Atom              | -    | TBD    |

# Notes and deep thoughts

Of course, this section will be removed from the specification once all the ideas below are fully crystalized without inconsistencies. 

I am trying to find the most simple and elegant writing to address typical use cases of what a serialization language should serve.

Again, *UON* is not a programming language, I don't want to think about loops, functions, or conditionals. Everything should be easily parsed without much computation power. If someone wants to extend the capabilities, it can do at the application level by binding a `resolver` function to a existing type.

Data transformation has to be kept minimal. *UON* is not a language made to transform information, but it has to be able to represent data according to the destination domain: binary payload, human readable document, ... Arithmetic transformations shall be possible to represent data written from one unit to another. A payload that gives an energy value in Joules could be presented in kWh if required during the rendering. 

One important application of this feature is IoT. Let's imagine a small temperature sensor. It is composed of a very low-power MCU that transmit temperature data using BLE (Bluetooth Low Energy). To be interoperable, it uses *UON* to describe and encode the payload. 

```yaml
# http://iot-federation.io/uon/schemas/temperature-sensor
!temperature-sensor: !schema(
  description: "Standardized temperature sensor descriptor"
) {
  temperature(required: true): !!number(quantity: temperature),
  sampling-time(optional: true): !!number(quantity: frequency)
}
```

The above schema is device-agnostic. It does not define any unit nor any sized types allowing small devices to transmit data using `!!uint8` or `!!float32`. So the data-consumer that receives the binary data shall be able to interpret the data by only knowing this reference schema. However, the payload should be lightweight and it should not include details such as encoding type. 

In the case of our, temperature sensor the binary representation looks like this, eight bit long.

```
[ !!uint8 ]
```

The data-consumer that listen to this sensor (imagine a MQTT payload), will need more information. So, the device establish a refined schema:

```yaml
# Inherit of !temperature-sensor
!schema(!temperature-sensor, refine: true, id: 02d28a9a-6e7b-11e8-adc0-fa7ae01bbebc) { 
  temperature(inherit: true): !!uint8(unit: celsius),
}
```

That's it! The new refined schema is compatible with `!temperature-sensor` the temperature value is still a `!!number`, it has the same quantity (temperature). The optional `sampling-time` is still optional so if it is transmitted, the device will need to encode it using a full type notation i.e. the type identifier (0xab for a `!!float64`) followed by the 64-bit data. 

Now this refined schema has to be exchanged from the BLE temperature sensor to the data-consumer as a contract that describe the payload. How is it done? Well, I am not yet sure of the best solution but here some possibility: 

* The refined schema is transmitted once by the device either in plain or binary data. Notice that binary format get rid of properties, unless it belongs to  `!!schema` so in Python doing `my_refined_schema.to_binary()` gives full binary payload that can be parsed and understood. The good thing is only the altered properties and types are mentioned. 
* The refined schema is stored on a global server and a URI gives the access to it. The data-receiver should then ask the device for the URI of the refined schema. This is possible in MQTT by simply subscribing to `/my-device/.schema` 

```yaml
!schema(http://my-sensor.com/schemas/s231abx)
```

The generated binary payload for this will be still heavy because it contains the string:  `http://my-sensor.com/schemas/s231abx`. With additional data such as `0x80` (schema), and the properties identifiers. 

* The refined schema is stored somewhere on a global server, but the device transmit only the `uuid` of the schema. The payload is then 128-bit + the schema type `0x80`.  Its better, but the data-receiver should be able to match the `uuid` somewhere. 

Let's consider for one moment the data receiver. A nice MQTT client which subscribe to this data:

```python
from uon import Parser
import paho.mqtt.client as mqtt

# Create an optimized Uon parser
parser = Parser(schema='!schema(http://my-sensor.com/schemas/s231abx)')

client = mqtt.Client()
client.on_message = on_message
client.connect("iot.eclipse.org")
client.subscribe("a-sensor")

def on_message(client, userdata, msg):
   print("Received message from %s" % msg.topic)
   print(parser.from_binary(msg.payload).to_uon())
```

The received message will decoded into:

```
{
  temperature = 23 °C
}
```

Why is it displayed in degree Celsius? Because the refined schema specifies it so. What about if we would like to get an agonistic temperature, in SI units. Well, the original schema does not specify the unit. By default units are SI compliant, so given in Kelvin. The fully agnostic answer will be:

```
{
  temperature: 296.15 K
}
```

But, if I want manually process the temperature at the application level I can do this:

```python
from uon.types import Float

def on_message(client, userdata, msg):
   payload = parser.from_binary(msg.payload)
   temp = payload.temperature
   # type(temp) is <!!uint8>, so temp.unit is `celsius`
   # I can change that easily
   print("Temperature: %f °F" % 
       float(Float(temp, unit='fahrenheit')))

# Temperature: 73.4 °F
```

Isn't beautiful?

This is great, but *UON* is powerful enough to do it on its own. Let's suppose the data-receiver will just act as an intermediary. It receives the binary payload and transmit it in plain *UON* to another device with explicit information. In other words, it has to generate this:

```
!!uon(version: 0.0.1) !@(http://iot-federation.io/uon/schemas/temperature-sensor) {
  temperature: 296.15
}
```

```python
temp.to_uon(version: '0.0.1', describe: true)
```

Let's now imagine we want to transmit the payload to another very lightweight device that understand the temperature sensor, but in Kelvin with a negative offset, and with data given in `!!float32`. I simply need to refine the original schema and coerce the received payload into it: 

```python
refined = """!schema(!temperature-sensor) { 
  temperature(inherit: true): !!float32(unit: kelvin offset: -10.2)}"""

def on_message(client, userdata, msg):
   client.publish('topic-to/other-device', 
       parser.from_binary(msg.payload).coerce(refined).to_binary())
```

Why would it work that smoothly? Because the payload is still a derived version of `temperature-sensor` so the properties of `temperature` can be modified without changing the original type `!!number`.  

## Aggregation

One missing feature especially on MQTT is data aggregation. For example, a `!!3d-accelerometer` streams the following at 100 Hz

```yaml
{
  t: 5433335321 ns
  x: 12.43 m/s²
  y: 32.5 m/s²
  z: 0.05 m/s²
}
```

Despite the binary data is really small, when streamed at that speed over AMQP or MQTT, it would quickly flood the network. If we define a new type `!3d-accelerometer-3` which contain 3 values, we will loose the interoperability because it is not a `!!3d-accelerometer`. Let's imagine this schema: 

```yaml
!3d-accelerometer-3: !schema [ !3d-accelerometer, !3d-accelerometer, !3d-accelerometer]
```

It is ugly for numerous reasons. What would be prettier would be instead:

```yaml
!schema: !repeat(3, !3d-accelerometer)
```

But how can it be understood or either refined? I am not sure I have the answer yet. The scenario is the following. Somebody is expecting from a topic 3d acceleration values. It is expecting a `!3d-accelerometer` payload. So the device should be able to refine the schema in a way it can aggregate several values in the same payload. But not all types are meant to be aggregated. A schema for instance does not change over time, a user documentation, a network description also does not change over time. 

In addition, when the payload is very light, such as our temperature-sensor, carrying the timestamp for each value is stupid. What we should have instead is the initial timestamp and the increment between the values. Some payload could also have values that does not change over a period of time, but they still need to be transmitted. Let's review this

* Find a way to gather data into the same dataset
* But doing so without the need to define a new schema. 
* Having a solution to group similar values that do not often changes
* Add a known offset to some values to retrieve the time increment. 
* `i * factor + offset` would cover 99% of the use-cases

```yaml
!acc-seq: !schema !seq(
  min: 1, 
  max: 255, 
) [ !3d-accelerometer ]

data: !acc-seq(index: .time, offset: 5433335321, increment: 250) [
    {x: 12.43 m/s², y: 32.5 m/s², z: 0.05 m/s²},
    {x: 12.43 m/s², y: 32.5 m/s², z: 0.05 m/s²},
    {x: 12.43 m/s², y: 32.5 m/s², z: 0.05 m/s²}    
]
```



## Sandbox

```yaml
{
  # Month name
  !month: !schema(
  ) !!oneof {
    january(capitalize: true, coerce: {!!int: 1}),
    february(capitalize: true, coerce: {!!int: 2}),
  }
  
  # How to translate month names in another language?
  # More generally, how to inherit a type, and set a property
  #
  #     "2" == !!str !!int 2
  #
  # Int is coerced into a string, it becomes a string, so it keeps all the 
  # properties from the closest parent, such as `comment` and `description`,
  # but it loose the coerce property (TODO: does it?)
  # 
  # Possible hint? By writing !!prop(content), we pick-up the content property 
  # which is a string. No, it is not the solution... Bad guess...
  # 
  # Woot, I need a property to inherit properties...
  # 
  #     !!str(inherit-prop: {foo}) !!int(foo: 42) 23
  #
  !month-fr: !schema !match({
    january: "Janvier",
    february: "Février",
  }) !month
  
  # http://uon-language.io/type/country
  !country: !!schema(
    id: !!uuid d925fe72-6e69-11e8-adc0-fa7ae01bbebc,
    desc: "Standardized country code with associated country name",
    example: "!country ch",
    rank: 1,
    version: 0.0.1
  ) !!oneof {
    af("Afghanistan"),
    ax("Åland Islands"),
    al("Albania")
  },
  
  # Countries according to ISO 3166-1
  # http://uon-language.io/type/country
  !country: !schema(
    id: !!uuid d925fe72-6e69-11e8-adc0-fa7ae01bbebc,
    desc: "Standardized country code with associated country name",
    example: "!country ch",
    rank: 1,
    version: 0.0.1
  ) !!oneof {
    af("Afghanistan"),
    ax("Åland Islands"),
    al("Albania")
  },

  # Country-name
  # Read name property and render it as a string
  !country-name: !schema(
    desc: "Cast a !country into its name (access name property)",
  ) !!str !!prop(name) !country,

  # International geographic point location (ISO 6709:2008)
  !geo: !!schema(
    desc: "Standardized Geographic point location"
    example: "!geo \"+401213.1N+4012.22E123.45\"",
    rank: 1,
    version: 0.0.1
  ) !!str(
    pattern: //,
    coerce: {
      !!map: {
        latitude: {
          deg: !!operation "$1+42/12+$2",
          min: "$2",
          sec: "$3",
        },
        longitude: {
          deg: "$1",
          min: "$2",
          sec: "$3"        
        },
        height: !!number(unit: meter) "$8"
      }
    }
  ), 

  !person: !schema(
    dependencies: {
      http://uon-language.io/type/country,
      http://uon-language.io/type/geo,
    }
  ) {
    name: !!str(min: 1),
    country: !country,
    location: !geo,
    other-location: !@(http://uon-language.io/type/geo),
    genre: !any { male, female },
    partner: !person # Recursive reference
  }
}
```



# Annexes

## Regulations and standards

*UON* must be as standard as possible against existing standards for expressing data such as IP address, timestamp or mathematical notation. 

| UON Type          | Standards                                                | Example                                                  |
| ----------------- | -------------------------------------------------------- | -------------------------------------------------------- |
| `!!uuid`          | [RFC 4122](https://tools.ietf.org/html/rfc4122)          | `uuid:f81d4fae-7dec-11d0-a765-00a0c91e6bf6`              |
| `!!jwt`           | [RFC 7797](https://tools.ietf.org/html/rfc7797)          | `xxxxx.yyyyy.zzzzz`                                      |
| `!!datetime`      | [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601)       | `2018-06-10T16:25:17+00:00`                              |
| `!!date`          | [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601)       | `2018-06-10`                                             |
| ``!!time``        | [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601)       | `16:25:17`                                               |
| `!!regex`         | [PCRE](https://www.pcre.org/)                            | `/foo|[bBar]/`                                           |
| `!!unit`          | [ISO 80000-1](https://en.wikipedia.org/wiki/ISO_80000-1) | `celsius`                                                |
| `!!math`          | [MathJax](https://www.mathjax.org/)                      | `$x_{1,2}=-b\pm\frac{\sqrt{b^2-4\cdota\cdotc}{2\cdota}$` |
| `!!number-system` | -                                                        | `real`                                                   |
| `!ipv4`           | [RFC 791](https://tools.ietf.org/html/rfc791)            | `192.168.1.42`                                           |
| `!!ipv6`          | [RFC 2732](https://www.ietf.org/rfc/rfc2732.txt)         | `1080::8:800:200C:417A`                                  |
| `!!urn`           | [RFC 8141](https://tools.ietf.org/html/rfc8141)          | `urn:example:foo-bar-baz-qux?+CCResolve:cc=ch`           |
| `!!uri`           | [RFC 2396](https://www.ietf.org/rfc/rfc2396.txt)         | `news:comp.infosystems.www.servers.unix`                 |
| `!req`            | [RFC  2119](https://www.ietf.org/rfc/rfc2119.txt)        | `!req(must) "Be portable and interoperable"`             |
| `!log`            | [RFC 5424](https://tools.ietf.org/html/rfc5424)          | `!log(error) "Unable to bind socket"`                    |
| `!!str`           | [RFC 3629](https://tools.ietf.org/html/rfc3629)          | `"𝚄O𝙽™ ı𝘴 𝛼 𝒇𝛼𝐧𝜏𝓪𝕤𝕥⍳ⲥ 𝓈𝙚𝓻𝚤𝕒lℹ𝙯𝝰𝜏ｉο𝓃 l𝙖ϖℊʋ𝙖𝗀𝕖"`          |



## Type names vs other languages

Consensus matters, *UON* aims to use the largest consensus, by considering several aspects:

* How many languages agree on type names?
* Which language has better credit for a chosen type (high level, low level language)?
* Tradition over time
* Simplicity, shorter is better, simple is better

| UON    | Math    | C/C++ | Java    | Python | Go     | Ruby   | C#     | PHP     |
| ------ | ------- | ------ | ------- | ------ | ------ | ------ | ------ | ------- |
| `!!int8` | - | `int8_t` | `byte` | `c_byte` |        |        |        | - |
| `!!uint8` | - |  | - |  | | | | - |
| `!!int16` | - | `int16_t` | `short` | `c_short` |        |        |        | - |
| `!!uint16` | - |  | - |  | | | | - |
| `!!int32` | - | `int32_t` | `int` | `c_int` |        |        |        | int |
| `!!uint32` | - |  | `uint` |  | | | | - |
| `!!int64` | - |  | `long` |  | | | | int |
| `!!uint64` | - |  | `ulong` |  | | | | |
| `!!int` | integer | - | `BigInteger` | `int` | | | | |
| `!!uint` | natural | - | `BigInteger` | `int` | | | | |
| `!!number` |  | - |  |  | | | | |
| `!!frac` | rational |  |  |  | | | | |
| `!!bool` | -       | bool   | boolean |        |        |        |        | boolean |
| `!!str` | -       | char * | String  | str    | string | String | string | string  |
| `!!float` | real |        |         |        |        |        |        |         |
| `!!float32` | - |        |         |        |        |        |        |         |
| `!!float64` | - |        |         |        |        |        |        |         |
| `!!float128` | - |        |         |        |        |        |        |         |
| `!!decimal32` | - |        |         |        |        |        |        |         |
| `!!decimal64` | - |        |         |        |        |        |        |         |
| `!!decimal128` | - |        |         |        |        |        |        |         |
| `!!complex` | complex |        |         |        |        |        |        |         |
