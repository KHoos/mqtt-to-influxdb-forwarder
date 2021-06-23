# zigbee2mqtt to InfluxDB forwarder #

This tool forwards IoT sensor data from zigbee2mqtt to an InfluxDB instance.
It is a fork from [mhaas/mqtt-to-influxdb-forwarder: IoT MQTT to InfluxDB forwarder](https://github.com/mhaas/mqtt-to-influxdb-forwarder),
I changed a few lines to parse mqtt messages from [zigbee2mqtt](https://www.zigbee2mqtt.io/) to import the sensor data into influxdb.

## MQTT topic structure ##

The topic structure should be path-like, where the first element in the
hierarchy contains the standard name. Below that the unique sensor address is
expected, the individual measurements are published as leaf nodes or as a json
object with multiple measurements. Each sensor node can have multiple sensors.

The tool takes a list of node names and will auto-publish measurements found
below these node names that have the pattern of a unique sensor address. Any
measurements which look numeric will be converted to a float.

### Zigbee2mqtt MQTT topic structure ###

zigbee2mqtt/0x00158d0006fafb00 {"battery":100,"humidity":37.86,"linkquality":84,"pressure":1032,"temperature":28.67,"voltage":3045}

## Translation to InfluxDB data structure ##

The MQTT topic structure and measurement values are mapped as follows:

- the unique address is mapped to a tag named sensor\_address
- the InfluxDB measurement name is set to 'environment'
- the measurement value is stored as a field named 'value'.
- the json tags are used as field names and the values set.

### Example translation ###

The following log excerpt should make the translation clearer:

	DEBUG:forwarder.InfluxStore:Writing InfluxDB point: {'fields': {u'linkquality': 84.0, u'temperature': 28.67, u'battery': 100.0, u'humidity': 37.86, u'pressure': 1032.0, u'voltage': 3045.0}, 'tags': {'sensor_address': u'0x00158d0006fafb00'}, 'measurement': 'environment'}

## Complex measurements ##

If the MQTT message payload can be decoded into a JSON object, it is considered a
complex measurement: a single measurement consisting of several related data points.
The JSON object is interpreted as multiple InfluxDB field key-value pairs.
In this case, there is no automatic mapping of the measurement value to the field
named 'value'.

### Example translation ###

An example translation for a complex measurement:

    DEBUG:forwarder.MQTTSource:Received MQTT message for topic zigbee2mqtt/0x00158d0006fafb00 with payload {"battery":100,"humidity":33.35,"linkquality":93,"pressure":1034,"temperature":28.05,"voltage":3055}
    DEBUG:forwarder.InfluxStore:Writing InfluxDB point: {'fields': {u'linkquality': 93.0, u'temperature': 28.05, u'battery': 100.0, u'humidity': 33.35, u'pressure': 1034.0, u'voltage': 3055.0}, 'tags': {'sensor_address': u'0x00158d0006fafb00'}, 'measurement': 'environment'}


### Example InfluxDB query ###

    select temperature from environment;

The data stored in InfluxDB via this forwarder are easily visualized with [Grafana](http://grafana.org/)

## License ##

See the LICENSE file.

## Versioning ##

[Semantic Versioning](http://www.semver.org)
