# DHT

The AtomVM DHT library can be used to drive DHT11 and DHT12 temperature and humidity sensors that can be attached to ESP32 devices.

The AtomVM DHT library is only supported on the ESP32 platform.

> Note.  The DHT11 and DHT12 are notoriously inaccurate temperature sensors, though they are cheap and may be good enough for simple applications.  This driver uses bit banging on GPIO pins and may not make the best use of resources on your ESP32.  Consider alternative temperature sensors, such as the SHT3x or Bosch BMP or BME line of sensors.

## Build Instructions

The AtomVM DHT library is implemented as an AtomVM component, which includes some native C code that must be linked into the ESP32 AtomVM image.  In order to build and deploy this client code, you must build an AtomVM binary image with this component included.

For general instructions about how to build AtomVM and include third-party components into an AtomVM image, see the [AtomVM Build Instructions](https://www.atomvm.net/doc/master/build-instructions.html).

Once the AtomVM image including this component has been built, you can flash the image to your ESP32 device.  For instructions about how to flash AtomVM images to your ESP32 device, see the AtomVM [Getting Started Guide](https://www.atomvm.net/doc/master/getting-started-guide.html).

Once the AtomVM image including this component has been flashed to your ESP32 device, you can then include this project into your [`rebar3`](https://www.rebar3.org) project using the [`atomvm_rebar3_plugin`](https://atomvm.github.io/atomvm_rebar3_plugin), which provides targets for building AtomVM packbeam files and flashing them to your device.

## Programmer's Guide

The DHT API can be used to drive common DHT11 and DHT22 temperature and humidity sensors.

DHT22 devices are reported to have higher resolution and range in both temperature and humidity readings.

Temperature and humidity readings can be taken at intervals not less that 1 second apart for DHT11 devices, and 2 seconds apart for DHT22 devices.  The DHT API will ensure that any one DHT instance will not read at less than the recommended interval.

### Lifecycle

To start the DHT driver, use the `dht:start/1` function.  Pass in a configuration map, which may contain the following entries

| Key | Value | Default | Required | Description |
|-----|-------|---------|----------|-------------|
| `pin` | `integer()` | none | yes | The data pin to which the DHT11 or DHT22 is connected. |
| `device` | `dht_11 \| dht_12` | `dht_11` | no | The device type (DHT11 or DHT12). |

For example:

    %% erlang
    Config = #{
        pin => 22,
        device => dht_11
    },
    {ok, DHT} = dht:start(Config),
    ...

Stop the DHT driver via the `dht:stop/1` function.  Supply the reference to the DHT instance returned from `dht:start/1`:

    %% erlang
    ok = dht:stop(DHT)

### Taking Readings

Take readings using the `dht:take_reading/1` function.  Supply a reference to the DHT instance returned from `dht:start/1`.

A reading is expressed as pair containing the temperature (in degrees celcius) and relative humidity (expressed as a percentage).  The temperature and humidity readings are each pairs of integers, representing the whole and fractional part (to a precision of one digit).

For example,

    %% erlang
    case dht:take_reading(DHT) of
        {ok, Reading} ->
            {{Temp, TempFractional}, {Hum, HumFractional}} = Reading,
            io:format("Temperature: ~p.~pC  Humidity: ~p.~p%~n", [Temp, TempFractional, Hum, HumFractional]);
        Error ->
            io:format("Error taking reading: ~p~n", [Error])
    end,
    ...
