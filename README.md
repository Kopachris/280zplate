# 280zplate
Arduino interface to 280zx faceplate

## Electrical
Five analog inputs to Arduino from potentiometers:
* Volume - A0
* Balance - A1
* Fader - A2
* Bass - A3
* Treble - A4

Nine (more?) digital inputs to Arduino from face buttons:
* Presets 1-5 - D0-D4
* Up/Next - D5
* Down/Back - D6
* Seek - D7
* Mute - D8

Arduino communicates to Raspberry Pi and is powered via USB.

Power button connects in series to Raspberry Pi pins 5 and 6.

## Serial
Arduino communicates to Raspberry Pi via USB/serial. Message format is `[Start byte]`, `[Input pin]`, `[Value]`, `[Checksum mod-256]`.

### Start byte
Start byte indicates source of transmission: `0xad` for Arduino -> Pi, `0xda` for Pi -> Arduino.

### Input pin
Input pins are hex representations of the pin name: `0xa0` through `0xa4` for analog inputs, and `0xd0` through `0xd8` for digital inputs.

### Value
Values are single byte unsigned integers. Values for digital inputs are `0x00` for button off and `0x01` for button on.

### Call-response pattern and example
1. Arduino sends volume control message: `0xad 0xa0 0xff 0x4c`
2a. (CONTINGENCY) Pi does not respond within 0.05 seconds
2b. Arduino resends volume control message.
3. Pi responds with same message, different start byte and checksum: `0xda 0xa0 0xff 0x79`

## Control flow
In Arduino's main loop, read values of all pins. If new value does not match previously stored value then send message, push the awaited response to a list and store the value. If a response is already waiting for that pin, remove it and replace it with the current awaited response. Read serial for any pending responses (0.05 second timeout). Resend any messages still waiting on responses.
