# UON Unified Object Notation (UON ™ )

## Draft Version 0.0.1 <!-- omit in toc -->

**This version:** http://github.com/uon-language/specification/blob/spec.md

**Bug tracker**: http://github.com/uon-language/specification/issues

**Editor**: Yves Chevallier <ycr@x0x.ch>

Copyright© 2020 Yves Chevallier

- [UON Unified Object Notation (UON ™ )](#uon-unified-object-notation-uon-%e2%84%a2)
  - [Introduction](#introduction)
  - [Scope](#scope)
  - [Normative references](#normative-references)
  - [Terms and definitions](#terms-and-definitions)
  - [Why UON™?](#why-uon%e2%84%a2)
  - [Language](#language)
    - [UON Related to other standards](#uon-related-to-other-standards)
      - [Related to JSON](#related-to-json)
      - [Related to YAML](#related-to-yaml)
      - [Related to XML](#related-to-xml)
      - [Related to Protocol Buffers](#related-to-protocol-buffers)
    - [Self described (mostly)](#self-described-mostly)
    - [Information Model](#information-model)
    - [Symbol Pairs](#symbol-pairs)
  - [Syntax and Grammar](#syntax-and-grammar)
    - [Reserved Symbols](#reserved-symbols)
    - [Example](#example)
    - [Other examples](#other-examples)
  - [Comments](#comments)
  - [Formatting](#formatting)
    - [Binary](#binary)
    - [Minimal](#minimal)
    - [Canonical](#canonical)
  - [Datatypes](#datatypes)
    - [Data Structures (or Structural types)](#data-structures-or-structural-types)
    - [Scalar](#scalar)
      - [Numbers Datatypes](#numbers-datatypes)
      - [Rational, Complex and Quaternion](#rational-complex-and-quaternion)
    - [Sized Datatypes](#sized-datatypes)
      - [Arbitrary size](#arbitrary-size)
      - [Fixed size](#fixed-size)
    - [Rich Datatypes](#rich-datatypes)
    - [Constraint Datatypes](#constraint-datatypes)
  - [Properties](#properties)
    - [Application](#application)
    - [Presentation](#presentation)
    - [Validation](#validation)
    - [Example](#example-1)
  - [Type details](#type-details)
    - [Type](#type)
    - [Null and Booleans](#null-and-booleans)
    - [String](#string)
    - [Number](#number)
      - [Precision](#precision)
    - [Keyword](#keyword)
    - [References](#references)
    - [Binary, Octal and Hexadecimal representation](#binary-octal-and-hexadecimal-representation)
    - [IPv4](#ipv4)
    - [Blob](#blob)
    - [Emails](#emails)
  - [Physics](#physics)
  - [Units](#units)
    - [Base Units](#base-units)
    - [Derived Units](#derived-units)
    - [Other accepted units](#other-accepted-units)
    - [Units of information](#units-of-information)
    - [Astronomical units](#astronomical-units)
    - [Prefix](#prefix)
    - [Derived quantities and units](#derived-quantities-and-units)
  - [Coercion](#coercion)
  - [Validation](#validation-1)
    - [Validation schemas](#validation-schemas)
      - [Embedded schema](#embedded-schema)
      - [Included schema](#included-schema)
      - [Linked schema](#linked-schema)
      - [Separated schema](#separated-schema)
    - [Validation Properties](#validation-properties)
  - [Advanced features](#advanced-features)
    - [Encryption](#encryption)
    - [Capabilities](#capabilities)
    - [Destructuring](#destructuring)
  - [RESTful access](#restful-access)
  - [Raw Conversion](#raw-conversion)
    - [JSON](#json)
    - [XML](#xml)
    - [YAML](#yaml)
  - [File format and extensions](#file-format-and-extensions)
  - [Unified API](#unified-api)
  - [Implementations](#implementations)
    - [Parser](#parser)
  - [Syntax Highlight and support in text editors](#syntax-highlight-and-support-in-text-editors)
  - [Evaluations](#evaluations)
- [Binary Encoding](#binary-encoding)
    - [!ipv4](#ipv4-1)
    - [!jwt](#jwt)
    - [!blob](#blob-1)
- [Annexes](#annexes)
  - [Regulations and standards](#regulations-and-standards)
  - [Available Types](#available-types)
  - [Reserved Keywords](#reserved-keywords)
  - [Regex validation](#regex-validation)
  - [Type names vs other languages](#type-names-vs-other-languages)
  - [Examples](#examples)
    - [Git repository](#git-repository)
    - [Temperature Sensor](#temperature-sensor)

## Introduction

Still in 2020 three major serialization formats are commonly used across the Internet: `XML`, `JSON` and `YAML`. Both are great, simple and well specified, but many would argue that:

* `XML` is great but cumbersome
* `JSON` is awesome but lack validation by design
* `YAML` is human friendly but unsafe and not ideal for m2m communication

Seen from another perspective:

* `XML` is fully specified with validation schema `XSD` and representation `XSLT`
* `JSON` is lightweight and understood by most developers and programming languages
* `YAML` is very readable and a superset of `JSON`

*UON™* aims to steal all the amazing features of these serialization formats into a single format that encompass them.

Also, *UON* would like to go a step further by adding support for validation schema and magnitude units to extend interoperability in the sense of the Industry 4.0. The internet of things (IoT) often require to communicate between small devices with very limited computation power. The perspective to have an application communication protocol that is both interoperable and suitable for low power exchanges is more than welcome. *UON* is an attempt to serve these two needs.

When two devices want to exchange information they have two commuinication channels:

- An online communication channel used for payload data transmission
- A contractual communication channel used for schema agreement

In the following figure, we have on the left, two devices: a low power temperature sensor, and a powerful home automation gateway. To reduce the payload size, both devices agree on a schema which describe the payload format.

![](assets/data-exchange.png)

On the right side of the figure, the generic schema that describe a temperature sensor is **refined** by narrowing the schema for instance by saying the generic temperature expressed with a number, is now an unsigned eight bit value expressed in Celsius degrees.

## Scope

*UON*™ pronounced "You On" is a human-machine-friendly serialization language designed to serve the purpose of:

* Transmitting data across digital mediums
* Being human readable and human editable
* Being close to international standards and the largest consensus
* Serializing/deserializing data across different programming languages
* Unambiguous strict representation by design
* Support references and merge
* Support properties
* Standard API
* Binary encoding payload for low power embedded devices
* Partial zero-copy operations for binary payloads
* Validation schema supported by design (Inspired from [JSON-Schema](http://json-schema.org/) and [Voluptuous](https://pypi.org/project/voluptuous/))
* Transformation schema supported by design (inspired from what `XSLT` is to `XML`)
* Platform neutral
* Endianess neutral

## Normative references

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED",  "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119.

## Terms and definitions

For the purposes of this document, the following terms and definitions apply.

* **RFC** Request for comments
* **UON**  Unified Object Notation, specified in this document
* **ISO** International Standardization Organization
* **YAML** Yet Another Markup Language
* **JSON** JavaScript Object Notation

## Why UON™?

The [IoT](https://en.wikipedia.org/wiki/Internet_of_things) world is nowadays dominated by [XML](https://www.w3.org/TR/xml/), [JSON](https://www.json.org/) and [YAML](http://yaml.org/). Other serialization protocols such as [Protocol Buffers](https://en.wikipedia.org/wiki/Protocol_Buffers) aim to a better interoperable solution by offering a programming language agnostic solution. Unfortunately, none of them are complete. They focus only on narrow areas instead of addressing a more general solution that encompass low-power MCU with limited computation sources, web services and high-end applications.

UON™ would like to federate the IoT world by taking in account the [Industry 4.0](https://en.wikipedia.org/wiki/Industry_4.0) needs. In other words, the Unified Object Notation is meant to describe messages sent and received from:

* Battery-powered telemetry sensors
* Web APIs
* IPC

To achieve this goal, UON defines a way to:

* Encode/Decode payloads either written in binary or in human readable form
* Describe a payload through a validation schema

## Language

*UON*™ is not a programming language, but a data representation language. It is built upon concepts described by C, Perl, Python, Ruby and JSON. It is meant to be UTF-8 by default, but compatible with other encoding standards.

*UON* features four levels of complexity that each of them add more features. Remember that **UON** is a superset of **JSON**.

* **UON:0** is stricly speaking **JSON**, keywords have to be double quoted, comments are not permitted and well, it is simply **JSON**.
* **UON:1** encompass some of the **YAML** standard. It is a reduced set of **YAML**.
* **UON:2** adds type properties which allows to qualify a type, for instance by setting the base of a number `!int(base:36) ah1ddb1`.
* **UON:3** extends **UON:2** by adding extended types and syntaxic sugars to simplify the readability.

![](assets/complexity-levels.png)

An interesting point is that **UON:3** could still be coerced in **UON:0**. For instance, the following **UON:3** code can be written in **JSON**

```yaml
{
  person(desc: "A person"): {
    name: !str(length: 42, pattern: /[A-Za-z ]+/) "John Doe"
    age: !int(base:4, dictionary: ["Ga", "Bu", "Zo", "Meu"]) "Ga Ga Zo Meu Bu"
  }
}
```

Here the equivalent **JSON** format:

```json
{
  "key": "person",
  "properties": {
    "desc": "A person"
  },
  "value": {
    "name": {
      "type": "str"
      "properties": {
        "length": 42,
        "pattern": {
          "type": "regex",
          "value": "[A-Za-z ]+"
        },
      }
      "value": "John Doe"
    }
    "age": {
      "type": "int",
      "properties": {
        "base": 4,
        "dictionary": ["Ga", "Bu", "Zo", "Meu"],
      },
      "value": "Ga Ga Zo Meu Bu"
    }
  }
}
```

The following figure graphically shows the different layers of *UON*:

![](assets/supersets.png)

### UON Related to other standards

#### Related to JSON

Both *JSON* and *UON* aim to be human readable data interchange formats. *UON* extends *JSON* capabilities and readability with:

* More data types support
* Allow no double quotes on mapping keys since they are promoted to a `keyword` type rather than `string`
* Validation schema by design inspired by JSON-Schema
* Allow type coercion
* Add support for comments
* Add support for set, and ordered mapping

#### Related to YAML

*YAML* is very complete, it supports references (recursive references), comments, complex types, but its syntax lack of rigidity. They are multiple way of writing a *YAML* which can be confusing to some people and hard to validate. *UON* extends some *YAML* capabilities by adding:

* More data types support
* Strict formatting closer to *JSON*
* Validation schema by design
* Resolved and unresolved references
* Unambiguous comment position

In *YAML* it is possible to create a reference to an existing location in the tree. However this is always resolved after parsing which means, the reference information is lost. *UON* supports unresolved references in which even after parsing the document, the reference information is preserved.

Comments in *YAML* are not formally part of the object tree, but they are somehow supported by `ruamel.yaml` Python package with the `round_trip_load` method. In *UON*, comments are part of the language, and they appear in the object tree.

One major difference with *YAML* is the tag notations. On *YAML* native tags are noted using two exclamation marks e.g. `!!int`, but user types have only one `!`, they are so-called non-specific tag. *UON* aims to be simple and lightweight. Using an additional `!` for native types would encourage user types as they are quicker to read. So in *UON* native tags are primary tags.

#### Related to XML

*XML* has proven its strength over many years. It is still a very complete language, but it is also more verbose than other serialization languages which make it difficult to use with a lightweight infrastructure such as battery powered MCU used on telemetry embedded devices. However it provides a full support for validation (*XSD*) and presentation (*XSLT*). *UON* took the tags attributes that are missing from *JSON* and *YAML* which as called: type properties.

#### Related to Protocol Buffers

Protocol buffers is a relatively new standard developed by Google which allows binary payload generations. So it defined some advanced types. Platform neutral serialization language developed by Google.

*UON* got inspired from Protocol Buffers by the following

* `repeated`, Fields can be repeated any number of times
* Fixed size scalar values e.g. `int32`, `uint64`, `sint32`
* Enumerations
* One-of keyword

### Self described (mostly)

*UON* is aimed to be mostly self described in the sense that most of the *UON* standard is defined using *UON*. For example, the ipv4 type implementation does not relies on language specific code. It is defined is pure *UON*:

```
!ipv4: !schema("http://uon.io/ipv4") !str(
  description: "Type representing an Internet Protocol version 4 address defined in RFC 760",
  links: [
    !url "https://tools.ietf.org/html/rfc760"
  ]
  pattern: /^((?&b))\.((?&b))\.((?&b))\.((?&b))(?(DEFINE)\b(?<b>25[0-5]|2[0-4]\d|[01]?\d\d?))$/,
  coerce: {
    !seq: [!int @@$[0], !int @@$[1], !int @@$[2], !int @@$[3]],
    !int: !eval [
      @@[0],
      @@[1], 256, !mul, !add,
      @@[2], 65535, !mul, !add,
      @@[3], 16777216, !mul, !add
    ]
  },
  binary: [
    !uin8 @@[0], !uin8 @@[1], !uin8 @@[2], !uin8 @@[3]
  ]
  properties(merge=true): {
    mask(optional: true): !bool(default: false)
  }
)
```

### Information Model

The *UON™* information model is shown by this figure below. Each item that composes *UON* object tree is  a `Type` that may have several properties. The content can be either a `Scalar` or a `Collection` of items.

Three kind of properties exists:

* Presentation properties
* Validation properties
* Application properties
*
![](assets/information-model.png)

### Symbol Pairs

Symbol pairs are syntactic sugars used to enhance readability by associating a specific symbol pair to a particular type of data.

| Symbol Pairs | Description                    | Type     |
|--------------|--------------------------------|----------|
| `[ ]`        | Ordered Sequence               | `!seq`   |
| `{ }`        | Mapping (by default unordered) | `!map`   |
| `" "`        | String                         | `!str`   |
| `/ /`        | Regular expression             | `!regex` |
| `$ $`        | Mathematical expression        | `!math`  |
| `( )`        | Properties                     | -        |
| `@( )`       | Unresolved reference           | `!ref`   |
| `@@( )`      | Resolved reference             | `!ref`   |
| `!@()`       | Reference to a type            | `!ref`   |
| `< >`        | Structural container           | -        |

Here is an example that uses all the supported symbol pairs:

```yaml
{
  data: !seq(sort: asc) [3, 1, 2],
  name: "John Doe",
  pattern: /\b[a-z]\d{5}(?=0{3..4})-\w+/i,
  equation: $\frac{1}{x}$
}
```

Structural containers are not meant to be directly used, but they can show how data are imbricated. The idea was inspired from *YAML* and it helps to understand hierarchical relationship in a *UON* description:

```yaml
!uon<version: !version<0.0.1>>
<
  !map<comment: <Basic Example>>
  <
    <!keyword<brand>: !str<Toyota>>,
    <!keyword<model>: !str<Prius>>,
    <!keyword<year>: !int<2016>>
  >
>
```

Note that the above example should be written as follows in structured format:

```yaml
# Basic Example
{
  brand: "Toyota",
  model: "Prius",
  year: 2016
}
```

Or in its minimal version:

```yaml
{brand:"Toyota",model:"Prius",year:"2016"}
```

Or even in its binary version:

```text

```

## Syntax and Grammar

The *UON* syntax can be expressed by the following figure:

![](assets/syntax.png)

Everything is a value with an associated type and properties. A *key* is a value of type `!key`. Types can be nested so the following is valid, but not minimal:

```yaml
!key !str "sequence": !seq [
  !int !uint !int8 42,
  {
    !str: !int !uint
    !int: !str "Hello"
    !foo: 42
  }
]
```

Every *UON* description can be reduced to its canonical form:

```yaml
sequence: [
  42,
  {
    !str: 0
    !int: "Hello"
    !foo: 42
  }
]
```

### Reserved Symbols

The only reserved language symbols are the following

| Symbol  | Description                                                  |
| ------- | ------------------------------------------------------------ |
| `:`     | Key value separator                                          |
| `::`    | Type assignation, only valid with a non-existing user type as a key |
| `,`     | Element separator                                            |
| `!`     | Standard type (cannot be defined at the user level)          |
| `!!`    | User type used to extend *UON* capabilities.                 |
| `@`     | Resolved reference                                           |
| `@@`    | Unresolved reference                                         |
| `#`     | Comment                                                      |
| `.inf`  | Infinity (used within `!number`, `!float`, or `!decimal`)    |
| `.nan`  | Not a Number (used within `!number`, `!float`, or `!decimal`) |
| `true`  | Boolean (`!bool`) value `true`                               |
| `false` | Boolean (`!bool`) value `false`                              |
| `null`  | Null (`!null`) value                                         |

### Example

Here a comprehensive *UON* example:

```yaml
# Overview of what UON is capable of
!uon(version: 0.0.1) {
  # Mapping, a dictionary of key-value pairs with unique keys
  map: {
    key-1: "A value",
    key-2: "Another value"
  },
  # Ordered mapping, but keys are unique
  omap: !omap {
    foo: 1,
    bar: 2
  },
  # Sequences
  seq: [1, 2, 3],
  # Unordered set of unique values
  set: {"foo", "bar"},
  # Ordered set of unique values
  oset: !oset {"bar", "foo"},
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
      decimal: -1 432 879.09,
      int: 1408,
      hex: 0x24321,
      oct: 0o19283,
      bin: [
        0b10001, # Sugar syntax
        !int(base: 2) 10001, # Explicit base property
        !int(base: 2) [1, 0, 0, 0, 1] # Coercion from a sequence
      ],
      float: [12.132e6, -.inf, .nan],
      complex: 12 + 4j,
      quaternion: 12 + 12i + 23j + 43k,
      quantity: {
         length: [
           12m, # Sugar syntax
           !num(unit: "inches", uncertainty: [1, 2]) 32 # Explicit form with plus 1, minus 2
         ],
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
  	merge: !map(merge: @scalar.number) {
       int: 1409 # Override existing value
  	}
  },
  # Rich types
  rich: {
     uuid: !uuid "82584ce5-d086-41ff-978f-57323ebf5b9d",
     jwt: !jwt "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpYXQiOjF9.9GeY7G6YbEh90kFhyhqsL29uFQCbIlk2Y-YVhae61g0",
     base64: !blob(encoding: base64) "W55IGNhcm5hbCBwbGVhc3VyZS4="
  }
  # Validation schema
  schema: !schema("http://schema.org/Book") {
    abridged: !bool,
    bookEdition: !str,
    bookFormat: !@("http://schema.org/Person"),
    isbn(optional: true): !@("urn:uon:isbn"),
    numberOfPages: !uint32(
    	description: "Represent the number of page of a book"
    	min: 1
    ),
    text(dummy: true): !str(
      description: "This dummy field can be used as a reference by other fields"
    ),
    about: !@(.dummy),
    accessMode: !@(.dummy)
    genre: !oneof("Male", "Female"),
    link: !oneof(!url, !urn, !uri)
  },

  # Schema Type
  !str: !schema("http://uon.io/str") !type(
    description: """
       Base string encoded in UTF-8
    """t,
    properties: {
      min(optional: true): !uint(default: 0),
      max(optional: true): !uint(default: .inf), greater-than: @.min),
      pattern(optional: true): !regex,
    }
  )
}
```

### Other examples

```yaml
# Basic UON example
!uon(version: 0.0.1) !("urn:car") {
  brand: "Toyota",
  model: "Prius",
  year: 2016
},

# Type description for a !car, and validation schema
!uon(version: 0.0.1) !schema("urn:car") {
  brand: !str(pattern: /[A-Z]\w+/),
  model: !str(pattern: /[A-Z]\w+/),
  year(required: false): !dec(min: 1930)
  color(required: false): !oneof(
     !str,
     !number
  )
}

# Coersion is implicit, the parsed result is `12.`
!uon !float !str 12
```

## Comments

Inline comments with `#` are never serialized when transmitted unless explicitly mentioned. Comment with `#` are a syntactic sugar to fill the `comment` property of the related object. A comment is associated to the closest key-pair value.

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
!map(comment: "Comment global to the document")
{
  !keyword(comment: "Comment associated to the key-pair { brand: "Toyota" }") brand: "Toyota",
}
```

## Formatting

*UON* provide three different formats depending on the usage:

* **binary** for m2m communication with lightweight architectures
* **minimal** for m2m communication in UTF-8
* **canonical** for storing, visualizing and editing
* **free** in which nested types can appear

### Binary

* Payload is encoded in a strict binary form
* LED128 format for numbers
* Padding is preserved as much as possible to allow zero-copy payload read/alter

Let's consider a car:

```yaml
{
  model: "Prius",
  brand: "Toyota",
  year: 2016
}
```

It would be encoded as follow:

```text
\x12\x00\x11\x00Toyota\x00\x12\x00model\x00\x11\x00Prius\x00\x12\x00year\x00\x29\x00\xe0\x0f
```

The encoding takes 38 bytes and the year is encoded using the LEB128 format.

- `\x12`: `!keyword`
- `\x00`: No properties
- `\x11\x00Toyota\x00`: String with no properties, with value and terminator
- `\x12\x00model\x00`: Keyword with no properties, with value and terminator
- `\x11\x00Prius\x00`: String with no properties, with value and terminator
- `\x12\x00year\x00`: Keyword with no properties, with value and terminator
- `\x29`: Variable length integer in LEB 128 format
- `\x0e\x0f`: LEB 128 payload

Note that this binary payload can be reduced by using a description schema. If the parser is aware of what the type `!!car` is, it could just decode the payload encoded as follow:

```text
Toyota\x00Prius\x00\xe0\x0f
```

Of course, the goal of *UON* is to generate minimal payload, so we could reduce it further by transfering more information to the validation schema:

1. If the schema lists the different allowed brands only the id of the brand will be on the payload
2. The year could be expressed as an `!uint8` with an offset of `2000`

```text
\x01\x00\x10Toyota\x00
```

So here the schema for this particular encoding:

```
!!car: !schema {
  brand: !enum(values: ["Opel", "Toyota", "Audi", "Seat", "Ford"]),
  model: !str(max: 128, capitalized: true),
  year: !uint8(offset:2000)
}
```

Notice that in the above example, the key orders has been reversed, `Toyota` is now located at the end of the payload. This is perfectly wanted to allow zero-copy because the model name is the only variable size payload.

Binary encoding follow these rules:

- Keys are ordered by alphabetical orders unless they are ordered sequence
- Variable length values are put after the fixed size values

### Minimal

The minimal form is a canonical form with no spaces and no comments:

* Spaced and new lines are removed
* Comments are removed unless configured differently

```yaml
!uon(version:0.0.1)car{brand:"Toyota",model:"Prius",year:2016}
```

### Canonical

The canonical form is a two space indentation with coerced multiple types. For instance the `!int(base: 10) !str "123"` is reduced to `123`.

* Format uses two space indentations, same as JSON and YAML, but strictly imposed by design
* Explicit types are converted into implicit ones if not necessary
* Comments are written using the `#` notation

```yaml
# This is a car...
!uon(version: 0.0.1) !car {
  brand: "Toyota",
  model: "Prius",
  year: 2016
}
```

## Datatypes

One important feature of *UON* is that any kind of information belongs to a type. Types are organized in different categories:

### Data Structures (or Structural types)

![](assets/type-base.svg)

They are primitive data types all derived from the `!type` master data type.

| Type      | Based on | Description                                                |
|-----------|----------|------------------------------------------------------------|
| `!type`   | -        | Base type                                                  |
| `!map`    | `!type`  | Unordered Mapping (also called *HashMap* or *Dictionary* ) |
| `!omap`   | `!type`  | Ordered Mapping                                            |
| `!seq`    | `!type`  | Ordered Sequence (also called *List* or *Array*)           |
| `!scalar` | `!type`  | Scalar value representable with a string                   |
| `!set`    | `!map`   | Unordered set of values                                    |
| `!oset`   | `!omap`  | Ordered set of values                                      |

*UON* aims to represent static data which are therefore immutable by design. This explains the absence of Records or Tuples. However, when a composite type is used as a key, it will be parsed as an immutable type if the destination language allows it.

```python
>>> # Example in Python
>>> uon.loads("{[1,2]:[1,2]}")
{(1,2):[1,2]}
```

Unordered mapping (`!map`, `!set`) are always sorted naturally when serialized e.g. `[1, 2, 10, 100, a]`

Sets are mapping with null values

### Scalar

Scalar values are any kind of values that may be represented as a string

| Type       | Based on  | Description                                                                            |
|------------|-----------|----------------------------------------------------------------------------------------|
| `!null`    | `!scalar` | Null type, can be associated to other type e.g. `!null(!!str)`                         |
| `!bool`    | `!scalar` | Boolean, `true` or `false`                                                             |
| `!str`     | `!scalar` | String encoded in `UTF-8`                                                              |
| `!number`  | `!scalar` | Any kind of representable numeric value                                                |
| `!keyword` | `!str`    | Keyword used for mapping keys and references (`/^[a-z](?:(?:[a-z0-9-])*[a-z0-9])?$/i`) |
| `!ref`     | `!str`    | Reference to another type                                                              |

#### Numbers Datatypes

The number type is much more complete than JSON, XML or YAML. It aims to serve goals of engineering and scientific areas such as:

* Physics and Mathematics
  * Quantities with uncertainties and units
  * Complex and Quaternions
* Computer science
  * Low-level representation (hexadecimal, octal and binary)
  * Fractional values for frequency ratio
  * Fixed-Point values in

![](assets/numbers.png)

| Type         | Based on  | Description                                                        |
|--------------|-----------|--------------------------------------------------------------------|
| `!dec`       | `!number` | Decimal (i.e. Base 10) number e.g. `12345`                         |
| `!float`     | `!number` | Floating point `IEEE-754`                                          |
| `!bin`       | `!dec`    | Binary representation of decimal number                            |
| `!oct`       | `!dec`    | Octal representation of decimal number                             |
| `!hex`       | `!dec`    | Hexadecimal representation of decimal number                       |
| `!complex`   | `!float`  | Complex value e.g. `42+7j`                                         |
| `!magnitude` | `!number` | Physical value with an associated unit                             |
| `!frac`      | `!seq`    | Fraction of two integers, to represent repeating such as `0.33333` |
| `!fixpoint`  | `!dec`    | Fixed point value                                                  |

#### Rational, Complex and Quaternion

To allow UON to be used in astronomy, physics and mathematics it supports rational numbers $\mathbb{Q}$, complex number $\mathbb{C} $ and quaternions $\mathbb{H} $. There extended types inherit from a sequence where:

```yaml
{
  rational: 643 / 123
  complex: 12 + 6i,
  quaternion: 6 + 3i + 9j - 8k,
}
```

These types could be describes as follow:

```yaml
{
    !rational := !seq [ !int(desc: "Numerator"), !uint(desc: "Denominator") ],
    !complex := !seq [ !int(desc: "Real"), !int(desc: "Imaginary") ],
    !quaternion := !seq [
      !int(desc: "Real"),
      !int(desc: "i"),
      !int(desc: "j"),
      !int(desc: "k")
    ],
}
```

### Sized Datatypes

When serialized in binary format, the size of the payload has to be provided. If not, the serializing will use a 7-bit encoding where the 8th bit is used as a continuation bit formerly named [LEB128](https://en.wikipedia.org/wiki/LEB128). Variable size numeric values are less efficient at serializing and deserializing.

| Type         | Based on | Description                                              |
|--------------|----------|----------------------------------------------------------|
| `!int8`      | `!dec`   | `8 b` Signed Integer                                     |
| `!int16`     | `!dec`   | `16 b` Signed Integer                                    |
| `!int32`     | `!dec`   | `32 b` Signed Integer                                    |
| `!int64`     | `!dec`   | `64 b` Signed Integer                                    |
| `!int128`    | `!dec`   | `128 b` Signed Integer                                   |
| `!uint8`     | `!dec`   | `8 b` Unsigned Integer                                   |
| `!uint16`    | `!dec`   | `16 b` Unsigned Integer                                  |
| `!uint32`    | `!dec`   | `32 b` Unsigned Integer                                  |
| `!uint64`    | `!dec`   | `64 b` Unsigned Integer                                  |
| `!uint128`   | `!dec`   | `128 b` Unsigned Integer                                 |
| `!float32`   | `!float` | `32 b` Single precision floating point number            |
| `!float64`   | `!float` | `64 b` Double precision floating point number            |
| `!float128`  | `!float` | `128 b` Quadruple precision floating point number        |
| `!decimal16` | `!float` | `16 b` Single precision floating point number in base 10 |
| `!decimal32` | `!float` | `32 b` Single precision floating point number in base 10 |
| `!decimal64` | `!float` | `64 b` Single precision floating point number in base 10 |

#### Arbitrary size

The types `int`, `uint`, `float` and `decimal` are arbitrary the following is valid:

```yaml
{
  int: -564 645 543 123 241e143
  uint: +483813344e-122
  float: -1345.392828482943833e-44
  decimal: -1 345.392 828 482 943 833e-44
}
```

The number `564 645 543 123 241e143` also written:

```
6 464 554 312 324 100 000 000 000 000 000 000 000 000 000 000 000 000 000 000 000 000 000 000 000 000 000 000 000 000 000 000 000 000 000 000 000 000 000 000 000 000 000 000 000 000 000 000 000 000 000 000 000
```

Can be encoded in binary with this form:

```
<!int><leb128 length><data>
```

The length of this number is 521 bit, the length is therefore `0b1000001001` written in LEB128 we have `0b00001001 0b10000100` , the `!int` type is `0x29`. Eventually we will get this payload:

```
<0b101001><0b00001001 0b10000100><0b11110001000100110000000110100010010011111100010010001010000010100111010010101110101110011111100010110000101111100011011011000010101001110110010000111001111011110111111101110011011001101001011010000010101010000100101100000011011110100110011001001001011001101101110111010110101101010000110000011100011110101110101011001100010010100011011000010010010011110010100001111000101001010100000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000>
```

#### Fixed size

```yaml
{
  natural: {
    uint256: 115792089237316195423570985008687907853269984665640564039457584007913129,
    uint128: 340282366920938463463374607431768211455,
    uint64: 18446744073709551615,
    uint32: 4 294 967 295,
    uint16: 65 535,
    uint8: 255
  }
  integer: {
    int256: -5789604461865809771178549250434395392663499233282028201972879200395656481
    int128: -170141183460469231731687303715884105728
    int64: -9223372036854775808,
    int32: -2 147 483 648
    int16: -32 768
  }
  real: {
    radix-binary: {
      float128: .inf
      float64: .inf
      float32: 3.402823e+38,
    }
    radix-decimal: {
      floatd128: .inf
      floatd64: .inf
      floatd32: 3.402823e+38,
    }
  }
}
```

### Rich Datatypes

Rich data types are always based on the `!str` but they can be parsed using a regex pattern. Each rich type can be coerced into different format.

All these formats have a strict binary encoding format when the data is transmitted in binary.

| Type        | Based on | Description                                                               | Coercible in             |
|-------------|----------|---------------------------------------------------------------------------|--------------------------|
| `!version`  | `!str`   | [SemVer](https://semver.org/) 2.0.0 version number                        | `!seq`, `!map`           |
| `!regex`    | `!str`   | PCRE2 Regular expression                                                  |                          |
| `!time`     | `!str`   | Time value e.g. `12:43:01` (`ISO 8601`)                                   | `!seq`, `!map`           |
| `!date`     | `!str`   | Date value e.g. `1982-02-16` ( `ISO 8601`)                                | `!seq`, `!map`           |
| `!datetime` | `!str`   | Full date time e.g. ` 2018-06-08T23:02:51+00:00 ` ( `ISO 8601`)           | `!seq`, `!map`, `!epoch` |
| `!ipv4`     | `!str`   | IP version 4 described in [RFC 791](https://tools.ietf.org/html/rfc791)   | `!seq`, `!uint32`        |
| `!ipv6`     | `!str`   | IP version 4 described in [RFC 8200](https://tools.ietf.org/html/rfc8200) | `!seq`, `!uint128`       |
| `!jwt`      | `!str`   | JSON Web Token                                                            | `!seq`, `!map`           |
| `!uuid`     | `!str`   | Unique User Identifier, a 128-bit data                                    |                          |
| `!epoch`    | `!dec`   | UNIX timestamp                                                            | `!datetime`              |
| `!math`     | `!str`   | Mathematical notation `$\frac{2}{3}$`                                     |                          |

```yaml
{
  urn: urn:oasis:names:specification:docbook:dtd:xml:4.1.2,
  url: http://www.google.com,
  math: $\x_{1,2}=-b\pm\frac{\sqrt{b^2-4ac}}{2a}$,
  uuid: c339822f-c28d-4d30-b205-f4763820efb2,
  version: [
    1.2.35,
    1.0.0-alpha,
    1.0.0-x.7.z.92,
    1.0.0+20130313144700
  ]
  regex: /^#?([a-f0-9]{6}|[a-f0-9]{3})$/,
  datetime: [
    2018-06-16T18:58:54+00:00,
    2018-06-16T18:58:54Z,
    20180616T185854Z
  ]
  date: [
    2018-12-09,
    2018-W24,
    2018-W24-6
  ]
  time: 12:32:42
  ipv4: 192.168.1.6/23,
  ipv6: [
    2001:0db8:0000:0000:0000:ff00:0042:8329,
    2001:db8:0:0:0:ff00:42:8329,
    2001:db8::ff00:42:8329,
    ::1/128,
    ::ffff:0:0:0/96
  ]
  epoch: !epoch 1529178670,
}
```

### Constraint Datatypes

Constraint data types are only used for validation schema

| Type      | Based on | Description                                           |
|-----------|----------|-------------------------------------------------------|
| `!oneof`  | `!map`   | Alternative possibility e.g. `!oneof( !!str, !!null)` |
| `!schema` | `!type`  | *UON* Validation schema                               |

## Properties

Three kind of properties exist:

### Application

These properties can only be read from *UON* using the `!prop` type. They are accessible from the application (Python, JavaScript, ...). They are used to generate a serialized file (binary, or UON), but they are never explicitly passed.

### Presentation

Presentation properties are any property that influence the presentation of a value, such as `!seq(orderd: true) [9, 8, 7]` a resolved payload will be `[7, 8, 9]`

### Validation

Validation properties are used to validate, constrain and describe a *UON* file.

### Example

Properties can be seen as attributes passed to a constructor of a class. When writing:

```yaml
!uon(version: 0.0.1) {}
```

You essentially instantiate a `!uon` class with the attribute `version` set to `0.0.1`. This instance is inherited automatically by the following argument here `{}` which is a `!map`.  In *UON* only types can be instantiated, but since everything is static on a *UON* document, `!uon` and `!uon()` performs the same. It is not allowed to add properties to objects such as:

```yaml
{
  # Incorrect
  {foo: 1, bar: 2}(ordered:true)

  # Correct
  !map(ordered:true) {foo: 1, bar: 2}
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

### Type

Type is the base type of all other types

| Property Name         | Description                                     | Example                              |
|-----------------------|-------------------------------------------------|--------------------------------------|
| `comment: !str`       | Add a comment to an object                      | `!dec(comment: "The Answer") 42`     |
| `description: !str`   | Description for documentation purposes          | `!dec(description: "The Answer") 42` |
| `binformat: !keyword` | Encoding format used for binary payload         | `!dec(binformat: int32) 42`          |
| `optional: !bool`     | Is the value optional?                          |                                      |
| `content: !type`      | This is the content value of object (read only) |                                      |

The `content` property is not meant to be used directly, but it appears in the `UON` DOM.

### Null and Booleans

The three keywords `false`, `true` and `null` have a special meaning in UON. The `null` value indicates the absence of value. It is optional, though:

```yaml
{
  null: {
    explicit: null,
    implicit: , # Here null is implicit
  },
  bool: {
    false,
    true
  }
}
```

Boolean value defined by this schema

```yaml
!uon(version: 0.0.1) {
  !bool: !type(urn:uon:2018:types:bool)
    !oneof(
      set: {
        !keyword(coercion: {!number: 0}) false,
        !keyword(coercion: {!number: 1}) true
      },
      properties: {
        alias: !set {
          !key: !oneof(set: {false, true})
        }
      }
    )
}
```

| Property Name | Description                           | Example                       |
|---------------|---------------------------------------|-------------------------------|
| `alias: !set` | Adds accepted alternative for Boolean | `!bool(alias: {on: true}) on` |

### String

A string is an arbitrary sequence of characters encoded in [UTF-8](https://www.ietf.org/rfc/rfc3629.txt). Since *UON* is a text-based language, any other type can be coerced into a string.

| Format    | Example                    |
|-----------|----------------------------|
| Literal   | `"A literal string"`       |
| Raw       | `r"Here \n is not parsed"` |
| Multiline | `"""A multiline string"""` |
|           |                            |

```yaml
!uon(version: 0.0.1) {
  !str: !type(urn:uon:2018:types:string)
    !type(
      properties: {
      pattern: !regex,
        min: !uint,
        max: !uint(ge: @.min)
      }
    )
}
```

| Property Name     | Description                                          | Example                         |
|-------------------|------------------------------------------------------|---------------------------------|
| `pattern: !regex` | Acceptance pattern                                   | `!str(pattern: /0x[0-9a-f]+/i)` |
| `min: !uint`      | Minimum length                                       | `!str(min: 2)`                  |
| `max: !uint`      | Maximum length (must be greater or equal than `min`) | `!str(min: 2, max: 18)`         |

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

Numbers accept the [E-notation](https://en.wikipedia.org/wiki/Scientific_notation) with the letter `e` as  "times ten raised to the power of"

| Property Name                | Description                                          | Example                          |
|------------------------------|------------------------------------------------------|----------------------------------|
| `magnitude: !prefix`         | Order of magnitude                                   | `!number(magnitude: k) 12`       |
| `base: !uint(min:2, max:62)` | Positional notation (base-2, base-10...)             | `!number(base: 4) 222 # 42`      |
| `min: !number`               | Minimum accepted number                              | `!number(min: 2)`                |
| `max: !number(min: @.min)`   | Maximum length (must be greater or equal than `min`) | `!number(min: 2, max: 18)`       |
| `le: @.max`                  | Less than or equal to                                | `!number(le: 2)`                 |
| `ge: @.min`                  | Greater than or equal to                             | `!number(ge: 2)`                 |
| `lt: !number`                | Less than                                            | `!number(lt: 2)`                 |
| `gt: !number(min: @.lt)`     | Greater than                                         | `!number(gt: 2)`                 |
| `uncertainty: !number`       | Uncertainty value such as in `12 ± 0.01`             | `!number(uncertainty: 0.1) 12`   |
| `unit: !unit`                | Associated unit                                      | `number(unit: celsius) 21.3`     |
| `fix(default: 0): !uint`     | Fixpoint representation, value is divided by `!uint` | `number(base: 2, fix: 2) 110010` |

Positional notation uses the representable symbols in this order `"0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ"`, which makes the maximum possible representable base 62

#### Precision

The fact that floating-point numbers cannot precisely represent all real numbers justifies the difference between `!num` and `!float`.

For example the `!float32 0.1` is not representable in IEEE 754 binary format:

```
!num 0.100000001490116119384765625 == !float32 0.1
```

IEEE 754:2008 introduces a new format named `decimal` which has a radix of 10. This allows representing exactly emulated decimal which is useful for financial and tax computations. However, most implementation and processors do not support this format:

```
!num 0.1 == !decimal32 0.1
```

### Keyword

A keyword is essentially a string which can be represented without double quotes. It can be either `undefined` or `defined`. An undefined keyword is not defined in the search path, while the defined keyword is documented and defined somewhere in the search path.

```yaml
!uon(version: 0.0.1) {
  !keyword: !schema(urn:uon:2018:types:keyword) !!str(
     pattern: /^[a-z][a-z0-9-]+(?!-)$/i,
     properties: {
       enum: !!dec,
       coercion: {
         !number: @..enum
       }
     }
  )
}
```

| Property Name | Description       | Example                    |
|---------------|-------------------|----------------------------|
| `enum: !dec`  | Enumeration value | `!keyword(enum: 1e3) kilo` |

### References

A reference refers to another field on a *UON* document. A reference can be resolved or not when generating a document. It can be a local reference or a remote reference when the target has to be fetched outside from the document. References can also act as hyperlinks, this is especially the case when representing a UON document in a Web browser.

```yaml
!ref: !type(
  resolve(default: false): !bool,
  type: !str
) !oneof {
  !url,
  !str(pattern: /^\.\w+/ )
}
```

You can do absolute or relative references.

| Reference | Description    |
|-----------|----------------|
| `@.foo`   | This level     |
| `@..foo`  | First parent   |
| `@...foo` | Second parent  |
| `!@`      | Link to a type |

Parenthesis are not required, but recommended.

| Explicit                          | Syntactic sugar |
|-----------------------------------|-----------------|
| `!ref "..foo.bar"`                | `@(..foo.bar)`  |
| `!ref(resolve: true) "..foo.bar"` | `@@(.foo.bar)`  |
| `!ref(type: ".foo.bar")`          | `!@(.foo.bar)`  |

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


| Property Name                               | Description                             | Example                        |
|---------------------------------------------|-----------------------------------------|--------------------------------|
| `resolve(default: false): !!bool`           | Resolve reference at payload generation | `!ref(resolve=true) ..foo.bar` |
| `type(default: !null): !ref(resolve: true)` | Used as type                            |                                |

The schema of a reference is expressed as follows:

```yaml
!uon(version: 0.0.1) {
  !keyword: !schema(urn:uon:2018:types:reference) !oneof {
    !uri,
    !str
  },
  properties: {
    resolve(default: false): !bool
  }
}
```

URN references rely on an external UON document that resolve a urn. The resolve document is usually passed as follows `!uon(urn-resolve: @(http://some-domain.com))`. A set can be passed to `urn-resolve`

A possible value for this resolver is:

```yaml
# Resolver for URN
!uon(version: 0.0.1) {
   ietf: {
     rfc:
       2648: !url "https://www.ietf.org/rfc/rfc2648.txt"
   }
}
```

### Binary, Octal and Hexadecimal representation

```yaml

!uon(version: 0.0.1) {
  !bin: !schema(urn:uon:2018:types:binary) !!str(
     pattern: /^0b([01]+)$/,
     coerce: {
       !number: !number(base: 2) "$1"
     }
     properties: {
       size: !uint,
     }
  )
}
```

```yaml
# Value of this is `2.16015625`
!number !bin(fix: 8) 0b1000101001
```

### IPv4

The IPv4 type is defined in UON

```yaml
!ipv4: !!schema("http://uon.io/ipv4") !str(
  description: "Type representing an Internet Protocol version 4 address defined in RFC 760",
  links: [
    !url "https://tools.ietf.org/html/rfc760"
  ]
  pattern: /^((?&b))\.((?&b))\.((?&b))\.((?&b))(?(DEFINE)\b(?<b>25[0-5]|2[0-4]\d|[01]?\d\d?))$/,
  coerce: {
    !seq: [!int "$1", !int "$2", !int "$3", !int "$4"]
  },
  properties(merge=true): {
    mask(optional: true): !bool(default: false)
  }
)
```

### Blob

Blob are binary content that can be represented with as string with a chosen encoding.

| Encoding | Pattern                                             | Padding | Standard                                            |
|----------|-----------------------------------------------------|---------|-----------------------------------------------------|
| `base85` | `/[0-9a-zA-Z\.-:\+=\^!\/\*\?&<>\(\)\[\]\{\}@%$#]+/` | No      | Z85 variant                                         |
| `base58` | `/[A-HJ-NP-Za-km-z1-9]*=*/`                         | `=`     | Bitcoin, `base58.cpp`                               |
| `base64` | `/[A-Za-z0-9+\/]*=*/`                               | `=`     | [RFC 4648](https://tools.ietf.org/html/rfc4648) § 4 |
| `base32` | `/[A-Z2-7]*=*/`                                     | `=`     | [RFC 4648](https://tools.ietf.org/html/rfc4648) § 6 |
| `base16` | `/[0-9a-f]*/`                                       | No      | [RFC 4648](https://tools.ietf.org/html/rfc4648) § 8 |

| Prop-ID | Prop-Type    | Name                                              | Description                                      |
|---------|--------------|---------------------------------------------------|--------------------------------------------------|
| `0xff`  | Presentation | `encoding: !!key(set: $bin2str, default: base64)` | Type of encoding used for presentation           |
| `0xfe`  | Validation   | `modulo: !!uint(default: 1)`                      | Binary size in byte must be multiple of `modulo` |
|         |              |                                                   |                                                  |
### Emails

Unfortunately, UON does not have a type for emails. The main reason is because there is no regex that 100% works. The RFC5322 is not enough to check an email validity. One also has to refer to RFC5321, RFC3696 and RFC1034 and RFC1035. That said the email, `"()<>[]:,;@\\\"!#$%&'-/=?^_{}| ~.a"@example.org` is perfectly valid, but lots of parsers would not accept it. As it is not possible to satisfy everybody. UON does not define the email type. However a user type `!!email` could be defined for a specific application.

## Physics

Physical constants

## Units

An important information for telemetry systems and the Internet of Things is the unit of magnitudes. When transmitting a temperature, it is essential to know which is the unit used to represent that number. *UON* uses the ISQ system ([ISO/IEC 80000-1:2009](https://www.iso.org/standard/30669.html)). Currently none of the serialization formats listed [here](https://en.wikipedia.org/wiki/Comparison_of_data_serialization_formats) provide native support for units.

### Base Units

The International System of Units list 7 fundamental units that are used as standard. *UON* does not support imperial units deliberately because among the 195 countries in the world only three countries do not use the metric system: Liberia, Myanmar and the USA. However *UON*

| Unit symbol | Unit name | Quantity name                             |
|-------------|-----------|-------------------------------------------|
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
!number(quantity: temperature) 250.24
```

### Derived Units

The International System of Units assigns special names to 22 derived units, which includes two dimensionless derived units, the `radian` and the `steradian`.

| Unit Symbol | Unit Name      | Quantity name                      | Equivalents                                                          |
|-------------|----------------|------------------------------------|----------------------------------------------------------------------|
| `Hz`        | Hertz          | `frequency`                        | $1/\text{s}$                                                         |
| `rad`       | Radian         | `angle`                            | 1                                                                    |
| `sr`        | Steradian      | `solid-angle`                      | 1                                                                    |
| `N`         | Newton         | `[force, weight]`                  | $\text{kg}\cdot\text{m}/\text{s}^2$                                  |
| `Pa`        | Pascal         | `[pressure, stress]`               | $\text{kg}\cdot\text{m}^{-1}\cdot\text{s}^{-2}$                      |
| `J`         | Joule          | `[energy, work, heat]`             | $\text{kg}\cdot\text{m}^2\cdot\text{s}^{-3}$                         |
| `W`         | Watt           | `[power, radian-flux]`             | $\text{kg}\cdot\text{m}^2\cdot\text{s}^{-2}$                         |
| `C`         | Coulomb        | `electric-charge`                  | $\text{s}\cdot\text{A}$                                              |
| `V`         | Volt           | `voltage`                          | $\text{kg}\cdot\text{m}^{2}\cdot\text{s}^{-3}\cdot\text{A}^{-1}$     |
| `F`         | Farad          | `electric-capacitance`             | $\text{kg}^{-1}\cdot\text{m}^{-2}\cdot\text{s}^{4}\cdot\text{A}^{2}$ |
| `Ω `        | Ohm            | `[electric-resistance, impedance]` | $\text{kg}\cdot\text{m}^2\cdot\text{s}^{-3}\cdot\text{A}^{-2}$       |
| `S`         | Siemens        | `electric-conductance`             | $\text{kg}^{-1}\cdot\text{m}^{-2}\cdot\text{s}^{3}\cdot\text{A}^{2}$ |
| `Wb`        | Weber          | `magnetic-flux`                    | $\text{kg}\cdot\text{m}^2\cdot\text{s}^{-2}\cdot\text{A}^{-1}$       |
| `T`         | Tesla          | `magnetic-field`                   | $\text{kg}\cdot\text{s}^{-2}\cdot\text{A}^{-1}$                      |
| `H`         | Henry          | `electric-inductance`              | $\text{kg}\cdot\text{m}^2\cdot\text{s}^{-2}\cdot\text{A}^{-2}$       |
| `°C`        | Degree Celsius | `temperature`                      | $\text{K}$                                                           |
| `lm`        | Lumen          | `luminous-flux`                    | $\text{cd}$                                                          |
| `lx`        | Lux            | `illuminance`                      | $\text{m}^{-2}\cdot\text{cd}$                                        |
| `Bq`        | Becquerel      | `radioactivity`                    | $\text{s}^{-1}$                                                      |
| `Gy`        | Gray           | `absorbed-dose`                    | $\text{m}^2\cdot\text{s}^{-2}$                                       |
| `Sv`        | Sievert        | `equivalent-dose`                  | $\text{m}^2\cdot\text{s}^{-2}$                                       |
| `kat`       | Katal          | `catalytic-activity`               | $\text{s}^{-1}\cdot\text{mol}$                                       |

These derived units can be used either on a payload or on a validation schema:

```yaml
[
  !number(unit: "°C") 21.13,   # Explicit unit
  21.13 °C                     # Also valid
]
```

### Other accepted units

The SI system lack some useful units as listed below. Millimeter of mercury is part of UON because of its large use in medicine. The electronvolt is widely used in fundamental physics.

| Unit Symbol | Unit Name             | Quantity name | Equivalents                           |
|-------------|-----------------------|---------------|---------------------------------------|
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
|-------------|---------------|---------------|
| `b`         | Bit           | Base 2        |
| `B`         | Octet or Byte | Exactly 8 b   |
| `nat`       | Nat           | Base `e`      |
| `dit`       | Dit           | Base 10       |

### Astronomical units

| Unit symbol | Unit name         | Quantity name | Equivalents              |
|-------------|-------------------|---------------|--------------------------|
| `ly`        | Light year        | `distance`    | $9.46 \cdot \text{Gm}$   |
| `AU`        | Astronomical Unit | `distance`    | $63'240 \cdot \text{ly}$ |
| `pc`        | Parsec            | `distance`    | $0.3066 \cdot \text{ly}$ |

### Prefix

Prefix can be added in front of a unit

| Symbol | Name  | Factor                            | Power      |
|--------|-------|-----------------------------------|------------|
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
|--------|-------|---------------------------|----------|
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

If coerced in `!map`, it would become:

```yaml
value: !magnitude {
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

## Coercion

In *UON* data can be coerced into any compatible data. This is usually written by specifying the type of a value.

Here the value `42` is a `!uint` because it does not have any decimal information. However it is coerced into a signed integer with the explicit casting `!!int`.

```yaml
!int 42
```

In the example, one tries to coerce a signed integer into an unsigned one. This is not possible because the equality is not met anymore: `-42 != 42`

```yaml
!uint -42
```

However, in this example, the coercion is acceptable. It is possible to convert a Celsius temperature into a Kelvin temperature without information loss `!num(unit = celsius) 23.2 == !num(unit = kelvin) 296.35`. The criteria is the quantity. Both types have the same quantity.

```yaml
!float32(unit: kelvin) 23.2 °C
```

Again here, there is a information loss. Conversion is not reversible,

```yaml
!int 42.15 # Acceptable
!int !float 42.15 # Unacceptable
```

When coercing from one type to another, some properties cannot be altered.

```yaml
!num(base: 2, scale: 3, offset: 4) !num(base: 10, scale: 2, offset: 3) 12345
```

This is possible

## Validation

### Validation schemas

Validation is mostly related to *schemas*. *UON* offers 4 types of strategies:

* Embedded schema
* Included schema
* Linked schema
* Separated schema

Each solution has advantages and disadvantages

#### Embedded schema

An embedded schema is mixing data and constraints on the same dataset. The advantage is that the filler directly knows what's expected. The disadvantage is that the filler can modify the constraints.

```yaml
# Person with embedded schema
{
    firstname: !str(pattern=/[A-Z][a-z]+/) "John",
    lastname: !str(min: 2, max: 32) "Doe",
    age: !uint(
      min: 7,
      max: 77,
      desc: "Age of the person that is able to play table games"
    ) 35
}
```

#### Included schema

Included schema is pretty similar to embedded schema. The validation document is also included with the dataset.

The main advantage is readability. data set and data validation are therefore separated. The advantage is still that the filler can look at the schema, and self validate the content. Again the disadvantage is that schema could be modified. This can be prevented by adding a signature which needs to be validated against a trusted authority or certificate.

```yaml
# Person with included schema
!uon {
  data: {
    firstname: "John",
    lastname: "Doe",
    age: 35
  }
  schema: !type(
    signature: "absbdb3b2HJG43hreklj430932rHvdsj"
  ) {
    firstname: !str(pattern=/[A-Z][a-z]+/),
    lastname: !str(min: 2, max: 32),
    age: !uint(min: 7, max: 77, desc: "Age of the person that is able to play table games")
  }
}
```

#### Linked schema

Here the validation schema is linked to the dataset. The schema can still be accessible, but not modifiable. The main disadvantage is that the schema needs to be fetched from an external source.

```yaml
# Person with linked schema
!@(http://schema.example.com/person) {
    firstname: "John",
    lastname: "Doe",
    age: 35
}
```

#### Separated schema

In this strategy data and validation are separated and unrelated.

```yaml
# Person dataset
!!person {
    firstname: "John",
    lastname: "Doe",
    age: 35
}
```

```yaml
# Person schema
!!person: {
  firstname: !str(pattern=/[A-Z][a-z]+/),
  lastname: !str(min: 2, max: 32),
  age: !uint(min: 7, max: 77, desc: "Age of the person that is able to play table games")
}
```

### Validation Properties

| Property       | Description                                                                                                                                                                                    | Example                                                 |
|----------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------------------------------------------------|
| `optional`     | Type or key is optional, default `false`                                                                                                                                                       | `foo(optional: false): !!str(min: 10)`                  |
| `exclusive`    | Cannot be present along with                                                                                                                                                                   | `foo-or-bar: { !!exclusive( foo: !!str, bar: !!dec ) }` |
| `min`          | Minimum length or value (equivalent to `le`)                                                                                                                                                   | `age: !!int(min: 0, max: 200)`                          |
| `max`          | Maximum length or value (equivalent to `ge`)                                                                                                                                                   |                                                         |
| `repr`         | Representation format (coercive)                                                                                                                                                               | `!seq(repr: !complex)`, `[1, 20]` becomes `1+20j`      |
| `quantity`     | Quantity value for physical magnitudes                                                                                                                                                         | `!dec(quantity: "temperature")`                         |
| `dummy`        | The value is dummy and does not appear in the payload. Usually used as a reference on other fields                                                                                             |                                                         |
| `default`      | Default value when missing                                                                                                                                                                     | `enabled: !!bool(default: false)`                       |
| `merge`        | Merge the value with the inherited type, false by default                                                                                                                                      |                                                         |
| `offset`       | Offset added to the serialized value (used in binary transmissions)                                                                                                                            |                                                         |
| `scale-factor` | Multiplication factor (used in binary transmissions)                                                                                                                                           |                                                         |
| `gt`           | The value must be greater than this value                                                                                                                                                      |                                                         |
| `lt`           | The value must be greater than this value                                                                                                                                                      |                                                         |
| `ge`           | Greater or equal                                                                                                                                                                               |                                                         |
| `le`           | Lesser or equal                                                                                                                                                                                |                                                         |
| `description`  | Description of the key or type                                                                                                                                                                 |                                                         |
| `link`         | List of hyperlinks                                                                                                                                                                             |                                                         |
| `alias`        | Used to accept values as an alias of another one                                                                                                                                               | `!bool(alias: {on: true, off: false})`                  |
| `resolve`      | Resolve reference when serializing, default `false`                                                                                                                                            |                                                         |
| `hash`         | Hash of the associated value computed on the binary format. This is useful to make sure a document as not been modified. The hash value can be an external reference. It is computed in SHA256 |                                                         |
| `signature`    | Any value can be signed using a digital signature.                                                                                                                                             |                                                         |

## Advanced features

### Encryption

A UON document can be encrypted. If so, the content of the document is simply a string value usually in base64. UON MUST support AES CBC (128, 192, 256 bit)

```yaml
!uon {
  login: "John",
  password: "sequoiaTree"
}
```

Here is what should look the encrypted payload with the passcode `UON`:

```yaml
!uon(encryption: aes128) !base64 "XWTrUQIavBWU58SXjpdmZAe/NX4C7Xc1nMQenvK0oKzzyWnwfe8Y5UkmMibT1mwG"
```

The encryption use the binary format as input to AES algorithm

Let's take an example of the encryption use. Two parties would like to exchange launch codes for interplanetary missiles. The payload content is made public:

```yaml
!!missile-code := !map(extensible: false) {
  alpha: !str(pattern: /(\da-z){12}/),
  beta:  !str(pattern: /(\da-z){12}/),
  gamma: !str(pattern: /(\da-z){12}/)
}
```

However the refined schema is only known by both parties Alice and Bob.

```yaml
!!missile-code := !@(http://somewhere.com/missile-code)(
  refine: !!missile-code,
  encryption: aes128,
  password: "top-secret"
)
```

Let's see how it works:

```python
# Bob and Alice
>>> password = "top-secret"
... parser = Parser(schema: """
...   !!missile-code := !@(http://somewhere.com/missile-code)(
...     refine: !!missile-code,
...     encryption: aes128,
...     password: "top-secret"
... )
... """ % password)

# Bob
>>> client.write(parser.from_dict({
...     'alpha': 'abfw594324qi',
...     'beta':  'a59fjc432wqe',
...     'gamma': '95jgfl324nfs',
... }).to_binary())

# Alice
>>> parser.from_binary(client.read()).to_dict()
{
    'alpha': 'abfw594324qi',
    'beta':  'a59fjc432wqe',
    'gamma': '95jgfl324nfs',
}

# Eve
>>> from uon import Parser
>>> parser = Parser(schema: "http://somewhere.com/missile-codes")
>>> parser.from_binary(sniffer.read()).to_uon()
"XWTrUQIavBWU58SXjpdmZAe/NX4C7Xc1nMQenvK0oKzzyWnwfe8Y5UkmMibT1mwG"
```

What is awesome is that the encryption is transparent for both Alice and Bob. When they decode the binary payload against the refined schema, the content is automatically decoded and parsed. Thus, disabling or changing the encryption method only happens on the refined schema, not on the application code.

### Capabilities

As UON is made to increase interoperability, the UON standard may not be fully supported, also extensions to the language could be added by third-party people. To ensure two communication nodes can communicate together, they can share minimal UON document with their capabilities such as:

```yaml
!capabilities {
  encryption: {
    aes
  }
  formats: {
    minimal: true,
  	binary: true,
  }
}
```

### Destructuring

With the help of references (`!ref`), *UON* data can be easily unstructured to only access a subset of data. This might be useful when used within a RESTful API.

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

*UON* source can be serialized in other formats such as YAML, XML or JSON with respect of their capabilities and limitations.

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

They are two ways of generating a JSON:

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

In the context of this draft, no implementation is fully available, however, projects in development are available below:

| Language   | Link                                  | Status |
|------------|---------------------------------------|--------|
| C          | -                                     | TBD    |
| C++        | -                                     | TBD    |
| C#         | -                                     | TBD    |
| JavaScript | -                                     | TBD    |
| Perl       | -                                     | TBD    |
| Python     | http://github.com/uon-language/py-uon | TBD    |
| PHP        | -                                     | TBD    |

### Parser

UON Parser aims to be very fast so it relies on two passes:

* Rough pass to identify the base structure of the document. A PreParsed object is created with the original text content and a dictionary of `Element` which have
  * start position
  * end position
  * identified type (if identified)
  * object, the parsed object when available
  * The idea is not to descend through all the trees. Long and complex strings can be parsed at a glance by just looking at the first non-escaped closing `"`.  The preparsed scalar.
    * The type is identified so we have `(0, 32, 'int')`
    * The type isn't identified yet so we have `(0, 32, None)`
    * The type can only be a list of some types `(0, 32, ('int', 'float'))`
* Second pass
  * The Object representation is built from the first pass but it is built on request.

## Syntax Highlight and support in text editors

I believe *UON* will only be used by others when its syntax will be fully supported by all the major text editors.

| Editor            | Link | Status |
|-------------------|------|--------|
| Microsoft VS Code | -    | TBD    |
| Vim               | -    | TBD    |
| Atom              | -    | TBD    |

One short term goal is to write a UON parser, then integrate this parser to Visual Studio Code as by 2019 it becames the most popular editor.

## Evaluations

*UON* should allow simple evaluations using the postfix notation because it is easier to implement in different languages and can easily be represented with a sequence.

```yaml
!eval: !schema() !seq(min: 2) [
  !number,
  !add, !sub, !mul, !div
  !not, !and, !or,
  !ref(resolve: false)
]
```

Adding the support for evaluations would allow to provide support for a new unit conversion. Presenting a temperature value into a different compatible unit should be possible. Units are not types, they are built into the number class.

To coerce a unit into another we must make sure the values we use as input are good. In other words, we must check the quantity of both sides are the same. How to properly do this?

Let's start from this:

```yaml
!number: !schema(
    coerce: {
      !number(unit: celsius): !number(quantity: temperature) !eval [212.234, @@.content, 543.1, !mul, !add],
      !number(unit: farheneit): !number(quantity: temperature) !eval [212.234, @@.content, 543.1, !mul, !add],
      !number(unit: kelvin): !number(quantity: temperature) !eval [212.234, @@.content, 543.1, !mul, !add],
    }
)
```

As *UON* would not allow to change the quantity of a given number. One must check this quantity is not changed. This possible by coercing the result of the evaluation with a number associated to a given quantity. We should notice that the following will fail at parsing the schema:

```yaml
# Error because quantity a length cannot be converted into temperature.
!number(quantity: temperature) !!number(unit: meter) 42
```

Again, UON does not allow to transform data. It only offers to represent data differently without content loss. It means, quantity of a number has to be preserved

# Binary Encoding

The binary encoding is one of the key feature of *UON* because it allows to reduce the payload to its minimal form.

The rules are:

- Keywords are ordered by names, not order
- Variable size payload are put after fixed size payloads

### !ipv4

In the IP version 4 schema (defined above), the binary form is defined as:

```yaml
binary: [!uint8 @[0], !uint8 @[1], !uint8 @[2], !uint8 @[3]]
```

| Offset in B | Value  | Type     |
|-------------|--------|----------|
| 0           | Byte 0 | `!uint8` |
| 1           | Byte 1 | `!uint8` |
| 2           | Byte 2 | `!uint8` |
| 3           | Byte 3 | `!uint8` |

### !jwt

| Offset | Value  | Type    |
|--------|--------|---------|
| 0      | xxx    | `!blob` |
| 1      | yyy    | `!blob` |
| 2      | zzz    | `!blob` |
| 3      | Byte 3 | `!blob` |

### !blob

```yaml
coerce: {
  !!binary: [
    !!uint(desc: "Blob Length") @@...length,
    !$iterate(start: 0, iteration: @...length: @@...content)
  ]
}
```

| Offset | Value  | Type    |
|--------|--------|---------|
| 0      | length | `!uint` |
| 1      | data   | `!blob` |

# Annexes

## Regulations and standards

*UON* must be as standard as possible against existing standards for expressing data such as IP address, timestamp or mathematical notation.

| UON Type    | Standards                                                | Example                                                                 |
|-------------|----------------------------------------------------------|-------------------------------------------------------------------------|
| `!uuid`     | [RFC 4122](https://tools.ietf.org/html/rfc4122)          | `uuid:f81d4fae-7dec-11d0-a765-00a0c91e6bf6`                             |
| `!jwt`      | [RFC 7797](https://tools.ietf.org/html/rfc7797)          | `xxxxx.yyyyy.zzzzz`                                                     |
| `!datetime` | [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601)       | `2018-06-10T16:25:17+00:00`                                             |
| `!date`     | [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601)       | `2018-06-10`                                                            |
| ``!time``   | [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601)       | `16:25:17`                                                              |
| `!regex`    | [PCRE](https://www.pcre.org/)                            | `/foo|[bBar]/`                                                          |
| `!unit`     | [ISO 80000-1](https://en.wikipedia.org/wiki/ISO_80000-1) | `celsius`                                                               |
| `!math`     | [MathJax](https://www.mathjax.org/)                      | `$x_{1,2}=-b\pm\frac{\sqrt{b^2-4\cdota\cdotc}{2\cdota}$`                |
| `!ipv4`     | [RFC 791](https://tools.ietf.org/html/rfc791)            | `192.168.1.42`                                                          |
| `!ipv6`     | [RFC 2732](https://www.ietf.org/rfc/rfc2732.txt)         | `1080::8:800:200C:417A`                                                 |
| `!urn`      | [RFC 8141](https://tools.ietf.org/html/rfc8141)          | `urn:example:foo-bar-baz-qux?+CCResolve:cc=ch`                          |
| `!uri`      | [RFC 2396](https://www.ietf.org/rfc/rfc2396.txt)         | `news:comp.infosystems.www.servers.unix`                                |
| `!req`      | [RFC  2119](https://www.ietf.org/rfc/rfc2119.txt)        | `!req(must) "Be portable and interoperable"`                            |
| `!log`      | [RFC 5424](https://tools.ietf.org/html/rfc5424)          | `!log(error) "Unable to bind socket"`                                   |
| `!str`      | [RFC 3629](https://tools.ietf.org/html/rfc3629)          | `"𝚄O𝙽™ ı𝘴 𝛼 𝒇𝛼𝐧𝜏𝓪𝕤𝕥⍳ⲥ 𝓈𝙚𝓻𝚤𝕒lℹ𝙯𝝰𝜏ｉο𝓃 l𝙖ϖℊʋ𝙖𝗀𝕖"` |
| `!float`    | [IEEE 754:2008](https://en.wikipedia.org/wiki/IEEE_754)  | `3.1415926535`                                                          |

## Available Types

This annex lists all the native *UON* types. Currently 80 types have been reserved. *UON* supports 256 different types as a type identifier is encoded using a `!uint8` type.

| ID   | Name          | Description                                          | Derived from |
|------|---------------|------------------------------------------------------|--------------|
| 0x00 | `!type`       | Base type                                            | -            |
| 0x01 | `!seq`        | Ordered sequence                                     | `!type`      |
| 0x02 | `!map`        | Unordered list of key-value pairs                    | `!type`      |
| 0x03 | `!set`        | Unordered set of unique data                         | `!seq`       |
| 0x04 | `!omap`       | Ordered list of key-value pairs                      | `!seq`       |
| 0x05 | `!oset`       | Ordered set of unique data                           | `!seq`       |
| 0x06 | `!scalar`     | Any scalar representable with a string               | `!type`      |
| 0x07 | `!uon`        | UON document                                         |              |
| 0x08 | `!oneof`      | Validation only                                      | `!set`       |
| 0x09 | `!and`        | Validation only                                      | `!seq`       |
| 0x0a | `!or`         | Validation only                                      | `!seq`       |
| 0x0b | `!add`        | Validation only `10 == !add [ 5, 5 ]`                | `!seq`       |
| 0x0c | `!sub`        | Validation only `0 == !sub [ 10, 10 ]`               | `!seq`       |
| 0x0d | `!mul`        | Validation only `10 == !mul [ 10, 2 ]`               | `!seq`       |
| 0x0e | `!div`        | Validation only `5 == !div [ 10, 2 ]`                | `!seq`       |
| 0x0f | `!prop`       | Validation only, read the referred property as value | `!type`      |
| 0x10 | `!null`       | Null value                                           | `!scalar`    |
| 0x11 | `!str`        | String                                               | `!scalar`    |
| 0x12 | `!keyword`    | Keyword `/[a-z][a-z0-9-]*(?=-)/`                     | `!str`       |
| 0x13 | `!rkeyword`   | Reserved keyword (not directly usable)               | `!keyword`   |
| 0x14 | `!bool`       | Boolean value `/false|true/`                         | `!key`       |
| 0x15 | `!num`        | Any real number `/[+-](0|[1-9]|[1-9\d*)(\.?\d*)?/`   | `!scalar`    |
| 0x16 | `!ref`        | Reference to another value                           | `!str`       |
| 0x17 | `!blob`       | Raw Binary Content                                   | `!scalar`    |
| 0x18 | -             | Reserved                                             |              |
| 0x19 | -             | Reserved                                             |              |
| 0x1a | -             | Reserved                                             |              |
| 0x1b | -             | Reserved                                             |              |
| 0x1c | -             | Reserved                                             |              |
| 0x1d | -             | Reserved                                             |              |
| 0x1e | -             | Reserved                                             |              |
| 0x1f | -             | Reserved                                             |              |
| 0x20 | `!real`       | Any kind of representable number (TODO: Keep it?)    | `!num`       |
| 0x21 | `!float`      | Binary Floating point number encoded using IEEE 754  | `!real`      |
| 0x22 | `!decimal`    | Decimal Floating point number encoded using IEEE 754 | `!real`      |
| 0x23 | `!float32`    | 32-bit binary float                                  | `!floatb`    |
| 0x24 | `!float64`    | 64-bit binary float                                  | `!floatb`    |
| 0x25 | `!float128`   | 128-bit binary float                                 | `!floatb`    |
| 0x26 | `!decimal32`  | 32-bit decimal float                                 | `!floatd`    |
| 0x27 | `!decimal64`  | 64-bit decimal float                                 | `!floatd`    |
| 0x28 | `!decimal128` | 128-bit decimal float                                | `!floatd`    |
| 0x29 | `!int`        | Variable length integer value $\in \mathbb{N} $      | `!real`      |
| 0x30 | `!uint`       | Variable length unsigned integer $\in \mathbb{Z} $   | `!int`       |
| 0x31 | `!int8`       | 8-bit signed integer                                 | `!int`       |
| 0x32 | `!int16`      | 16-bit signed integer                                | `!int`       |
| 0x33 | `!int32`      | 32-bit signed integer                                | `!int`       |
| 0x34 | `!int64`      | 64-bit signed integer                                | `!int`       |
| 0x35 | `!int128`     | 128-bit signed integer                               | `!int`       |
| 0x36 | `!int256`     | 256-bit signed integer                               | `!int`       |
| 0x37 | `!unt8`       | 8-bit unsigned integer                               | `!int`       |
| 0x38 | `!unt16`      | 16-bit unsigned integer                              | `!int`       |
| 0x39 | `!unt32`      | 32-bit unsigned integer                              | `!int`       |
| 0x3a | `!unt64`      | 64-bit unsigned integer                              | `!int`       |
| 0x3b | `!unt128`     | 128-bit unsigned integer                             | `!int`       |
| 0x3c | `!unt256`     | 256-bit unsigned integer                             | `!int`       |
| 0x3d | `!complex`    | Complex number $\in \mathbb{C} $                     | `!seq`       |
| 0x3e | `!quaternion` | Quaternion number $\in \mathbb{H} $                  | `!seq`       |
| 0x3f | `!frac`       | Rational number $\in \mathbb{Q} $                    | `!seq`       |
| 0x40 | `!version`    | Semantic version number x.y.z                        | `!str`       |
| 0x41 | `!regex`      | Regular expression, with default PCRE flavor         | `!str`       |
| 0x42 | `!datetime`   | Date time value according to ISO 8601                | `!str`       |
| 0x43 | `!date`       | Only date value                                      | `!datetime`  |
| 0x44 | `!time`       | Only time value                                      | `!datetime`  |
| 0x45 | `!ipv4`       | IP Version 4 e.g. `192.168.1.4/23`                   | `!uint32`    |
| 0x46 | `!ipv6`       | IP Version 6                                         | `!uint128`   |
| 0x47 | `!jwt`        | JSON Web Token                                       | `!str`       |
| 0x48 | `!uuid`       | 128-bit Universal Unique Identifier                  | `!uint128`   |
| 0x49 | `!epoch`      | UNIX timestamp                                       | `!uint32`    |
| 0x4a | `!math`       | Mathematical expression                              | `!str`       |
| 0x4b | `!uri`        | Uniform Resource Identifier                          | `!str`       |
| 0x4c | `!url`        | Uniform Resource Locator                             | `!uri`       |
| 0x4d | `!urn`        | Uniform Resource Name                                | `!uri`       |
| 0x4e | `!log`        | Log entry (severity, datetime, module)               | `!str`       |
| 0x4f | `!req`        | Requirement Specification (must, should, ...)        | `!str`       |

## Reserved Keywords

These are units, quantities and properties defined in the standard

| ID   | Keyword             |
|------|---------------------|
| 0x00 | `false`             |
| 0x01 | `true`              |
| 0x02 | `null`              |
| 0x03 | `comment`           |
| 0x04 | `description`       |
| 0x05 | `optional`          |
| 0x06 | `required`          |
| 0x07 | `min`               |
| 0x08 | `max`               |
| 0x09 | `lt`                |
| 0x0a | `le`                |
| 0x0b | `gt`                |
| 0x0c | `ge`                |
| 0x0d | `exclusive`         |
| 0x0e | `quantity`          |
| 0x0f | `dummy`             |
| 0x10 | `default`           |
| 0x11 | `merge`             |
| 0x12 | `inherit`           |
| 0x13 | `offset`            |
| 0x14 | `link`              |
| 0x15 | `alias`             |
| 0x16 | `hash`              |
| 0x17 | `signature`         |
| 0x18 | `ordered`           |
| 0x19 |                     |
| 0x1a |                     |
| 0x1b |                     |
| 0x1c |                     |
| 0x1d |                     |
| 0x1e |                     |
| 0x1f |                     |
| 0x20 | `meter`             |
| 0x21 | `kilogram`          |
| 0x22 | `second`            |
| 0x23 | `ampere`            |
| 0x24 | `kelvin`            |
| 0x25 | `mole`              |
| 0x26 | `candela`           |
| 0x27 | `hertz`             |
| 0x28 | `radian`            |
| 0x29 | `steradian`         |
| 0x30 | `newton`            |
| 0x31 | `pascal`            |
| 0x32 | `joule`             |
| 0x33 | `watt`              |
| 0x34 | `coulomb`           |
| 0x35 | `volt`              |
| 0x36 | `farad`             |
| 0x37 | `ohm`               |
| 0x38 | `siemens`           |
| 0x39 | `weber`             |
| 0x3a | `tesla`             |
| 0x3b | `henry`             |
| 0x3c | `celsius`           |
| 0x3d | `lumen`             |
| 0x3e | `lux`               |
| 0x3f | `becquerel`         |
| 0x40 | `gray`              |
| 0x41 | `sievert`           |
| 0x42 | `katal`             |
| 0x43 | `liter`             |
| 0x44 | `hour`              |
| 0x45 | `minute`            |
| 0x46 | `day`               |
| 0x47 | `month`             |
| 0x48 | `year`              |
| 0x49 | `bar`               |
| 0x50 | `mmhg`              |
| 0x51 | `electronvolt`      |
| 0x52 | `bit`               |
| 0x53 | `byte`              |
| 0x54 | `nat`               |
| 0x55 | `nit`               |
| 0x56 | `light-year`        |
| 0x57 | `parsec`            |
| 0x58 | `astronomical-unit` |
| 0x59 |                     |
| 0x5a |                     |
| 0x5b |                     |
| 0x5c |                     |
| 0x5d |                     |
| 0x5e |                     |
| 0x5f |                     |
| 0x60 | `aes128`            |
| 0x61 |                     |
| 0x62 |                     |
| 0x63 |                     |
| 0x64 | `base85`            |
| 0x65 | `base58`            |
| 0x66 | `base64`            |
| 0x67 | `base32`            |
| 0x68 | `base16`            |
| 0x69 |                     |
| 0x6a |                     |
| 0x6b |                     |
| 0x6c |                     |
| 0x6d |                     |
| 0x6e |                     |
| 0x6f |                     |
| 0x70 |                     |
| 0x71 |                     |
| 0x72 |                     |
| 0x73 |                     |
| 0x74 |                     |
| 0x75 |                     |
| 0x76 |                     |
| 0x77 |                     |

## Regex validation

Some of the rich types can be coerced from a string using a PCRE regex:
| Type        | Regex                            |
|-------------|----------------------------------|
| `!datetime` | https://regex101.com/r/1AhuPW/1/ |
| `!ipv4`     | https://regex101.com/r/y8jPbQ/1  |
|             |                                  |


## Type names vs other languages

Consensus matters, *UON* aims to use the largest consensus, by considering several aspects:

* How many languages agree on type names?
* Which language has better credit for a chosen type (high level, low-level language)?
  * C/C++ within C99/C11 is the most used language at bare-metal level. The type name is coherent.
* Tradition over time
  * Tradition for numbers is to use the system names such as `real`, `natural`, `rational`
* Simplicity: shorter is better, simple is better
  * It is largely agreed that `str`means string, but with 3 chars shorter, despite most languages use the full name.

The following table shows the type names for various programming languages and the retained name in *UON*:

| UON          | Math     | C/C++      | Java         | Python         | Go          | C#        | PHP       |
|--------------|----------|------------|--------------|----------------|-------------|-----------|-----------|
| `!int8`      | -        | `int8_t`   | `byte`       | `c_byte`       | `byte`      | `byte`    | -         |
| `!uint8`     | -        | `uint8_t`  | -            | `c_ubyte`      | `rune`      | `sbyte`   | -         |
| `!int16`     | -        | `int16_t`  | `short`      | `c_short`      | `int16`     | `short`   | -         |
| `!uint16`    | -        | `uint16_t` | `char`       | `c_ushort`     | `uint16`    | `ushort`  | -         |
| `!int32`     | -        | `int32_t`  | `int`        | `c_int`        | `int32`     | `int`     | `int`     |
| `!uint32`    | -        | `uint32_t` | `uint`       | `c_uint`       | `uint32`    | `uint`    | -         |
| `!int64`     | -        | `int64_t`  | `long`       | `c_longlong`   | `int64`     | `long`    | `int`     |
| `!uint64`    | -        | `uint64_t` | `ulong`      | `c_ulonglong`  | `uint64`    | `ulong`   | -         |
| `!int128`    | -        | -          | -            | -              | -           | -         | -         |
| `!uint128`   | -        | -          | -            | -              | -           | -         | -         |
| `!int`       | integer  | -          | `BigInteger` | `int`          | `int`       | -         | `int`     |
| `!uint`      | natural  | `size_t`   | `BigInteger` | `int`          | `uint`      | -         | -         |
| `!number`    | -        | -          | -            | `decimal`      | -           | -         | -         |
| `!frac`      | rational | -          | -            | -              | -           | -         | -         |
| `!bool`      | -        | `bool`     | `boolean`    | `bool`         | `bool`      | `bool`    | `boolean` |
| `!str`       | -        | `char *`   | `String`     | `str`          | `string`    | `string`  | `string`  |
| `!float`     | real     | -          | `BigDecimal` | `float`        | -           | -         |           |
| `!bfloat32`  | -        | `float`    | `float`      | `c_float`      | `float32`   | `float`   |           |
| `!bfloat64`  | -        | `double`   | `double`     | `c_double`     | `float64`   | `double`  |           |
| `!bfloat128` | -        | -          | -            | `c_longdouble` | -           | -         |           |
| `!dfloatl32` | -        | -          | -            | -              | -           | -         |           |
| `!dfloat64`  | -        | -          | -            | -              | -           | -         |           |
| `!dfloat128` | -        | -          | -            | -              | -           | `decimal` |           |
| `!complex`   | complex  | -          | -            | -              | `complex64` | -         |           |

We note that only *UON* can express true rational numbers or true real numbers.

## Examples

### Git repository

Bob would like to represent a simplified [Git](https://en.wikipedia.org/wiki/Git) repository with UON. It first write an information model such as this one:

![](assets\git-information-model.png)

Now, this information model can be translated into a UON schema:

```yaml
!uon(version: 0.0.1) {
  !git-repository: {
    head: !ref(base: "refs"),
    refs: {
      !key: !str
    },
    authors: {
      !str(desc: "email"): !!str
    },
    objects: {
      !key: !!oneof(
        !commit,
        !blob,
        !tree
      )
    }
  },
  !!ref: !ref(base: "objects"),
  !!filename: !str,
  !!blob: !oneof { !blob, !str },
  !!commit: !map(extensible: false) {
    tree: !ref,
    parent(optional: true): !ref,
    author: !map(extensible: false) {
      identity: !ref(base: "authors"),
      date: !datetime
    }
  },
  !!tree: !!schema {
    !!filename: {
      permission: !oneof{ read, write, readwrite },
      ref: !!ref
    }
  }
}
```

A valid payload would be:

```yaml
!!git-repository {
  head: @(master)
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
        permissions: readwrite,
        ref: @(e69de29bb2d1d6434b8b29ae775ad8c2e48c5391)
      },
      "foo": {
        permissions: read,
        ref: @(d9e80f6f7602c18d035af9659303047b30204182)
      }
    },
    e69de29bb2d1d6434b8b29ae775ad8c2e48c5391: !blob "I am bar",
    d9e80f6f7602c18d035af9659303047b30204182: !blob "I am foo"
  }
}
```

This payload can be encoded in binary with minimal overhead thanks to the validation schema. The unresolved references are great because they allow navigation for example with Python:

```python
>>> from uon import Parser
... parser = Parser(schema: "http://example.com/git-repository")
... repo = parser.load('repository.uon')
... print(repo.head.tree['foo'].ref)
'I am bar'
```

### Temperature Sensor

Well, this is perhaps not a very good example, but we can show that UON is able to convert value to other units.

```python
>>> from uon import Parser
... parser = Parser("!real(unit:km)")
... def ly2km(distance):
...     return float(parser.loads("!real(unit:ly) %f" % x).dumps())
...
... ly2km(1)
9460000000000
```

The parser is initialized with the schema `!real(unit:km)` , but we load the value in light year. As both values are the same quantity: `distance`, UON is able to convert from one value to the other
