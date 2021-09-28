# Lovense

## Background

[Lovense](https://www.lovense.com/) has been manufacturing wireless Bluetooth
sex toys since 2011. Many of their models lines have undergone multiple hardware
and firmware revisions in that time. The protocol changes are usually minor, but
for clarity: unless otherwise indicated, this document refers to the latest
hardware and firmware revisions for each model.

## Bluetooth Details

Lovense toys use a serial-style RPC protocol over Bluetooth, with command
messages sent from the client and response messages sent from the toy. Messages
are terminated by `;` semicolons. Valid commands will always recieve at least
one response, unless that command itself shuts down the toy first.

Depending on the model and firmware versions, valid commands that do not have
meaningful return value will respond with either the string `OK` or the original
command string, while invalid commands will respond with either the string `ERR`
or the original command string prefixed with `UNKNOWN,`.

Here is an example session, with whitespace added to distinguish transmitter and
reciever.

```
DeviceType;

  C:11:0082059AD3BD;

GetBatch;

  190124;

Battery;

  95;

GetPatten;

  P:01234;

GetPatten:4;

  P4:1/5:000042003720;
  P4:2/5:000002436658;
  P4:3/5:997339993001;
  P4:4/5:291111115111;
  P4:5/5:1110000000;

PowerOff;
```

Early Lovense toys supported exchanging these messages over a Bluetooth SPP
duplex stream for backwards compatibility, but the preferred (and now only)
approach is instead to use a Bluetooth LE service exposing a single GATT
characteristic, which can be assigned a value to transmit a message, and which
emits a value change notification for each recieved message.

The specific names and UUIDs used to identify Lovense Bluetooth LE services and
characteristics can vary across models and firmware versions, but they seem to
confirm to the following rules:

- Device names must consist of the following:
  - the prefix `LVS-`, followed by
  - a name or identifier indicating a Lovense model line, followed by
  - a two digit suffix indicating the firmware version
- Service UUIDs must match one of the following:
  - `0000fff0-0000-1000-8000-00805f9b34fb` (first generation)
  - `6e400001-b5a3-f393-e0a9-e50e24dcca9e` (second generation)
  - `XY300001-002Z-4bd4-bbd5-a6920e4c5653`, where
    - `X` is the digit `4` or `5`
    - `Y` is any digit (`0` to `F`)
    - `Z` is the digit `3` or `4`
- Characteristic UUIDs may have any value.

If you are using a Bluetooth API such as WebBluetooth which requires all service
UUIDs to be declared upfront, you may need to generate the full list of possible
permutation of these rules. The service will only declare a single
characteristic, which your Bluetooth API should let you access without knowing
the UUID ahead of time. Once connected, you may want to start by using the
[`DeviceType`] command to confirm that the device you've connected to is the one
you expect.

## Protocol

Commands for Lovense toys follows these rules:

- Commands and replies are strings, using semicolons to mark their end.
- All commands start with a command identifier word, then possibly either
  specifiers or levels, delimited by colons. e.g. "Vibrate:5;" would set
  vibration to 5.
- Replies are in the context of the command (i.e. sending "Battery;" will just
  return a number, like "85;"), but can still be colon delimited lists.
- Commands that do not return a context specific value will return "OK;" on
  success, "ERR;" on error.

## Commands

### Support Matrix

| Command                   | [Ambi] | [Diamo] | [Domi] | [Edge] | [Ferri] | [Hush] | [Lush] | [Max] | [Nora] | [Osci] | [Quake] |
| :------------------------ | :----: | :-----: | :----: | :----: | :-----: | :----: | :----: | :---: | :----: | :----: | :-----: |
| [`Air:In:𝛘`]              |   ❌    |    ❔    |   ❌    |   ❌    |    ❔    |   ❌    |   ❌    |  ✔️   |   ❌    |   ❌    |    ❌    |
| [`Air:Level:𝛘`]           |   ❌    |    ❔    |   ❌    |   ❌    |    ❔    |   ❌    |   ❌    |  ✔️   |   ❌    |   ❌    |    ❌    |
| [`Air:Out:𝛘`]             |   ❌    |    ❔    |   ❌    |   ❌    |    ❔    |   ❌    |   ❌    |  ✔️   |   ❌    |   ❌    |    ❌    |
| [`ALight:𝛘`]              |   ❔    |    ❔    |   ❔    |   ❔    |    ❔    |   ❔    |   ❔    |   ❔   |   ❔    |   ❔    |    ❔    |
| [`AutoSwith:𝛘:𝛄`]         |   ❔    |    ❔    |   ❔    |   ❔    |    ❔    |   ❔    |   ❔    |   ❔   |   ❔    |   ❔    |    ❔    |
| [`Battery`]               |   ✔️   |    ❔    |   ✔️   |   ✔️   |    ❔    |   ✔️   |   ✔️   |  ✔️   |   ✔️   |   ✔️   |   ✔️    |
| [`DeviceType`]            |   ✔️   |    ❔    |   ✔️   |   ✔️   |    ❔    |   ✔️   |   ✔️   |  ✔️   |   ✔️   |   ✔️   |   ✔️    |
| [`GetALight`]             |   ❔    |    ❔    |   ❔    |   ❔    |    ❔    |   ❔    |   ❔    |   ❔   |   ❔    |   ❔    |    ❔    |
| [`GetAS`]                 |   ❔    |    ❔    |   ❔    |   ❔    |    ❔    |   ❔    |   ❔    |   ❔   |   ❔    |   ❔    |    ❔    |
| [`GetBatch`]              |   ✔️   |    ❔    |   ✔️   |   ✔️   |    ❔    |   ✔️   |   ✔️   |  ✔️   |   ✔️   |   ✔️   |   ✔️    |
| [`GetLevel`]              |   ❔    |    ❔    |   ❔    |   ❔    |    ❔    |   ❔    |   ❔    |   ❔   |   ❔    |   ❔    |    ❔    |
| [`GetLight`]              |   ❔    |    ❔    |   ❔    |   ❔    |    ❔    |   ❔    |   ❔    |   ❔   |   ❔    |   ❔    |    ❔    |
| [`GetPatten:𝛘`]           |   ❔    |    ❔    |   ❔    |   ❔    |    ❔    |   ❔    |   ❔    |   ❔   |   ❔    |   ❔    |    ❔    |
| [`GetPatten`]             |   ❔    |    ❔    |   ❔    |   ❔    |    ❔    |   ❔    |   ❔    |   ❔   |   ❔    |   ❔    |    ❔    |
| [`Light:𝛘`]               |   ❔    |    ❔    |   ❔    |   ❔    |    ❔    |   ❔    |   ❔    |   ❔   |   ❔    |   ❔    |    ❔    |
| [`PowerOff`]              |   ✔️   |    ❔    |   ✔️   |   ✔️   |    ❔    |   ✔️   |   ✔️   |  ✔️   |   ✔️   |   ✔️   |    ❔    |
| [`Preset:𝛘`]              |   ❔    |    ❔    |   ❔    |   ❔    |    ❔    |   ❔    |   ❔    |   ❔   |   ❔    |   ❔    |    ❔    |
| [`Rotate:𝛘`]              |   ❌    |    ❔    |   ❌    |   ❌    |    ❔    |   ❌    |   ❌    |   ❌   |   ✔️   |   ❌    |    ❌    |
| [`RotateAntiClockwise:𝛘`] |   ❌    |    ❌    |   ❌    |   ❌    |    ❌    |   ❌    |   ✔️   |   ❌   |   ❌    |        |         |
| [`RotateChange`]          |   ❌    |    ❔    |   ❌    |   ❌    |    ❔    |   ❌    |   ❌    |   ❌   |   ✔️   |   ❌    |    ❌    |
| [`RotateClockwise:𝛘`]     |   ❌    |    ❌    |   ❌    |   ❌    |    ❌    |   ❌    |   ✔️   |   ❌   |   ❌    |        |         |
| [`SetLevel`]              |   ❔    |    ❔    |   ❔    |   ❔    |    ❔    |   ❔    |   ❔    |   ❔   |   ❔    |   ❔    |    ❔    |
| [`StartMove:𝛘`]           |   ❌    |    ❔    |   ❌    |   ❌    |    ❔    |   ❌    |   ❌    |  ✔️   |   ✔️   |   ❌    |    ❌    |
| [`Status:𝛘`]              |   ❔    |    ❔    |   ❔    |   ❔    |    ❔    |   ❔    |   ❔    |   ❔   |   ❔    |   ❔    |    ❌    |
| [`StopMove:𝛘`]            |   ❌    |    ❔    |   ❌    |   ❌    |    ❔    |   ❌    |   ❌    |  ✔️   |   ✔️   |   ❌    |    ❌    |
| [`Vibrate:𝛘`]             |  All   |    ❔    |   ❔    |   ❔    |    ❔    |   ❔    |   ❔    |   ❔   |   ❔    |   ❔    |    ❔    |
| [`Vibrate𝛘:𝛄`]            |   ❔    |    ❔    |   ❔    |   ✔️   |    ❔    |   ❔    |   ❔    |   ❔   |   ❔    |   ❔    |   ✔️    |

[`Air:In:𝛘`]: #AirIn𝛘
[`Air:Level:𝛘`]: #AirLevel𝛘
[`Air:Out:𝛘`]: #AirOut𝛘
[`ALight:𝛘`]: #ALight𝛘
[`AutoSwith:𝛘:𝛄`]: #AutoSwith𝛘𝛄
[`Battery`]: #Battery
[`DeviceType`]: #DeviceType
[`GetALight`]: #GetALight
[`GetAS`]: #GetAS
[`GetBatch`]: #GetBatch
[`GetLevel`]: #GetLevel
[`GetLight`]: #GetLight
[`GetPatten:𝛘`]: #GetPatten𝛘
[`GetPatten`]: #GetPatten
[`Light:𝛘`]: #Light𝛘
[`PowerOff`]: #PowerOff
[`Preset:𝛘`]: #Preset𝛘
[`Rotate:𝛘`]: #Rotate𝛘
[`RotateAntiClockwise:𝛘`]: #RotateAntiClockwise𝛘
[`RotateChange`]: #RotateChange
[`RotateClockwise:𝛘`]: #RotateClockwise𝛘
[`SetLevel`]: #SetLevel
[`StartMove:𝛘`]: #StartMove:𝛘
[`Status:𝛘`]: #Status:𝛘
[`StopMove:𝛘`]: #StopMove:𝛘
[`Vibrate:𝛘`]: #Vibrate:𝛘
[`Vibrate𝛘:𝛄`]: #Vibrate:𝛘:𝛄
[Ambi]: https://www.lovense.com/mini-bullet-vibrator-for-clitoral-simulation
[Diamo]: https://www.lovense.com/bluetooth-vibrating-cock-ring
[Domi]: https://www.lovense.com/super-powerful-wand-massager
[Edge]: https://www.lovense.com/adjustable-prostate-massager
[Ferri]: https://www.lovense.com/magnetic-panty-vibrator
[Hush]: https://www.lovense.com/vibrating-butt-plug
[Lush]: https://www.lovense.com/bluetooth-remote-control-vibrator
[Max]: https://www.lovense.com/male-masturbators
[Nora]: https://www.lovense.com/rabbit-vibrator
[Osci]: https://www.lovense.com/oscillating-gspot-vibrator
[Quake]: https://www.lovense.com/adjustable-dual-clit-gspot-vibrator

### `Air:In:𝛘`

### `Air:Level:𝛘`

### `Air:Out:𝛘`

### `ALight:𝛘`

### `AutoSwith:𝛘:𝛄`

### `Battery`

Returns the battery level of the toy as an integer percentage from 0-100.

Some toys will also prepend an `s` character (such as `s99`) to indicate when
they are active (i.e. when a vibrator motor is turned on).

#### Example

```
Battery;

  85;
```

Denotes 85% battery remaining.

### `DeviceType`

Returns the toy's model identifier, firmware version, and Bluetooth MAC address,
as a colon-delimited list.

| Model | Identifier |
| :---- | :--------: |
| Ambi  |    `L`     |
| Diamo |     ❔      |
| Domi  |    `W`     |
| Edge  |    `P`     |
| Ferri |     ❔      |
| Hush  |    `Z`     |
| Lush  |    `S`     |
| Max   |    `B`     |
| Nora  | `A` or `C` |
| Osci  |    `O`     |
| Quake |    `J`     |

#### Example

```
DeviceType;

  C:11:0082059AD3BD;
```

Denotes a "Nora" toy model, running firmware version 1.1, with a Bluetooth
address of `00:82:05:9A:D3:BD`.

### `GetALight`

### `GetAS`

### `GetBatch`

### `GetLevel`

### `GetLight`

### `GetPatten:𝛘`

### `GetPatten`

### `Light:𝛘`

### `PowerOff`

### `Preset:𝛘`

### `Rotate:𝛘`

### `RotateAntiClockwise:𝛘`

### `RotateChange`

### `RotateClockwise:𝛘`

### `SetLevel`

### `StartMove:𝛘`

### `Status:𝛘`

### `StopMove:𝛘`

### `Vibrate:𝛘`

### `Vibrate𝛘:𝛄`

#### Turn Off Power

Turns off power to the toy.

_Availability:_ All toys

_Command Format_

```
PowerOff;
```

_Return Example_

```
OK;
```

#### Device Status

Retreive the status of the toy.

_Availability:_ Most toys (confirmed unsupported: Quake)

_Command Format_

```
Status:1;
```

_Return Example_

```
2;
```

_Status Codes:_

- 2: Normal

#### Set Vibration Speed

Changes the vibration speed for the toy. Takes integer values from 0-20.

_Availability:_ All toys

_Command Format_

```
Vibrate:10;
```

Sets vibration speed to 10 (50%).

_Return Example_

```
OK;
```

#### Set Specific Vibrator's Speed

Changes the vibration speed for a specific vibrator in the toy. Takes a vibrator
index `1` or `2` and an integer vibration speed from 0-20.

_Availability:_ Edge, Quake

_Command Format_

```
Vibrate1:5;
```

Sets the speed of vibrator 1 to 5 (25%).

_Return Example_

```
Vibrate1:5;
```

#### Configure Toy Settings

There are settings configurable through the Lovense Remote application which
have read and write commands.

##### AutoSwith

Configures options labelled as follows in Lovense Remote:

- Turn off the toy when there is an accidental Bluetooth disconnection
- Toy will go to last level when it reconnects

Note both options are marked Beta. "Turn off" appears to mean disable vibration.

The read command returns 0 or 1; the write command accepts "Off" and "On".

_Availability:_ All toys? Confirmed: Domi, Hush, Lush 2

_Command Format_

```
GetAS;
```

Read "AutoSwith" options.

_Return Example_

```
AutoSwith:0:1;
```

Indicates "turn off on disconnect" disabled; "last level on reconnect" enabled.

_Command Format_

```
AutoSwith:On:Off;
```

Set AutoSwith features to On and Off, respectively.

_Return Example_

```
OK;
```

##### Light

Labelled "Enable/Disable LED" in Lovense Remote. Not shown in Lovense Remote for
Domi.

Controls power/connection LED.

The read command returns 0 or 1; the write command accepts "off" and "on".

_Availability:_ All toys? Confirmed: Domi, Hush, Lush 2

_Command Format_

```
GetLight;
```

Read Light setting.

_Return Example_

```
Light:1;
```

LED enabled

_Command Format_

```
Light:off;
```

Disable power/connection LED

_Return Example_

```
OK;
```

##### ALight

Labelled "Enable/Disable Lights" in Lovense Remote.

_Availability:_ Domi

Controls ring of white LEDs on Domi.

The read command returns 0 or 1; the write command accepts "Off" and "On".

_Command Format_

```
GetAlight;
```

Read ALight setting.

_Return Example_

```
Alight:1;
```

Lights enabled

_Important:_ note different capitalization between read and write commands

_Command Format_

```
ALight:Off;
```

Disable lights or power LED

_Return Example_

```
OK;
```

#### Preset Levels

The Domi allows customization of the low, medium, and high levels selectable
using the hardware buttons. The raw levels are the same integer values used for
the Vibrate command.

_Availability:_ Domi, Quake

_Command Format_

```
GetLevel;
```

Fetch configured levels.

_Return Example_

```
1,9,20;
```

Vibration levels. Indicates 1 for low, 9 for medium, 20 for high. This is the
factory default.

_Command Format_

```
SetLevel:3:16;
```

Set High to level 16. 1, 2, and 3 as the first argument refer to low, medium,
and high, respectively.

_Return Example_

```
OK;
```

#### Start Accelerometer Data Stream

Starts a stream of accelerometer data. Will send constantly until stop command
is sent. Incoming accelerometer data starts with the letter G, followed by 3
16-bit little-endian numbers.

_Availability:_ Max, Nora

_Command Format_

```
StartMove:1;
```

_Return Example_

```
GEF008312ED00;
```

Denotes [0x00EF, 0x1283, 0x00ED] accelerometer readings.

#### Stop Accelerometer Data Stream

Stops stream of accelerometer data.

_Availability:_ Max, Nora

_Command Format_

```
StopMove:1;
```

_Return Example_

```
OK;
```

#### Set Rotation Speed and Direction

Sets the rotation speed and direction at once. Takes integers values from 0-20.

_Availability_: Nora

```
RotateClockwise:10;
```

Sets rotation direction to clockwise and speed to 10 (50%).

_Return Example_

```
OK;
```

```
RotateAntiClockwise:2;
```

Sets rotation direction to anti-clockwise and speed to 2 (10%).

_Return Example_

```
OK;
```

#### Toggle Rotation Direction

Toggles the direction of rotation for the toy.

_Availability:_ Nora

_Command Format_

```
RotateChange;
```

_Return Example_

```
OK;
```

#### Set rotation speed

Changes the rotation speed of the Nora toy. Takes integer values from 0-20.

_Availability_: Nora

_Command Format_

```
Rotate:10;
```

Sets rotation speed to 10 (50%).

_Return Example_

```
OK;
```

#### Set Absolute Air Level

Changes the inflation level of the Max toy. Takes integer values from 0-5.

_Availability:_ Max

_Command Format_

```
Air:Level:3;
```

Sets air level to 3 (60%).

_Return Example_

```
OK;
```

#### Set Relative Inflation Level

Inflates relative to current level, i.e. if currently inflation level is 3, and
"Air:In:1;" is sent, will inflate to 4.

_Availability:_ Max

_Command Format_

```
Air:In:1;
```

Sets air level to 1 level more inflated than it was.

_Return Example_

```
OK;
```

#### Set Relative Deflation Level

Deflates relative to current level, i.e. if currently inflation level is 3, and
"Air:Out:1;" is sent, will deflate to 2.

_Availability:_ Max

_Command Format_

```
Air:Out:1;
```

Sets air level to 1 level deflated than it was.

_Return Example_

```
OK;
```

#### Get Production Batch Number

Returns the production batch number for this device. This digits appear to
correspond to a `YYMMDD` date during manufacture.

_Availability:_ All toys? Confirmed: Lush 2, Hush, Domi, Quake.

_Command Format_

```
GetBatch;
```

_Return Example_

```
190124;
```

#### Count Programmed Patterns

List the indexes of the patterns that are currently programmed into the device.
The maximum number of patterns in 10, so each index will always be a single
digit.

_Availability:_ Lush 2, Domi

_Command Format_

```
GetPatten;
```

_Return Example_

```
P:01234;
```

This return tells us that there are currently five patterns programmed on the
device, with indicies 0 through 4.

#### View Programmed Pattern

Returns a pattern that is currently programmed into the device.

Each pattern is represented as a series of digits from 0 to 9, each indicating
the vibration level for one half-second of the pattern. (This is different than
the 0-20 scale used by the Vibrate command.)

The response is split into multiple messages, each containing up to 12 digits (6
seconds). Each response has a prefix indicating the pattern index, the number of
parts in the response, and the index of the response.

For the Domi, the pattern length must be between 5 and 50 seconds, so the
response will use a maximum of 9 messages, so the part count and indicies will
always be a single digit.

For the Lush 2, the part count and indicies are padded to always use two digits.

_Availability:_ Lush 2, Domi, Quake

_Command Format_

```
GetPatten:4;
```

Domi response using one-digit part indices:

_Return Example_

```
P4:1/5:000042003720;
P4:2/5:000002436658;
P4:3/5:997339993001;
P4:4/5:291111115111;
P4:5/5:1110000000;
```

Lush 2 response using two-digit part indices:

_Return Example_

```
P4:01/01:346797643;
```

#### Run Programmed Patern

Starts running a programmed pattern on a loop. Takes an positive integer pattern
index to start running it, or 0 to stop running the pattern.

While Domi is able to take any pattern index, from 0 to 10, Lush 2 and Quake
only seem to be able to take indices from 0 to 4. Other toys have not been
tested.

_Availability:_ Lush, Hush, Ambi, Domi, Edge, Osci, Quake

_Command Format_

```
Preset:8;
```

_Return Example_

```
OK;
```
