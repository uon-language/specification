# UON Unified Object Notation (UON ™ )

## Draft Version 0.0.1

**This version:** http://github.com/uon-language/specification/blob/documentation.md

**Bug tracker**: http://github.com/uon-language/specification/issues

**Editor**: Yves Chevallier <ycr@x0x.ch>

Copyright© 2018 Yves Chevallier

# Introduction

## Why UON™?

The [IoT](https://en.wikipedia.org/wiki/Internet_of_things) world is nowadays dominated by [XML](https://www.w3.org/TR/xml/), [JSON](https://www.json.org/) and [YAML](http://yaml.org/). Other solutions such as [Protocol Buffers](https://en.wikipedia.org/wiki/Protocol_Buffers) attempt to offer a more interoperable solution programming language agnostic. Unfortunately none of them are complete by targeting either low-power MCU with limited computation sources, web services and high-end applications. UON™ would like to federate all the IoT world with regard to the [Industry 4.0](https://en.wikipedia.org/wiki/Industry_4.0) needs.

## What is UON™ and when should it be used?

*UON™* is a serialization language which is a superset of [JSON](https://www.json.org/) and a partial superset of [YAML](http://yaml.org/). It provides additional features useful for interoperability between different kind of devices. The major additional features are: 

* Strong typing
* Validation schema
* Binary payload
* Canonical form
* Physical quantities and units
* References 
* Standard API

A typical use is the communication between a low-power device and another device. Both have to agree on the nature of the payload content, here a temperature sensor. The validation schema acts as a contract between both nodes. It allows one to encode the payload with minimal binary size and the other to decode this payload. 

Alternatively, a validation schema can be refined to give additional information about the payload format. In this example, a temperature sensor is a device that publish a temperature quantity. The refined schema gives precision about the format of this quantity: unit, binary format, uncertainty, ...

![](C:\Users\ycr\Data\specification\assets\data-exchange.png)

## How far is it from JSON and YAML?

As previously said, *UON™* is a superset of JSON, but it offers much more. To quantify the distance between *UON™* and JSON 4 we have 4 different level of complexity: 

* `UON:0` is fully JSON compliant with regard to the [RFC8259](https://tools.ietf.org/html/rfc8259) 
* `UON:1` is partially compliant with YAML.
* `UON:2` provides type properties, coercion and multiline strings
* `UON:3` offers rich types and references 

![](C:\Users\ycr\Data\specification\assets\complexity-levels.png)



![](C:\Users\ycr\Data\specification\assets\supersets.png)

### Why not JSON?

JSON is great because it is a subset of JavaScript. However it misses some important features: 

* Lack of native schema validation (partially offered with [JSON Schema](http://json-schema.org/))
* Lack of types
* Lack of sets and ordered mappings
* Lack of references
* Lack of binary format

The JSON file `{"secured": true}` is semantically incorrect. The `secured` name is rather a keyword identifying a configuration setting than string representing an arbitrary text. Every textual value in JSON is  quoted without any possibility to assign a more precise meaning to it. 

### Why not YAML?

YAML is undoubtedly a fantastic serialization language. It is very readable and the possibility to add tags to values solves some limitations of JSON. Notice that YAML is also a superset of JSON. 

Unfortunately, YAML has some weaknesses: 

* It is unable to parse unknown tags
* It lack validation
* References are always resolved
* Its writing could be error prone.

For example the following YAML content can lead to errors because it is interpreted as `['foo bar', 'baz']` due to the missing comma.

```yaml
[
  foo
  bar,
  baz
]
```

# Information Model

The *UON™* information model is shown by this figure below. Each item that composes *UON* object tree is  a Type that may have several properties. The content can be either a scalar or a collection of items. 

Three kind of properties exists: 

* Presentation properties
* Validation properties
* Application properties



![](C:\Users\ycr\Data\specification\assets\information-model.png)

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
!!data: !map(extensible: false) {
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
| `( )`        | Properties                     | -        |

Here is a example that uses all the symbol pairs:

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
| `:=`    | Type assignation                                             |
| `,`     | Element separator                                            |
| `!`     | Standard type (cannot be defined at the user level)          |
| `!!`    | User type used to extend *UON* capabilities.                 |
| `@`     | Reference                                                    |
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

```yaml
{
  null: null,
  bool: true
}
```

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

```yaml
{
  rational: 643 / 123
  complex: 12 + 6i,
  quaternion: 6 + 3i + 9j - 8k,    
}
```

### Arbitrary size

```yaml
{
  int: -564 645 543 123 241e143
  uint: +483813344e-122 
  float: -1345.392828482943833e-44
  decimal: -1 345.392 828 482 943 833e-44
}
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

![](C:\Users\ycr\Data\specification\assets\git-information-model.png)

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

