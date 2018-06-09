# UON Unified Object Notation (UON ™ )

## Draft Version 0.0.1

http://github.com/uon-language/specification/spec.md

Yves Chevallier <ycr@x0x.ch>

Copyright(C) 2018 Yves Chevallier

## Introduction

In 2018 three major serialization format are commonly used across Internet: `XML`, `JSON` and `YAML`. Both are great, simple and well specified, but many would argue that: 

* `XML` is great but cumbersome 
* `JSON` is awesome but lack validation by design
* `YAML` is very readable, but isn't great for m2m communication

But seen from another perspective

* `xml` is fully specified with validation schema `XSD` and representation `XSLT`
* `JSON` is lightweight and understood by most programming languages
* `YAML` is very readable

*UON™* aims to steal all the amazing features of these serialization format into a single format that encompass them

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

 *UON*™ is not a programming language, but a data representation language. 

### Symbol Pairs

Symbol pairs are group of data that belong to a particular type. It helps visualizing the data

| Symbol Pairs | Description           | Type       |
| ------------ | --------------------- | ---------- |
| `[ ]`        | Ordered Sequence      | `!seq`     |
| `{ }`        | Mapping               | `!map`     |
| `" "`        | String                | `!str`     |
| `( )`        | Properties            | -          |
| `< >`        | Structural container  | -          |
| `# #`        | Comment               | `!comment` |
| `/ /`        | Regular expression    | `!regex`   |
| `$ $`        | Mathematics (MathJax) | `!math`    |



```yaml
!uon(version: 0.0.1) 
!map 
<
  !keyword "brand": !str "Toyota",
  !keyword "model": !str "Prius",
  !keyword "year": !int "2016",
>
```

### Symbols

The only reserved language symbols are the following

| Symbol  | Description                                                  |
| ------- | ------------------------------------------------------------ |
| `!`     | Standard type (cannot be (re)defined at the user level)      |
| `!!`    | User type                                                    |
| `$`     | Reserved keyword (may be removed on a future *UON* release)  |
| `,`     | Element separator                                            |
| `@`     | Reference                                                    |
| `.inf`  | Infinity (used within `!number`, `!float`, or `!decimal`)    |
| `.nan`  | Not a Number (used within `!number`, `!float`, or `!decimal`) |
| `true`  | Boolean value `true`                                         |
| `false` | Boolean value `false`                                        |
| `null`  | Null value                                                   |

## Example

