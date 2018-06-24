# UON Unified Object Notation (UON ™ )

## Draft Version 0.0.1

**This version:** http://github.com/uon-language/specification/blob/documentation.md

**Bug tracker**: http://github.com/uon-language/specification/issues

**Editor**: Yves Chevallier <ycr@x0x.ch>

Copyright© 2018 Yves Chevallier

# Introduction

## Why UON™?

The [IoT](https://en.wikipedia.org/wiki/Internet_of_things) world is nowadays dominated by [XML](https://www.w3.org/TR/xml/), [JSON](https://www.json.org/) and [YAML](http://yaml.org/). Other serialization protocols such as [Protocol Buffers](https://en.wikipedia.org/wiki/Protocol_Buffers) aim to a better interoperable solution by offering a programming language agnostic solution. Unfortunately none of them are complete. They focus only on narrow areas instead of addressing a more general solution that encompass low-power MCU with limited computation sources, web services and high-end applications. 

UON™ would like to federate the IoT world by taking in account the [Industry 4.0](https://en.wikipedia.org/wiki/Industry_4.0) needs. In other word, the Unified Object Notation is meant to describe messages sent and received from:

* Battery powered telemetry sensors
* Web APIs
* IPC 

To achieve this goal, UON defines a way to:

* Encode/Decode payloads either written in binary or in human readable form 
* Describe a payload through a validation schema

## What is UON™ and when should it be used?

*UON™* is basically a serialization language which is a superset of [JSON](https://www.json.org/) and a partial superset of [YAML](http://yaml.org/). It provides additional features useful for increasing interoperability between different kind of devices. Its main feature advantages compared to other similar languages are: 

* Strong typing
* Validation schema
* Binary payload
* Canonical form
* Physical quantities and units
* References 
* Standard API
* Type properties
* User types

A typical use is the communication between a low-power device and another device. Both have to agree on the nature of the payload content, here a temperature sensor. The validation schema acts as a contract between both nodes. It allows one to encode the payload with minimal binary size and the other to decode this payload. 

Alternatively, a validation schema can be refined to give additional information about the payload format. In this example, a temperature sensor is a device that publish a temperature quantity. The refined schema gives precision about the format of this quantity: unit, binary format, uncertainty, ...

![](assets\data-exchange.png)

### Strong Typing

In JSON for instance, they are only 6 different data types that can be used: `string`, `number`, `object`, `array`, `boolean` and `null`. Objects are either lists or mappings and numbers can be either integers or floating point numbers. It is therefore not possible to have more complex types neither refining an integer by specifying its length.

The example below shows a mapping with two key-value pairs, one with an explicit type (unsigned integers 8-bit) and the other with the implicit `!ipv4` type.

```yaml
{
    temperature: !uint8 23,
    ip: 192.168.1.6,
}
```

UON in its standard form defines 80 types allowing to encode UON data to an optimal binary payload.

### Validation Schema

Similarly to JSON-Schema, UON provides a solution to describe and validate a dataset. Schemas can be either  embedded in a payload or take place in a separated document. The example below shows a schema for the above payload example.

```yaml
{
    temperature: !uint8,
    ip: !ipv4,
    scalar: !uint
}
```

Schemas can also be refined. Refined schemas provide details on how a data is binary encoded. It can also constrain a value more than in the original schema.

### Binary Payload

UON aims to minimize the binary form of a dataset with well defined schemas. The above temperature example describes a payload with a two key-value pairs. With no associated schema, the binary form must includes the name of the keys and describe each type that is used. We show each data-block in corner brackets

```
｢!map｣｢length=2｣｢!key｣｢temperature｣｢!uint8｣｢23｣｢!key｣｢ip｣｢!ipv4｣｢192.168.1.6｣
```

After converting everything in binary we eventually get:

```
｢0x02｣｢0x02｣｢0x12｣｢temperature\x00｣｢0x37｣｢0x17｣｢0x12｣｢ip\x00｣｢0x45｣｢0xc0a80106｣
```

By associating a schema to the payload, we give more knowledge of the payload content. Therefore the name of the keys can be avoided as well as the data types. Therefore we get a minimal payload such as this one.

```
｢23｣｢192.168.1.6｣
```

### Canonical Form

Those who worked with YAML know that the same dataset can be expressed in different manner: 


```yaml
%YAML 1.2
--- # A way
sequence:
- one
mapping:
  sky: blue
  sea: green
--- # Another way
!!map {
  ? !!str "sequence"
  : !!seq [ !!str "one" ],    
  ? !!str "mapping"
  : !!map {
    ? !!str "sky" : !!str "blue",
    ? !!str "sea" : !!str "green",
  },  
}
--- # Another one
{ "sequence": [ one ], "mapping": { "sky": "blue", "sea": "green"}}
```

These forms could be quite confusing and UON offers a canonical form of how a dataset must be represented. The above example is therefore represented as follow:

```yaml
{ 
  sequence: [
    "one"
  ],
  mapping: {
  	sky: "blue",
  	sea: "green"
  }
}
```

UON provide 3 different canonical forms: 

* Descriptive
* Minimal
* Binary

We already saw the binary form earlier then the descriptive as right above. The minimal form removes all the unnecessary characters:

```yaml
{sequence:["one"],mapping:{sky:"blue",sea:"green"}}
```

###  Physical Quantities and Units

When exchanging telemetry data, the knowledge of the used unit is important. Therefore UON offers a support for physical quantities:

```yaml
{
    temperature: !uint8(unit: celsius) 23
}
```

The parenthesis you see allow to assign properties to a type. Here we associate a unit to the `!uint8` type.

### References

Similarly to YAML, UON allow references. The example below uses `@()` to indicate a relative reference to `Bob Williams`.

```yaml
{
    authors: {
        bob: "Bob Williams",
        alice: "Alice Smith"    
    }
    books: {
        {
           author: @(../../authors/bob),
           title: "How to not be suspicious about Eve?"
        },
    }
}
```

UON supports to kind of references: resolved and unresolved. 

Resolved references written `@(...)` are eliminated when the payload is built. Unresolved references written `@@(...)` remain in the payload. Choosing the most adequate reference can help to reduce the payload size.  

### Standard API

The main idea behind UON is to be usable with different programming language using the same API. For example each implementation should provide methods such as `load`, `loads`, `dump` and `dumps`. 

### Type properties

This is where UON becomes more complex than other serialization languages. It offers properties to type allowing to constrain, document or refine a value. Properties are associated to a type, not a value. 

The example below shows our temperature payload with constraints (minimum and maximum possible value) , the associated unit and the uncertainty of this value. 

```yaml
{
    temperature: !uint8(unit: celsius, min: 7, max: 92, uncertainty: 1) 23
}
```

Properties are separated in three kinds:

* Presentation Properties
* Validation Properties
* Internal Properties

### User types

UON allow to define user types. The following UON document define a new type `!!person` that can be used later in a payload: 

```yaml
!!person:= {
  "name": !str,
  "email": !str(pattern: /[^@]+@[^@]+\.[^@]+/)
}
```

```yaml
!!person {
  name: "John Doe",
  email: "jdoe@example.com"
}
```

User types are therefore used with schemas.

## How far is it from JSON and YAML?

As previously said, *UON™* is a superset of JSON, but it offers much more. To quantify the distance between *UON™* and JSON, we let's have a look at the 4 different level of complexity: 

* `UON:0` is fully JSON compliant with regard to the [RFC8259](https://tools.ietf.org/html/rfc8259) 
* `UON:1` is partially compliant with YAML.
* `UON:2` provides type properties, coercion and multiline strings
* `UON:3` offers rich types and references 

![](assets\complexity-levels.png)

On the figure below, we can see that some features of YAML 1.2 are not included in UON.

![](assets\supersets.png)

### Why not JSON?

JSON is great because it is a subset of JavaScript. However it misses some important features: 

* Lack of native schema validation (partially covered with [JSON Schema](http://json-schema.org/))
* Lack of extended types
* Lack of sets and ordered mappings
* Lack of references
* Lack of binary format

The JSON file below is semantically incorrect

```json
{
  "secured": true, 
  "admin": ["bob", "alice"]
}
```

First, the `secured` name is rather a keyword identifying a configuration setting than string representing an arbitrary text. Every textual value in JSON is quoted without any possibility to assign a more precise meaning to it. Secondly, the administrator list should not be a list. A list takes account of order and allow duplicate values. We should rather have a set.

```yaml
{
  secured: true, 
  admin: {
    "bob", 
    "alice"
  }
}
```

### Why not YAML?

YAML is undoubtedly a fantastic serialization language. It is very readable and the possibility to associate tags to values solves some limitations of JSON. Notice that YAML is also a superset of JSON. 

Unfortunately, YAML has some weaknesses: 

* It is unable to parse unknown tags
* It lack validation
* References are always resolved
* Its writing could be error prone.
* It is not safe by default

For example the following YAML content can lead to errors because it is interpreted as `['foo bar', 'baz']` due to the missing comma.

```yaml
[
  foo # Oh a missing comma here
  bar,
  baz
]
```

# Information Model

The *UON™* information model is shown by this figure below. Each item that composes *UON* object tree is  a `Type` that may have several properties. The content can be either a `Scalar` or a `Collection` of items. 

Three kind of properties exists: 

* Presentation properties
* Validation properties
* Application properties



![](assets\information-model.png)

## Types

Types are instances

## Properties

### Application 

These properties are not used or defined in a UON document, they are either static or derived properties that 

```yaml
!int16: !number(
  type: 0x14,
  system: "real"  
)
```

### Validation 

These properties constraint the node content They can can be a minimum or maximum value for a number, or a regex pattern to be respected for a string value. These properties are usually not included in a dataset because they do not condition a value. 

The example below is a validation schema that uses validation properties to constraint the value

```yaml
!!data:: !map(extensible: false) {
  year(optional: true): !uint(unit: year, min: 1990, max: 2050),
  data: !seq(min: 4, max: 8) [ !uint8 ]
  access: !oneof { read(id: 0), write(id: 1), readwrite(id: 2) }
}
```

A dataset would be this:

```yaml
!!data {
  year: 2018,
  data: [ 4, 8, 15, 16, 23, 42 ],
  access: read
}
```

### Presentation 

Any property that influence the interpretation of a node content is called presentation property. The uncertainty of a number, its unit, the encoding type for a binary content are presentation properties. 

In the example below, the properties `unit`, `uncertainty` or `offset` do not alter the content, but acts on how it is represented. In the example below the two writing `a` and `b` are equivalent. 

```yaml
{
  a: !number(unit: celsius, uncertainty: 0.01, offset: +10.0) 23.15
  b: 43.15±0.01 °C
}
```

Alternatively the three entries below are equivalent:

```yaml
{
  a: !blob(encoding: base64) "BAgPEBcq",
  b: !blob(encoding: base32) "AQEA6EAXFI======",
  c: !type(schema: !blob !seq(binary: true, length: 6) [ !uint8 ]) [4, 8, 15, 16, 23, 42]
}
```

# Syntax

## Symbol Pairs

Symbol pairs are syntactic sugars used to enhance readability by associating a specific symbol pair to a particular type of data.

| Symbol Pairs | Description                    | Type     |
| ------------ | ------------------------------ | -------- |
| `[ ]`        | Ordered Sequence               | `!seq`   |
| `{ }`        | Mapping (by default unordered) | `!map`   |
| `" "`        | String                         | `!str`   |
| `/ /`        | Regular expression             | `!regex` |
| `$ $`        | Mathematical expression        | `!math`  |
| `( )`        | Properties                     | `!map`   |

Here is an example that uses all the supported symbol pairs:

```yaml
{ 
  data: !seq(sort: asc) [3, 1, 2],
  name: "John Doe",
  pattern: /\b[a-z]\d{5}(?=0{3..4})-\w+/i,
  equation: $\frac{1}{x}$
}
```

## Reserved Symbols

The only reserved language symbols are the following

| Symbol  | Description                                                  |
| ------- | ------------------------------------------------------------ |
| `:`     | Key value separator                                          |
| `::`    | Type assignation, only valid with a non existing user type as a key |
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

# Overview

## Collections

```yaml
# Available collections
{
  # Unordered Mapping
  map: {
    firstname: "Alice",
    lastname: "Doe",
    email: "alice@doe.com"
  },
  
  # Ordered Mapping
  omap: !omap {
    4: "apple",
    8: "pineapple"
    15: "watermelon"
  },
  
  # Ordered Sequence
  seq: [4, 8, 15, 16, 23, 42],
  
  # Unordered Set
  set: {false, true, null},
  
  # Ordered Set
  oset: !oset {4, 8, 15, 16, 23, 42}
}
```

## Null and Booleans

The three keywords `false`, `true` and `null` have a special meaning in UON. The `null` value indicate the absence of value. It is optional though:

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

Null only exists for compatibility with JSON. 

## Numbers

### Quantities and uncertainties

```yaml
{
  quantities: {
    length: 12 m,
    duration: 24 h,
    acceleration: 35.12e3 m/s^2
  uncertainty: [
    12±0.01,
    -122.075e6±0.01 Ω,
    23.22±0.125 °C,
    12 + 6i W
  ]
}
```

### Rational, Complex and Quaternion

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

### Arbitrary size

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

### Fixed size

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

#### Rich types

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

# Resolving

*UON™* data is resolved differently depending on what we want: 

# Binary form

### Keywords encoding

In UON 256 reserved keywords are already assigned to a number. There keywords are mainly unit and quantity names. It means, when encoded in binary, the value only consumes 8 bits. 

Other keywords defined in a schema are naturally sorted and the index of each keyword becomes their value in LEB128. 

In the example below the keywords are identified as follow:

```yaml
{
  alpha: {
    "John": "Doe"
  }
  beta: {
    gamma: {
      celsius: null,
      mega: null,
      delta: false
    }
  }
}
```

```yaml
{
  keywords: {
    0x00: alpha,
    0x01: beta,
    0x02: delta,
    0x03: gamma
  }
}
```

The binary encoding will use these identifier to refer to their name:

```
｢!keyword｣ ｢0x00｣ # false
｢!rkeyword｣ ｢0x00｣ # alpha
```

the `!rkeyword` is a special type that cannot be used directly. 

# Examples

## Encryption

Two parties would like to exchange launch codes for interplanetary missiles. The payload content is made public:

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

## Temperature Sensor

## Git repository

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

## Unit conversion

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