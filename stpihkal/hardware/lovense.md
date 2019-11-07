# Lovense

## Introduction

[Lovense](https://www.lovense.com/) has been manufacturing sex toys since 2011.

Many of their product models have undergone revisions over the years. For
example, the Nora is currently on generation 6. If there are differences in
capabilities between revisions, they may have been overlooked here. We also
assume that devices are running with all available firmware updates installed.

## Bluetooth Details

While all Lovense devices use the same message protocol, they can communicate
over different Bluetooth differently depending on when they were released.

### Bluetooth 2.0 Support

The first devices released by Lovense, Max and Nora, used both Bluetooth 2.0 SPP
(emulating a serial port) and Bluetooth 4.0 Low Energy (BTLE). This was most
likely due to the sparse mobile support of BTLE when they were released. Support
for Bluetooth 2.0 was not included in future models.

### Bluetooth 4.0 Low Energy Support

Lovense device names always start with `LVS-`. What comes after that varies
depending on when the device was released. Early devices used names including
the single character model identifiers [described below](#device-information),
such as `LVS-A011`, while newer devices use the full model name, such
`LVS-Edge36`. The trailing digits indicate denote the firmware version the
device is running.

Lovense devices use Bluetooth 4.0 GATT characteristics to mimic the serial port
style control of their Bluetooth 2.0 protocol. The Bluetooth GATT service and
characteristic UUIDs can differ between models and firmware firmware versions.

Service IDs observed so far match one of the following patterns:

- `0000fff0-0000-1000-8000-00805f9b34fb` for the first generation.
- `6e400001-b5a3-f393-e0a9-e50e24dcca9e` for second generation.
- <code><b>XY</b>300001-002<b>Z</b>-4bd4-bbd5-a6920e4c5653</code> for the rest,
  where:
  - `X` is `4` or `5`
  - `Y` is any hex digit, `0` to `f`
  - `Z` is `3` or `4`

While some Bluetooth APIs can search for wildcard services, others such as
WebBluetooth require an exact service UUID to connect. For that case, it's
recommended to just generate out all 64 variations of the last service UUID
pattern, for a total of 66 service UUIDs, and specify them all in
`optionalServices` portion of a WebBluetooth connection filter.

Once connected to the service, it will expose two GATT characteristics. One
characteristic will be declared as writable, and it is used for transmitting
messages by setting its value. The other characteristic will be declared as
non-writable, and it is used for receiving messages by registering a listener
for changes to its value.

## Protocol

### Structure

Commands and replies are strings, terminated by semicolons.

All commands start with a command name, optionally followed by parameter values,
delimited using colon characters.

Most commands always receive a single message in reply, but some receive several
or even a continuous stream.

Commands that do not have more specific replies will typically reply with `OK;`
on success and `ERR;` on failure.

If the command name is unrecognized, the device will reply with some variation
of `UNKNOWN;`.

### Index

<!-- properties -->

- [Device Information](#device-information): `DeviceType`, `GetBatch`

<!-- state -->

- [Battery](#battery): `Battery`
- [Motion](#motion): `StartMove`, `StopMove`

<!-- controls -->

- [Vibration](#vibration): `Vibrate`, `Vibrate1`, `Vibrate2`
- [Rotation](#rotation): `Rotate`,
  `RotateAntiClockwise`
- [Inflation](#inflation): `Air`

<!-- programming -->

- [Patterns](#patterns): `GetPatten`, `Preset`
- [Levels](#levels): `GetLevel`, `SetLevel`

<!-- settings -->

- [Light Settings](#light-settings): `GetLight`, `GetALight`, `Light`, `ALight`
- [Connectivity Settings](#connectivity-settings): `GetAS`, `AutoSwith`

<!-- niche -->

- [Power Off](#power-off): `PowerOff`

### Device Information

#### `DeviceType;` (All)

Returns device's model, firmware version, and Bluetooth MAC address, as a
colon-delimited list. The model is identified using a single character, as
follows:

- `A` or `C`: Nora
- `B`: Max
- `L`: Ambi
- `O`: Osci
- `P`: Edge
- `S`: Lush
- `W`: Domi
- `Z`: Domi

For example, given a Nora device, running v1.1 firmware, with a Bluetooth
address of `00:82:05:9A:D3:BD`:

```
DeviceType;
```

```
C:11:0082059AD3BD;
```

#### `GetBatch;` (All)

Returns the production batch date for this device in `YYMMDD` format.

For example, given a device with a production batch date of January 24, 2019:

```
GetBatch;
```

```
190124;
```

### Battery

#### `Battery;` (All)

Returns the battery level of the device as an integer percentage from 0-100.

This value may sometimes be prefixed with an `s` character. The reason for this
is uncertain, but it might indicate that the toy is active and the battery is
under load.

For example, given an idle device with 85% battery remaining:

```
Battery;
```

```
85;
```

### Motion

#### `StartMove:1;` and `StopMove:1;` (Max, Nora)

Enables and disables the accelerometer data stream. The device will send
accelerometer data constantly <!-- TODO: approximately how frequently? --> until
stop command is sent. Accelerometer data messages will be prefixed with the `G`
character <!-- TODO: to distinguish it from other message replies sent
concurrently? -->, followed by 3 16-bit signed integers in little-endian hexadecimal,
indicating the 3D force vector measured by the device.

For example:

```
StartMove:1;
```

```
GEF008312ED00;
GE021316CA2CB;
```

```
StopMove:1;
```

```
OK;
```

### Vibration

#### `Vibrate:$SPEED;` (All)

Sets the vibration speed to `$SPEED`, using an integer scale from 0 to 20.

For example, to set the vibration speed to 10/20 (50%):

```
Vibrate:10;
```

```
OK;
```

#### `Vibrate1:$SPEED;`, and `Vibrate2:$SPEED;` (Edge)

Edge devices have two separate vibrators which may be controlled independently
using the `Vibrate1` and `Vibrate2` variants of the `Vibrate` command.

### Rotation

#### `Rotate:$CLOCKWISE:$SPEED;` (Nora)

Sets the rotation speed to `$SPEED` of the Nora device, using an integer scale
from 0 to 20. If "$CLOCKWISE" is `True`, rotates clockwise, if it is `False`,
rotates anticlockwise.

For example, to set the Nora device to rotate at 15/20 (75%) speed in the
clockwise direction:

```
RotateClockwise:10;
```

```
OK;
```

#### `Rotate:$SPEED;` (Nora)

Adjusts the rotation speed without changing the direction.

#### `RotateChange;` (Nora)

Toggles the rotation direction without changing its speed.

### Inflation

#### `Air:Level:$LEVEL;` (Max)

Set the inflation level of the device, using an integer scale from 0 to 5.

For example, to set the device to 3/5 inflation (60%):

```
Air:Level:3;
```

```
OK;
```

#### `Air:In:$LEVELS;` (Max)

Increases the inflation level of the Max device by `$LEVELS`.

#### `Air:Out:$LEVELS;` (Max)

Decreases the inflation level of the Max device by `$LEVELS`.

### Patterns

#### `GetPatten;` (Lush, Hush, Ambi, Domi, Edge, Osci)

List the indexes of the patterns that are currently programmed into the device.
The maximum number of patterns in 10, so each index will always be a single
digit.

For example, given a device with 5 patterns programmed (with indicies 0 through
4):

```
GetPatten;
```

```
P:01234;
```

#### `GetPatten:$INDEX;` (Lush, Hush, Ambi, Domi, Edge, Osci)

Returns a pattern that is currently programmed into the device.

Each pattern is represented as a series of digits from 0 to 9, each indicating
the vibration level for one half-second of the pattern. (This is different than
the 0 to 20 scale used by the Vibrate command.)

The response is split into multiple messages, each containing up to 12 digits (6
seconds). Each response has a prefix indicating the pattern index, the number of
parts in the response, and the index of the response.

For Domi devices, the pattern length must be between 5 and 50 seconds, so the
response will use a maximum of 9 messages, and the part count and indicies will
always be a single digit.

For example, given a Domi device:

```
GetPatten:4;
```

```
P4:1/5:000042003720;
P4:2/5:000002436658;
P4:3/5:997339993001;
P4:4/5:291111115111;
P4:5/5:1110000000;
```

For Lush devices, the part count and indicies are padded to always use two
digits.

For example, given a Lush device:

```
GetPatten:4;
```

```
P4:01/01:346797643;
```

#### `Preset:$INDEX;` (Lush, Hush, Ambi, Domi, Edge, Osci)

Starts running a programmed pattern on a loop. Takes an positive integer pattern
index to start running it, or 0 to stop running the pattern.

While Domi is able to take any pattern index, from 0 to 10, Lush 2 only seems to
be able to take indices from 0 to 4. Other toys have not been tested.

```
Preset:8;
```

```
OK;
```

### Levels

#### `GetLevel;` (Domi)

Gets the low, medium, and high speed levels configured for the Domi device. The
values are comma-delimited and use an integer scale from 0 to 20.

These levels are the first three options when operating the Domi device
manually. (The later options are programmed [patterns](#patterns).)

For example, to retrieve the levels from a factory-default Domi device:

```
GetLevel;
```

```
1,9,20;
```

#### `SetLevel:$INDEX:$LEVEL;` (Domi)

Sets the low (index `1`), medium (index `2`), or high (index `3`) speed level
configured for the device to the specified `$LEVEL`, using an integer scale from
0 to 20.

### Light Settings

#### `GetLight;` (All)

Gets whether the LED light in the device's power button is enabled (`1`) or
disabled (`0`).

For example, given a device where the LED is enabled:

```
GetLight;
```

```
Light:1;
```

#### `SetLight:$ENABLED;` (All)

Gets whether the LED light in the device's power button is enabled (`on`) or
disabled (`off`)

For example, to disable the LED:

```
Light:off;
```

```
OK;
```

#### `GetAlight` (Domi)

Gets whether the ring of LED lights around the head of the Domi device's wand is
enabled (`1`) or disabled (`0`).

For example, given a device where the ring of LEDs is enabled:

```
GetAlight;
```

```
Alight:1;
```

#### `ALight:$ENABLED;` (All)

Gets whether the LED light in the device's power button is enabled (`On`) or
disabled (`Off`)

For example, to disable the ring of LEDs:

```
ALight:Off;
```

```
OK;
```

### Connectivity Settings

#### `AutoSwith:$STOP_ON_DISCONNECT:$RESUME_ON_RECONNECT;` (All)

Enables or disables these device behaviours around Bluetooth disconnections:

1. Stop all device activity when disconnected.
2. Resume stopped activity when reconnected.

Each option must be specified to be `On` or `Off`.

For example, to configure the device to stop all activity on disconnect, and to
not automatically resume activity once reconnected:

```
AutoSwith:On:Off;
```

```
OK;
```

#### `GetAS;` (All)

Gets the current value of the two options described above. The response is
prefixed with `AutoSwith:`, colon-delimited and uses `1` to indicate enabled and
`0` to indicate disabled. (Note that this is different from the `On` and `Off`
used when setting the options.)

For example, given a device configured to stop all activity on disconnect, and
to not automatically resume activity once reconnected:

```
GetAS;
```

```
AutoSwith:1:0;
```

### Power Off

#### `PowerOff;` (All)

Entirely powers off the device. It won't be possible to reconnect until the
device is manually powered back on using its physical controls.

```
PowerOff;
```

```
OK;
```