### General overview

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
  } 
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
  }
  # Properties
  properties: {
      people(id: 1243): "Robert Ford" 
  }
  # References
  reference: {
  	ref: @scalar.number.decimal,
  	merge: !map(merge: @scalar.number) {
       int: 1409, # Override existing value
  	}
  # Rich types
  rich: {
     uuid: !!uuid "82584ce5-d086-41ff-978f-57323ebf5b9d",
     jwt: !!jwt "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpYXQiOjF9.9GeY7G6YbEh90kFhyhqsL29uFQCbIlk2Y-YVhae61g0",
     base64: !!base64 "W55IGNhcm5hbCBwbGVhc3VyZS4="
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
    genre: !any("Male", "Female"),
    link: !any(!!url, !!urn, !!uri)    
  }
  
  # Schema Type
  !str: !schema("http://uon.io/str") !type(
    description: """
       Base string encoded in UTF-8
    """t
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

# Type description for a !!car, and validation schema
!uon(version: 0.0.1) !schema("urn:car") {
  brand: !str(pattern: /[A-Z]\w+/),
  model: !str(pattern: /[A-Z]\w+/),
  year(required: false): !dec(min: 1930)
  color(required: false): !any(
     !str,
     !number
  )
}

# Coersion is implicit, the parsed result is `12.`
!uon !float !str 12
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
!map(comment: "Comment global to the document")
{  
  !keyword(comment: "Comment associated to the key-pair { brand: "Toyota" }") brand: "Toyota",
}
```



## Format

### Minimal

In minimal format:

* Spaced and new lines are removed
* Comments are enclosed in a comment block

```yaml
!uon(version:0.0.1)!(urn:car){brand:"Toyota",model:"Prius",year:2016}
```

### Binary

* Payload is encoded in a strict binary form
* LED128 format for numbers

```yaml
<tbd>
```

### Structured

* Document is indented using a two space indentation
* Explicit types are converted into implicit ones

```yaml
!uon(version: 0.0.1) !("urn:car") {
  brand: "Toyota"
  model: "Prius"
  year: 2016
}
```

## Datatypes

One important feature of *UON* is that any kind of information belongs to a type. Types are organized in different categories: 

### Data Structures (or Structural)

They are primitive data types all based on the `!type` master datatype. 

| Type      | Based on | Description                                            |
| --------- | -------- | ------------------------------------------------------ |
| `!type`   | -        | Base type                                              |
| `!map`    | `!type`  | Unordered Mapping (also called HashMap or Dictionary ) |
| `!omap`   | `!type`  | Ordered Mapping                                        |
| `!seq`    | `!type`  | Ordered Sequence (also called List or Array)           |
| `!scalar` | `!type`  | Scalar value                                           |
| `!set`    | `!map`   | Unordered set of values                                |
| `!oset`   | `!omap`  | Ordered set of values                                  |

*UON* aims to represent static data which is therefore immutable by design. This explain the absence of Records or Tuples. However, when a composite type is used as a key, it will be parsed as an immutable type if the destination language allows it.  

Unordered mapping (`!map`, `!set`) MUST always be sorted naturally when serialized e.g. `[1, 2, 10, 100, a]`

Sets are mapping with null values 

### Scalar

| Type      | Based on  | Description                                                  |
| --------- | --------- | ------------------------------------------------------------ |
| `!null`   | `!scalar` | Null type, can be associated to other type e.g. `!null(!str)` |
| `!bool`   | `!scalar` | Boolean, `true` or `false`                                   |
| `!char`   | `!scalar` | Character, one single symbol                                 |
| `!str`    | `!scalar` | String encoded in `UTF-8`                                    |
| `!number` | `!scalar` | Any kind of representable numeric value                      |
| `!key`    | `!str`    | Keyword used for mapping keys and references (`/^[a-z](?:(?:[a-z0-9-])*[a-z0-9])?$/i`) |
| `!schema` | `!type`   | *UON* Validation schema                                      |
| `!ref`    | `!str`    | Reference to another type                                    |

#### Numbers

| Type          | Based on  | Description                                                  |
| ------------- | --------- | ------------------------------------------------------------ |
| `!dec`        | `!number` | Decimal (i.e. Base 10) number e.g. `12345`                   |
| `!float`      | `!number` | Floating point `IEEE-754`                                    |
| `!oct`        | `!dec`    | Octal representation of decimal number                       |
| `!hex`        | `!dec`    | Hexadecimal representation of decimal number                 |
| `!complex`    | `!float`  | Complex value e.g. `42+7j`                                   |
| `!quaternion` | `!float`  | Quaternion value e.g. `1+2i+3k+4l`                           |
| `!magnitude`  | `!number` | Physical value with an associated unit                       |
| `!frac`       | `!seq`    | Fraction of two integers, to represent repeating such as `0.33333` |

### Sized

When serialized in binary format, the size of the payload has to be provided. If not, the serializing will use a 7-bit encoding where the 8th bit is used as a continuation bit formerly named [LEB128](https://en.wikipedia.org/wiki/LEB128). Variable size numeric values are less efficient at serializing and deserializing.  

| Type         | Based on | Description                                              |
| ------------ | -------- | -------------------------------------------------------- |
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

### Rich

Rich datatypes are always based on the `!str` but they can be parsed using a regex pattern. Each rich type can be coerced into different format.

All these formats have a strict binary encoding format when the data is transmitted in binary.

| Type        | Based on | Description                                                  | Coercible in                                      |
| ----------- | -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `!version`  | `!str`   | [SemVer](https://semver.org/) 2.0.0 version number           | `!seq`, `!map` |
| `!regex`    | `!str`   | PCRE2 Regular expression                                     |                                     |
| `!time`     | `!str`   | Time value e.g. `12:43:01` (`ISO 8601`)                      | `!seq`, `!map`       |
| `!date`     | `!str`   | Date value e.g. `1982-02-16` ( `ISO 8601`)                   | `!seq`, `!map` |
| `!datetime` | `!str`   | Full date time e.g. ` 2018-06-08T23:02:51+00:00 ` ( `ISO 8601`) | `!seq`, `!map`, `!epoch` |
| `!ipv4` | `!str`   | IP version 4 described in [RFC 791](https://tools.ietf.org/html/rfc791) | `!seq`, `!uint32` |
| `!ipv6` | `!str`   | IP version 4 described in [RFC 8200](https://tools.ietf.org/html/rfc8200) | `!seq`, `!uint128` |
| `!jwt` | `!str` | JSON Web Token | `!seq`, `!map` |
| `!uuid` | `!str` | Unique User Identifier, a 128-bit data |  |
| `!epoch` | `!dec` | UNIX timestamp | `!datetime` |

### Others

| Type       | Based on | Description                                      |
| ---------- | -------- | ------------------------------------------------ |
| `!any`     | `!map`   | Alternative possibility e.g. `!or( !str, !null)` |
| `!comment` | `!str`   | Comment, a dummy value, never transmitted        |

## Type details

### References

A reference can be:

* Absolute to the root of the document
* Relative to the current element
* Remote reference as URI: 
  * URL
  * URN
* Accessible remote references MUST point on UON valid files, otherwise the document is not valid
* URN references relies on an external UON document that resolve a urn. The resolve document is usually passed as follow `!uon(urn-resolve: @(http://some-domain.com))`. A set can be passed to `urn-resolve`

A possible value for a this resolver is: 

```yaml
# Resolver for URN
!uon(version: 0.0.1) {
   ietf: {
     rfc: 
       2648: !url "https://www.ietf.org/rfc/rfc2648.txt"
   }
}
```

### IPv4

The IPv4 type is defined in UON

```yaml
!ipv4: !schema("http://uon.io/ipv4") !str(
  description: "Type representing a Internet Protocol version 4 address defined in RFC 760",
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



## Units

An important information for telemetry systems and Internet of Things is the unit of magnitudes. When transmitting a temperature, it is essential to know which is the unit used to represent that number. *UON* uses the ISQ system ([ISO/IEC 80000-1:2009](https://www.iso.org/standard/30669.html)).

### Base Units

| Unit symbol | Unit name | Quantity name             |
| ----------- | --------- | ------------------------- |
| `m`         | Meter     | Length                    |
| `kg`        | Kilogram  | Mass                      |
| `s`         | Second    | Time                      |
| `A`         | Ampere    | Electric current          |
| `K`         | Kelvin    | Thermodynamic temperature |
| `mol`       | Mole      | Amount of substance       |
| `cd`        | Candela   | Luminous intensity        |

When only the quantity is known, the unit will be assumed to be SI.

### Derived Units

The International System of Units assigns special names to 22 derived units, which includes two dimensionless derived units, the `radian` and the `steradian`.

| Unit Symbol | Unit Name      | Quantity name                        | Equivalents                         | Base     |
| ----------- | -------------- | ------------------------------------ | ----------------------------------- | -------- |
| `Hz`        | Hertz          | `frequency`                          | $1/\text{s}$                        | $s^{-1}$ |
| `rad`       | Radian         | `angle`                              | $\text{m}/\text{m}$                 | $1$      |
| `sr`        | Steradian      | `solid-angle`                        | $\text{m}^2/\text{m}^2$             | $1$      |
| `N`         | Newton         | `[force, weight]`                    | $\text{kg}\cdot\text{m}/\text{s}^2$ |          |
| `Pa`        | Pascal         | `[pressure, stress]`                 |                                     |          |
| `J`         | Joule          | `[energy, work, heat]`               |                                     |          |
| `W`         | Watt           | `[power, radian-flux]`               |                                     |          |
| `C`         | Coulomb        | `electric-charge`                    |                                     |          |
| `V`         | Volt           | `voltage`                            |                                     |          |
| `F`         | Farad          | `electrical-capacitance`             |                                     |          |
| `Ω `        | Ohm            | `[electrical-resistance, impedance]` |                                     |          |
| `S`         | Siemens        | `electrical-conductance`             |                                     |          |
| `Wb`        | Weber          | `magnetic-flux`                      |                                     |          |
| `T`         | Tesla          | `magnetic-field`                     |                                     |          |
| `H`         | Henry          | `electrical-inductance`              |                                     |          |
| `°C`        | Degree Celsius | `temperature`                        |                                     |          |
| `lm`        | Lumen          | `luminous-flux`                      |                                     |          |
| `lx`        | Lux            | `illuminance`                        |                                     |          |
| `Bq`        | Becquerel      | `radioactivity`                      |                                     |          |
| `Gy`        | Gray           | `absorbed-dose`                      |                                     |          |
| `Sv`        | Sievert        | `equivalent-dose`                    |                                     |          |
| `kat`       | Katal          | `catalytic-activity`                 |                                     |          |

### Other accepted units

| Unit Symbol | Unit Name | Quantity name | Equivalents         | Base                |
| ----------- | --------- | ------------- | ------------------- | ------------------- |
| `l`         | Litre     | `volume`      | $ \text{dm}^3 $     | $ \text{dm}^3 $     |
| `h`         | Hour      | `time`        | $3600\cdot\text{s}$ | $3600\cdot\text{s}$ |
| `m`         | Minute    | `time`        |                     |                     |
| `d`         | Day       | `time`        |                     |                     |
| `m`         | Month     | `time`        |                     |                     |
| `y`         | Year      | `time`        |                     |                     |
| `bar`       | Bar       | `pressure`    |                     |                     |
|             |           |               |                     |                     |
|             |           |               |                     |                     |

### Units of information

Other non-SI units shall also be accepted such as unit of information

| Unit symbol | Unit name | Quantity name             |
| ----------- | --------- | ------------------------- |
| `b`         | Bit       | Base 2                    |
| `o`         | Octet     | Exactly 8 b               |
| `nat`       | Nat       | Base `e`                  |
| `dit`       | Dit       | Base 10                   |

### Prefix


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

## Validation

### Properties

| Property       | Description                                                  | Example                                             |
| -------------- | ------------------------------------------------------------ | --------------------------------------------------- |
| `optional`     | Type or key is optional, default `false`                     | `foo(optional: false): !str(min: 10)`               |
| `exclusive`    | Cannot be present along with                                 | `foo-or-bar: { !exclusive( foo: !str, bar: !dec ) } |
| `min`          | Minimum length or value (equivalent to `le`)                 | `age: !int(min: 0, max: 200)`                       |
| `max`          | Maximum length or value (equivalent to `ge`)                 |                                                     |
| `repr`         | Representation format (coercive)                             | `!seq(repr: !complex)`, `[1, 20]` becomes `1+20j`   |
| `quantity`     | Quantity value for physical magnitudes                       | `!dec(quantity: "temperature")`                     |
| `dummy`        | The value is dummy and does not appear in the payload. Usually used as a reference on other fields |                                                     |
| `default`      | Default value when missing                                   | `enabled: !bool(default: false)`                    |
| `merge`        | Merge the value with the inherited type, false by default    |                                                     |
| `offset`       | Offset added to the serialized value (used in binary transmissions) |                                                     |
| `scale-factor` | Multiplication factor (used in binary transmissions)         |                                                     |
| `gt`           | The value must be greater than this value                    |                                                     |
| `lt`           | The value must be greater than this value                    |                                                     |
| `ge`           | Greater or equal                                             |                                                     |
| `le`           | Lesser or equal                                              |                                                     |
| `description`  | Description of the key or type                               |                                                     |
| `link`         | List of hyperlinks                                           |                                                     |
| `alias`        | Used to accept values as an alias of another one             | `!bool(alias: {on: true, off: false})`              |
| `resolve`      | Resolve reference when serializing, default `false`          |                                                     |
| `hash`         | Hash of the associated value computed on the binary format. This is useful to make sure a document as not been modified. The hash value can be an external reference. It is computed in SHA256 |                                                     |
| `signature`    | Any value can be signed using a digital signature            |                                                     |

## Encryption 

A UON document can be encrypted. If so, the content of the document is simply a string value usually in base64. UON MUST support the following encryption type: 

* GPG
* Basic AES

## Capabilities

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

## Destructuring 

With the help of references (`!ref`), *UON* data can be easily destructured to only access a subset of data. This might be useful when used within a RESTful API. 

```yaml
!uon(version: 0.0.1) {
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

### Linked list

The example below represents a Git repository with linked references. When serialized, one can choose to flatten or not the references. 

Editors that supports the UON syntax highlight SHOULD be able to resolve references. 

```yaml
uon(version: 0.0.1) !!git-repository {
  head: @(refs.master)
  refs: {
    master: @(b43f2c2f661030e2cd4129787e71052567d4ef5a)
  }
  authors: {
    "jdoe@example.com": "John Doe"
  }
  objects: {
    b43f2c2f661030e2cd4129787e71052567d4ef5a: !!commit {
      tree: @(ee313d61f4eb880e14003a28690ea63980673d9c), 
      parent: @(c9825517d028c352af89a3229f011bdf085f7443),
      author: {
        identity: @(authors."jdoe@example.com"),
        date: !epoch 1528555907
      }        
    },
    c9825517d028c352af89a3229f011bdf085f7443: !!commit {
      tree: @(ea41dba10b54a794284e0be009a11f0ff3716a28), 
      parent: @(2770d3625580ecd3bd4cf57cb5bff9751f8cbb3c),
      author: {
        identity: @(authors."jdoe@example.com"),
        date: !epoch 1528555894
      }        
    },    
    2770d3625580ecd3bd4cf57cb5bff9751f8cbb3c: !!commit {
      tree: @(4d5fcadc293a348e88f777dc0920f11e7d71441c), 
      parent: @(2770d3625580ecd3bd4cf57cb5bff9751f8cbb3c),
      author: {
        identity: @(authors."jdoe@example.com"),
        date: !epoch 1528555881
      }
    },        
    ee313d61f4eb880e14003a28690ea63980673d9c: !!tree {
      "bar": {
        permissions: 100644,
        ref: @(e69de29bb2d1d6434b8b29ae775ad8c2e48c5391)
      },
      "foo": {
        permissions: 100644,
        ref: @(d9e80f6f7602c18d035af9659303047b30204182)
      }     
    },
    e69de29bb2d1d6434b8b29ae775ad8c2e48c5391: !!blob "",
  }
}
```

Here is the schema of this data structure, allowing validation: 

```yaml
!uon(version: 0.0.1) {
   !!git-repository: !schema {
     head: !ref(base: "refs"),
     refs: {
       !keyword: !str
     },
     authors: {
       !str: !str
     },
     objects: {
       !keyword: !any(
         !!commit,
         !!blob,
         !!tree
       ) 
     }
   },
   !!ref: !schema !ref(base: "objects"),
   !!filename: !schema !str,
   !!blob: !schema !any([!blob, !str]),
   !!commit: !schema {
     tree: !!ref,
     parent(optional: true): !!ref,
     author: {
       identity: !ref(base: "authors"),
       date: !datetime
     }
   },
   !!tree: !schema {
     !!filename: {
       permission: !!ref,
       ref: !!ref
     }
   }   
}
```

## RESTful access

When a UON file is exposed on a RESTful API, a *UON* file MUST be parsed automatically. 

###Example

The following *UON* file is stored on a server and made accessible from `http://example.com/midhuna`

```yaml 
!uon(version: 0.0.1) {
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

## Unified API

The unified API should be identical on all supported programming languages

- `to_json()` Identical to `translate('json')`
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

## Implementation

In the context of this draft, no implementation are fully available, however projects in development are available below: 

* Python: http://github.com/uon-language/py-uon