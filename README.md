# AtomVM DHT Driver

The AtomVM Nif can be used to drive DHT11 and DHT12 temperature and humidity sensors that can be attached to ESP32 SoCs.

This driver is included as an add-on to the AtomVM base image.  In order to use this driver, you must be able to build the AtomVM virtual machine, which in turn requires installation of the Espressif IDF SDK and tool chain.

> Note.  For build instructions of the AtomVM Virtual machine on the ESP32 platform, see [here](https://github.com/bettio/AtomVM/blob/master/doc/atomvm-esp32.md#adding-custom-nifs-and-third-party-components).

## Getting Started

Clone this repository in the `src/platforms/esp32/components` directory of the AtomVM source tree.

    shell$ cd .../AtomVM/src/platforms/esp32/components
    shell$ git clone https://github.com/fadushin/atomvm_dht.git

Create a `component_nifs.txt` file, if it does not already exist, in `src/platforms/esp32/main`.  The contents of this file should contain a single line for the ESP32 cam driver:

    atomvm_dht

Build and flash AtomVM (typically from `src/platforms/esp32`)

    shell$ cd .../AtomVM/src/platforms/esp32
    shell$ make flash
    ...

Once the AtomVM image is flashed to the ESP32 device, it includes NIFs for interfacing with DHT11 and DHT22 sensors attached to the device.

# DHT API

The DHT API can be used to drive common DHT11 and DHT22 temperature and humidity sensors.

DHT22 devices are reported to have higher resolution and range in both temperature and humidity readings.

Temperature and humity readings can be taken at intervals not less that 1 second apart for DHT11 devices, and 2 seonds apart for DHT22 devices.  The DHT API will ensure that any one DHT instance will not read at less than the recommended interval.

### Sample Code

The following code illustrates use of the DHT API:

    %% start a DHT11 reader on Pin 22
    {ok, DHT} = dht:start(22),

    %% take a measurement and print the results
    case dht:measure(DHT) of
        {ok, Measurement} ->
            {Temp, TempFractional, Hum, HumFractional} = Measurement,
            io:format("Temperature: ~p.~pC  Humidity: ~p.~p%~n", [Temp, TempFractional, Hum, HumFractional]);
        Error ->
            io:format("Error taking measurement: ~p~n", [Error])
    end,
    ...

### Example Program

The `dht_example` program illustrates use of the DHT API by taking a temperature and humidity reading every 30 seconds, and displaying the result on the console.

> Note.  Building and flashing the `dht_example` program requires installation of the [`rebar3`](https://www.rebar3.org) Erlang build tool.

To run this example program, connect the positive (+) lead on the DHT device to +3.3v power on the ESP32, the negative lead (-) to a ground pin on the ESP32, and the data pin to GPIO pin 21 on the ESP32 device.

    +-----------+
    |           o-------- +3.3v
    |   DHT11   o-------- data
    |     or    o-------- unused
    |   DHT22   o-------- gnd
    |           |
    +-----------+

Build the example program and flash to your device:

    shell$ cd .../AtomVM/src/platforms/esp32/components/atomvm_dht/examples/dht_example
    shell$ rebar3 esp32_flash

> Note.  This build step makes use of the [`atomvm_rebar3_plugin`](https://github.com/fadushin/atomvm_rebar3_plugin).  See the README.md for information about parameters for setting the serial port and baud rate for your platform.

Attach to the console usin gthe `monitor` Make target in the AtomVM ESP32 build:

    shell$ cd .../AtomVM/src/platform/esp32
    shell$ make monitor
    Toolchain path: /work/opt/xtensa-esp32-elf/bin/xtensa-esp32-elf-gcc
    WARNING: Toolchain version is not supported: crosstool-ng-1.22.0-95-ge082013a
    ...
    Found AVM partition: size: 1048576, address: 0x110000
    Booting file mapped at: 0x3f430000, size: 1048576
    Starting: dht_example.beam...
    ---
    Temperature: 21.3C  Humidity: 41.6%
    Temperature: 21.4C  Humidity: 41.4%
    Temperature: 21.3C  Humidity: 41.3%
    ...
