# Sandbox

```yaml
{
  # Month name
  !!month: !schema(
  ) !oneof {
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
  !!month-fr: !schema !match({
    january: "Janvier",
    february: "Février",
  }) !month

  # http://uon-language.io/type/country
  !!country: !type(
    id: !uuid d925fe72-6e69-11e8-adc0-fa7ae01bbebc,
    desc: "Standardized country code with associated country name",
    example: "!country ch",
    rank: 1,
    version: 0.0.1
  ) !oneof {
    af("Afghanistan"),
    ax("Åland Islands"),
    al("Albania")
  },

  # Countries according to ISO 3166-1
  # http://uon-language.io/type/country
  !!country: !type(
    id: !uuid d925fe72-6e69-11e8-adc0-fa7ae01bbebc,
    desc: "Standardized country code with associated country name",
    example: "!country ch",
    rank: 1,
    version: 0.0.1
  ) !oneof {
    af("Afghanistan"),
    ax("Åland Islands"),
    al("Albania")
  },

  # Country-name
  # Read name property and render it as a string
  !!country-name: !type(
    desc: "Cast a !country into its name (access name property)",
  ) !str !prop(name) !country,

  # International geographic point location (ISO 6709:2008)
  !!geo: !schema(
    desc: "Standardized Geographic point location"
    example: "!geo \"+401213.1N+4012.22E123.45\"",
    rank: 1,
    version: 0.0.1
  ) !!str(
    pattern: //,
    coerce: {
      !map: {
        latitude: {
          deg: !operation "$1+42/12+$2",
          min: "$2",
          sec: "$3",
        },
        longitude: {
          deg: "$1",
          min: "$2",
          sec: "$3"
        },
        height: !number(unit: meter) "$8"
      }
    }
  ),

  !!person: !schema(
    dependencies: {
      http://uon-language.io/type/country,
      http://uon-language.io/type/geo,
    }
  ) {
    name: !str(min: 1),
    country: !country,
    location: !geo,
    other-location: !@(http://uon-language.io/type/geo),
    genre: !any { male, female },
    partner: !person # Recursive reference
  }
}
```

## Advanced features?

### Aggregation

One missing feature especially on MQTT is data aggregation. Let's take the example of a  `!!3d-accelerometer` that streams the following at 100 Hz

```yaml
{
  t: 5433335321 ns
  x: 12.43 m/s²
  y: 32.5 m/s²
  z: 0.05 m/s²
}
```

Despite that the binary data is really small, when streamed at that speed over AMQP or MQTT, it would quickly flood the network. So if we define a new type `!3d-accelerometer-3` which contain 3 values, we will lose the interoperability because it is not an `!!3d-accelerometer` anymore. Let's imagine this schema:

```yaml
!3d-accelerometer-3: !schema [ !3d-accelerometer, !3d-accelerometer, !3d-accelerometer ]
```

Wouldn't be better to rewrite it as follow:

```yaml
!schema: !repeat(3, !3d-accelerometer)
```

But how can it be understood or either refined? I am not sure I have the answer yet. The scenario is the following. Somebody is expecting from a topic 3d acceleration values. It is expecting a `!3d-accelerometer` payload. So the device should be able to refine the schema in a way it can aggregate several values in the same payload. But not all types are meant to be aggregated. A schema for instance does not change over time, a user documentation, a network description also does not change over time.

In addition, when the payload is very light, such as our temperature sensor, carrying the timestamp for each value is stupid. What we should have instead is the initial timestamp and the increase between the values. Some payload could also have values that do not change over a period of time, but they still need to be transmitted. Let's review this

* Find a way to gather data into the same dataset
* But doing so without the need to define a new schema.
* Having a solution to group similar values that do not often change
* Add a known offset to some values to retrieve the time increment.
* `i * factor + offset` would cover 99% of the use cases

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


# Notes and deep thoughts

Of course, this section will be removed from the specification once all the ideas below are fully crystallized without inconsistencies.

I am trying to find the simplest and most elegant writing to address typical use cases of what a serialization language should serve.

Again, *UON* is not a programming language, I don't want to think about loops, functions, or conditionals. Everything should be easily parsed without much computation power. If someone wants to extend the capabilities, it can do at the application level by binding a `resolver` function to a existing type.

