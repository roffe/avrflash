# avrflash

Minimal Go library for flashing an ATmega328P/PB over its serial bootloader.
No avrdude, no CGO. Built to push firmware to an Arduino Nano from
[bdmtool](https://github.com/roffe/bdmtool).

Supports both bootloaders you are likely to meet on a Nano:

| Bootloader | Protocol | Notes |
|---|---|---|
| optiboot (stock Arduino) | STK500v1 | Signature check, page writes |
| urboot (MiniCore default) | urprotocol (`avrdude -c urclock`) | Vector-bootloader patching, chip erase if the bootloader has it, flash only (no metadata) |

The bootloader type is detected automatically from the sync reply.

## Install

```
go get github.com/roffe/avrflash
```

## Usage

```go
import "github.com/roffe/avrflash"

hexData, _ := os.ReadFile("firmware.hex") // Intel HEX
err := avrflash.Update("/dev/ttyUSB0", 115200, hexData, func(format string, v ...interface{}) {
	log.Printf(format, v...)
})
```

`Update` opens the port, pulses DTR/RTS for the Arduino auto-reset, syncs
with the bootloader, writes the image page by page and reports progress
through the callback. Nothing else may have the port open.

The baud rate must match the bootloader, not the sketch. Common values:

| Board / bootloader | Baud |
|---|---|
| Nano, optiboot | 115200 |
| Nano, old bootloader | 57600 |
| MiniCore urboot | whatever you picked in the board menu, usually 115200 |

## Limitations

- ATmega328P and ATmega328PB only (signature / MCU id is checked).
- Flash only. No EEPROM, no fuses, no verify readback.
- Intel HEX input only, 16-bit addresses (no extended address records).
- Requires a bootloader with auto-reset wiring (DTR to RESET cap).

## Test

```
go test ./...
```

The tests cover the HEX parser, urboot ack decoding and the vector patching.
Flashing itself needs real hardware.
