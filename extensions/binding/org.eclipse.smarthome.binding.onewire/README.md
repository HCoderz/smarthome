# OneWire Binding

The OneWire binding integrates OneWire (also spelled 1-Wire) devices. 
OneWire is a serial bus developed by Dallas Semiconductor.
It provides cheap sensors for temperature, humidity, digital I/O and more.
  
## Supported Things

### Bridges

Currently only one bridge is supported. 

The OneWire File System (OWFS, http://owfs.org) provides an abstraction layer between the OneWire bus and this binding. 
The `owserver` is the bridge that connects to an existing OWFS installation. 

### Things

There are different types of things: the generic ones (`counter2`, `digitalio`, `digitalio2`, `digitalio8`, `ibutton`, `temperature`), multisensors built around the DS1923/DS2438 chip (`ms-tx`) and more advanced sensors from Elaborated Networks (www.wiregate.de) (`ams`, `bms`) and Embedded Data System (`edsenv`). 

The thing types `ms-th`and `ms-tv` have been marked deprecated and will be updated to `ms-tx`automatically. 
Manually (via textual configuration) defined things should be changed to `ms-tx`. 

## Discovery

Discovery is supported for things. You have to add the bridges manually.  

## Thing Configuration

It is strongly recommended to use discovery and Paper UI for thing configuration.
Please note that:

* All things need a bridge.
* The sensor id parameter supports only the dotted format, including the family id (e.g. `28.7AA256050000`).
* Refresh time is the minimum time in seconds between two checks of that thing.
It defaults to 300s for analog channels and 10s for digital channels.
* Some thing channels need additional configuration, please see below in the channels section.

### OWFS Bridge (`owserver`)

There are no configuration options for the owserver besides the network address.
It consists of two parts: `address` and `port`.

The `address` parameter is used to denote the location of the owserver instance. 
It supports both, a hostname or an IP address. 

The `port` parameter is used to adjust non-standard OWFS installations.
It defaults to `4304`, which is the default of each OWFS installation.  
  
### Counter (`counter2`)

The counter thing supports the DS2423 chip, a dual counter.
Two `counterX` channels are supported. 

It has two parameters: sensor id `id` and refresh time `refresh`.
 

### Digital I/O (`digitalio`, `digitalio2`, `digitalio8`) 

The digital I/O things support the DS2405, DS2406, DS2408 and DS2413 chips.
Depending on the chip, one (DS2405), two (DS2406/DS2413) or eight (DS2408) `digitalX`  channels are supported.

It has two parameters: sensor id `id` and refresh time `refresh`.

### iButton (`ibutton`)

The iButton thing supports only the DS2401 chips.
It is used for presence detection and therefore only supports the `present` channel.
It's value is `ON` if the device is detected on the bus and `OFF` otherwise.

It has two parameters: sensor id `id` and refresh time `refresh`.

### Multisensor (`ms-tx`)

The multisensor is build around the DS2438 or DS1923 chipset. 
It always provides a `temperature` channel.

Depnding on the actual sensor, additional channels (`current`, `humidity`, `light`, `voltage`, `supplyvoltage`) are added.
If the voltage input of the DS2438 is connected to a humidity sensor, several common types are supported (see below).

It has two parameters: sensor id `id` and refresh time `refresh`.

### Temperature sensor (`temperature`)

The temperature thing supports DS18S20, DS18B20 and DS1822 sensors.
It provides only the `temperature` channel.

It has two parameters: sensor id `id` and refresh time `refresh`. 

### Elaborated Networks Multisensors (`ams`, `bms`)

These things are complex devices from Elaborated networks. 
They consist of a DS2438 and a DS18B20 with additional circuitry on one PCB.
The AMS additionally has a second DS242438 and a DS2413 for digital I/O on-board.
Analog light sensors can optionally be attached to both sensors.

These sensors provide `temperature`, `humidity` and `supplyvoltage` channels.
If the light sensor is attached and configured, a `light` channel is provided, otherwise a `current` channel.
The AMS has an additional `voltage`and two `digitalX` channels.

It has two (`bms`) or four (`ams`) sensor ids (`id`, `id1`, `id2`, `id3`).
The first id is always the main DS2438, the second id the DS18B20 temperature sensor.
In the case of the AMS, the third sensor id has to be the second DS2438 and the fourth the DS2413.

Additionally the refresh time `refresh` can be configured.
The AMS supports a `digitalrefresh` parameter for the refresh time of the digital channels.

Since both multisensors have two temperature sensors on-board, the `temperaturesensor` parameter allows to select `DS18B20` or `DS2438` to be used for temperature measurement.
This parameter has a default of `DS18B20` as this is considered more accurate.
The `temperature` channel is of type `temperature` if the internal sensor is used and of type `temperature-por-res` for the external DS18B20.

The last parameter is the `lightsensor` option to configure if an ambient light sensor is attached.
It defaults to `false`.
In that mode, a `current`  channel is provided.
If set to `true`, a `light` channel is added to the thing.
The correct formula for the ambient light is automatically determined from the sensor version.

### Embedded Data System Environmental sensors (`edsenv`)

This thing supports EDS0064, EDS0065, EDS0066 or EDS0067 sensors.
It has two parameters: sensor id `id` and refresh time `refresh`.

All things have a `temperature` channel.
Additional channels (`light`, `pressure`, `humidity`, `dewpoint`, `abshumidity`) will be added if available from the sensor automatically.


## Channels

| Type-ID             | Thing                      | Item                     | readonly   | Description                                        |
|---------------------|----------------------------|--------------------------|------------|----------------------------------------------------|
| absolutehumidity    | ms-tx, ams, bms, edsenv    | Number:Density           | yes        | absolute humidity                                  |
| current             | ms-tx, ams                 | Number:ElectricCurrent   | yes        | current                                            |
| counter             | counter2                   | Number                   | yes        | countervalue                                       |
| dewpoint            | ms-tx, ams, bms, edsenv    | Number:Temperature       | yes        | dewpoint                                           |
| dio                 | digitalX, ams              | Switch                   | no         | digital I/O, can be configured as input or output  |
| humidity            | ms-tx, ams, bms, edsenv    | Number:Dimensionless     | yes        | relative humidity                                  |
| humidityconf        | ms-tx                      | Number:Dimensionless     | yes        | relative humidity                                  |
| light               | ams, bms, edsenv           | Number:Illuminance       | yes        | lightness                                          |
| present             | all                        | Switch                   | yes        | sensor found on bus                                |
| pressure            | edsenv                     | Number:Pressure          | yes        | environmental pressure                             |
| supplyvoltage       | ms-tx                      | Number:ElectricPotential | yes        | sensor supplyvoltage                               |
| temperature         | temperature, ms-tx, edsenv | Number:Temperature       | yes        | environmental temperature                          |
| temperature-por     | temperature                | Number:Temperature       | yes        | environmental temperature                          |
| temperature-por-res | temperature, ams, bms      | Number:Temperature       | yes        | environmental temperature                          |
| voltage             | ms-tx, ams                 | Number:ElectricPotential | yes        | voltage input                                      |

### Digital I/O (`dio`)

Channels of type `dio` channels each have two parameters: `mode` and `logic`.

The `mode` parameter is used to configure this channels as `input` or `output`.

The `logic` parameter can be used to invert the channel.
In `normal` mode the channel is considered `ON` for logic high, and `OFF` for logic low.
In `inverted` mode `ON` is logic low and `OFF` is logic high.

### Humidity (`humidity`, `humidityconf`, `abshumidity`, `dewpoint`)

Depending on the sensor, a `humidity` or `humidityconf` channel may be added.
This is only relevant for DS2438-based sensors of thing-type `ms-tx`.
`humidityconf`-type channels have the `humiditytype` parameter.
Possible options are `/humidity` for HIH-3610 sensors, `/HIH4000/humidity` for HIH-4000 sensors, `/HTM1735/humidity` for HTM-1735 sensors and `/DATANAB/humidity` for sensors from Datanab.

All humidity sensors also support `absolutehumidity` and `dewpoint`.

### Temperature (`temperature`, `temperature-por`, `temperature-por-res`)

There are three temperature channel types: `temperature`, `temperature-por`and `temperature-por-res`.
The correct channel-type is selected automatically by the thing handler depending on the sensor type.

If the channel-type is `temperature`, there is nothing else to configure.

Some sensors (e.g. DS18x20) report 85 °C as Power-On-Reset value.
In some installations this leads to errorneous temperature readings.
If the `ignorepor` parameter is set to `true` 85 °C values will be filtered.
The default is `false` as correct reading of 85 °C will otherwise be filtered, too.

A channel of type `temperature-por-res` has one parameter: `resolution`.
OneWire temperature sensors are capable of different resolutions: `9`, `10`, `11` and `12` bits.
This corresponds to 0.5 °C, 0.25 °C, 0.125 °C, 0.0625 °C respectively.
The conversion time is inverse to that and ranges from 95 ms to 750 ms.
For best performance it is recommended to set the resolution only as high as needed. 
 
## Full Example

This is the configuration for a OneWire network consisting of an owserver as bridge (`onewire:owserver:mybridge`) as well as a temperature sensor and a BMS as things (`onewire:temperature:mybridge:mysensor`, `onewire:bms:mybridge:mybms`). 

### demo.things:

```
Bridge onewire:owserver:mybridge [ 
    network-address="192.168.0.51" 
    ] {
    
    Thing temperature mysensor [
        id="28.505AF0020000", 
        refresh=60
        ] {
            Channels:
                Type temperature-por-res : temperature [
                    resolution="11"
                ]
        } 
    
    Thing bms mybms [
        id="26.CD497C010000", 
        id1="28.D3E45A040000", 
        lightsensor=true, 
        temperaturesensor="DS18B20", 
        refresh=60
        ] {
            Channels:
                Type temperature-por-res : temperature [
                    resolution="9"
                ]
        } 
}
```

### demo.items:

```
Number:Temperature MySensor "MySensor [%.1f %unit%]" { channel="onewire:temperature:mybridge:mysensor:temperature" }
Number:Temperature MyBMS_T "MyBMS Temperature [%.1f %unit%]" { channel="onewire:bms:mybridge:mybms:temperature" }
Number:Dimensionless MyBMS_H "MyBMS Humidity [%.1f %unit%]"  { channel="onewire:bms:mybridge:mybms:humidity" }
```

### demo.sitemap:

```
sitemap demo label="Main Menu"
{
    Frame {
        Text item=MySensor
        Text item=MyBMS_T
        Text item=MyBMS_H
    }
}
```