Data transformation has to be kept minimal. *UON* is not a language made to transform information, but it has to be able to represent data according to the destination domain: binary payload, readable human documents, ... Arithmetic transformations shall be possible to represent data written from one unit to another. A payload that gives an energy value in joules could be presented in kWh if required during the rendering.

One important application of this feature is IoT. Let's imagine a small temperature sensor. It is composed of a very low-power MCU that transmits temperature data using BLE (Bluetooth Low Energy). To be interoperable, it uses *UON* to describe and encode the payload.

```yaml
# http://iot-federation.io/uon/schemas/temperature-sensor
!temperature-sensor: !schema(
  description: "Standardized temperature sensor descriptor"
) {
  temperature(required: true): !!number(quantity: temperature),
  sampling-time(optional: true): !!number(quantity: frequency)
}
```

The above schema is device-agnostic. It does not define any unit nor any sized types allowing small devices to transmit data using `!uint8` or `!float32`. So the data consumer that receives the binary data shall be able to interpret the data by only knowing this reference schema. However, the payload should be lightweight and it should not include details such as encoding type.

In the case of our, temperature sensor the binary representation looks like this, eight-bit long.

```
[ !!uint8 ]
```

The data consumer that listens to this sensor (imagine a MQTT payload), will need more information. So, the device establishes a refined schema:

```yaml
# Inherit of !temperature-sensor
!schema(!temperature-sensor, refine: true, id: 02d28a9a-6e7b-11e8-adc0-fa7ae01bbebc) {
  temperature(inherit: true): !!uint8(unit: celsius),
}
```

That's it! The new refined schema is compatible with `!temperature-sensor` the temperature value is still a `!number`, it has the same quantity (temperature). The optional `sampling-time` is still optional so if it is transmitted, the device will need to encode it using a full-type notation i.e. the type identifier (0xab for a `!float64`) followed by the 64-bit data.

Now this refined schema has to be exchanged from the BLE temperature sensor to the data consumer as a contract that describes the payload. How is it done? Well, I am not yet sure of the best solution but here some possibility:

* The refined schema is transmitted once by the device either in plain or binary data. Notice that binary format get rid of properties unless it belongs to  `!schema` so in Python doing `my_refined_schema.to_binary()` gives full binary payload that can be parsed and understood. The good thing is only the altered properties and types are mentioned.
* The refined schema is stored on a global server and a URI gives the access to it. The data receiver should then ask the device for the URI of the refined schema. This is possible in MQTT by simply subscribing to `/my-device/.schema`

```yaml
!@(http://my-sensor.com/schemas/s231abx)
```

The generated binary payload for this will be still heavy because it contains the string:  `http://my-sensor.com/schemas/s231abx`. With additional data such as `0x80` (schema), and the property identifiers.

* The refined schema is stored somewhere on a global server, but the device transmits only the `uuid` of the schema. The payload is then 128 bit + the schema type `0x80`.  Its better, but the data receiver should be able to match the `uuid` somewhere.

Let's consider for one moment the data receiver. A nice MQTT client who subscribes to this data:

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

The received message will decode into:

```yaml
{
  temperature = 23 °C
}
```

Why is it displayed in degree Celsius? Because the refined schema specifies it so. What about if we would like to get an agonistic temperature, in SI units. Well, the original schema does not specify the unit. By default units are SI compliant, so given in Kelvin. The fully agnostic answer will be:

```yaml
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

```yaml
!!uon(version: 0.0.1) !@(http://iot-federation.io/uon/schemas/temperature-sensor) {
  temperature: 296.15
}
```

```python
temp.to_uon(version: '0.0.1', describe: true)
```

Let's now imagine we want to transmit the payload to another very lightweight device that understands the temperature sensor, but in Kelvin with a negative offset, and with data given in `!float32`. I simply need to refine the original schema and coerce the received payload into it:

```python
refined = """!schema(!temperature sensor) {
  temperature(inherit: true): !!float32(unit: kelvin offset: -10.2)}"""

def on_message(client, userdata, msg):
   client.publish('topic-to/other-device',
       parser.from_binary(msg.payload).coerce(refined).to_binary())
```

Why would it work that smoothly? Because the payload is still a derived version of `temperature-sensor` so the properties of `temperature` can be modified without changing the original type `!number`.