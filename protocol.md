# Packet format

```
0xfe - start byte
0xea - some kind of protocol version
0x10 - some kind of protocol version
1 byte of packet length, payload length + 4
variable length payload
```

Payload is minimum of 1 byte, starting with command.

# Known commands

- 0x11, 8 bytes from C2442a - send alarm
- 0x12, height, weight, age, gender - send user info
- 0x14, hand - send dominant hand
- 0x15, null terminated list of bytes - send display device function
- 0x16, 4 bytes goal - send goal step
- 0x17, byte - send time system
- 0x18, bool - send quick view
- 0x19, byte - send display watchface
- 0x1a, byte - send metric system
- 0x1b, byte - send device language
- 0x1c, bool - send other message
- 0x1d, bool - send sedentary reminder
- 0x1e, byte - send device version
- 0x1f, 1 unknown byte - timing measure heart rate
- 0x21 - query all alarms
- 0x24 - query dominant hand
- 0x25 - query device function
- 0x25, 0xff - query device function support
- 0x26 - query goal step
- 0x27 - query time system
- 0x28 - query quick view
- 0x29 - query watchface
- 0x2a - query metric system
- 0x2b - query device language
- 0x2c - query other message
- 0x2d - query sedentary reminder
- 0x2e - query device version
- 0x2f - query timing measure heart rate
- 0x31, 4 bytes timestamp, 0x08 - sync time
- 0x33, byte - sync past step
- 0x32 - sync sleep
- 0x33, byte - sync past sleep
- 0x34 - query last dynamic rate
- 0x35, 1 unknown byte - query heart rate
- 0x36, 1 unknown byte - query heart rate
- 0x37- query movement heart rate
- 0x38, 37 bytes from C2438E - send watchface layout
- 0x39 - query watchface layout
- 0x3a - query sleep action
- 0x41, 0xff - call off hook
- 0x41, length, utf8 str - send message
- 0x42, 21 bytes from C2439F - send future weather
- 0x43, variable length from C2439F - send today weather
- 0x51, 0xff - shutdown
- 0x52 - send calibrate gsensor
- 0x54, byte - send step length
- 0x59, byte - query steps category
- 0x61 - find device
- 0x63 - start firmware upgrade?
- 0x63, 0x01 - switch to hs dfu
- 0x63, 0x00 - query hs dfu address
- 0x63, uint32 size - firmware file transfer
- 0x63, 0xff, 0xff, 0xff, 0xff - abort firmware upgrade
- 0x66 - switch camera view
- 0x68 - unknown
- 0x69, 0x00, 0x00, 0x00 - start measure blood pressure
- 0x69, 0xff, 0xff, 0xff - stop measure blood pressure
- 0x6b, 0x00 - start measure blood oxygen
- 0x6b, 0xff - stop measure blood oxygen
- 0x6c - ui file transfer
- 0x6d - unknown
- 0x6e, uint32 - start watchface upload?
- 0x6f, cmd - ecg heart rate command
- 0x71, hour from, min from, hour to, min to - send dnt
- 0x72, hour from, min from, hour to, min to - send quick view time
- 0x73, period, steps, start hour, end hour - send reminder to move
- 0x74, 0x00, 0x00, 0x00, 0x00 - watchface file check ok
- 0x74, 0xff, 0xff, 0xff, 0xff - watchface file check fail
- 0x74, uint32 size - start watchface file transfer
- 0x75, 14 bytes - set physiological period reminder
- 0x78, on - send breathing ligh
- 0x81 - query dnd time
- 0x82 - query quick view time
- 0x83 - query reminder to move
- 0x84 - query support watchface
- 0x85 - query physiological period
- 0x88 - query breathing light

# Services

```
0000fee2-0000-1000-8000-00805f9b34fb - commands
0000fee6-0000-1000-8000-00805f9b34fb - file transfer
```

# Data transfer
fe ea 10 09 74 00 00 4b f6 - start watchface transfer, 0x4bf6 file size

fe 74 97 00 81 05 14 1a 00 00 00 00 f0 18 00 40 35 
6b 17 22 0a 41 4e 6b 17 22 0a - crc16 and data blocks, of 256 bytes

# Packets (example from ble sniffer)

```
<- c0 1d 00 80 17 00 a5 01 00
?

<- 44 46 55 3d 30
? (large stream? timeout? mtu detection?)

<- 4d 4f 59 4f 55 4e 47
MOYOUNG

-> fe ea 10 0a 31 5d 98 9a 94 08 (0000fee2-0000-1000-8000-00805f9b34fb) - sync time
<- 4d 4f 59 2d 41 45 33 2d 31 2e 38 2e 31

-> fe ea 10 09 12 aa 41 14 00 - set user info, 170 cm, 65 kg, 20 years, male
-> fe ea 10 09 16 00 00 1f 40 - set goal step, 8000
-> fe ea 10 06 1a 00 - set metric system
-> fe ea 10 06 17 01 - set time system
-> fe ea 10 05 2f - query heart rate?

-> fe ea 10 05 2b - query device language
<- fe ea 10 06 2f 06
<- fe ea 10 0a 2b 00 00 90 1f ff
parsed language mask - 111111111111100000001001

-> fe ea 10 05 39

-> fe ea 10 06 25 ff - query device function support
<- fe ea 10 2a 39 00 00 01 ff ff 00 00 00 00 00 00 00 00 00 00
<- 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
<- 00 00

-> fe ea 10 05 85 - query physiological period
<- c0 1d 00 80 17 00 a5 01 00

-> fe ea 10 05 32 - sync sleep
-> fe ea 10 05 34 - query last dynamic rate
<- fe ea 10 05 32
```

# All outgoing messages logged

```
fe ea 10 0a 31 5d 98 9a 94 08 - sync time
fe ea 10 09 12 aa 41 14 00 - set user info
fe ea 10 09 16 00 00 1f 40 - set step goal
fe ea 10 06 1a 00 - set metric system
fe ea 10 06 17 01 - set time system
fe ea 10 05 2f - query heart rate
fe ea 10 05 2b - query device language
fe ea 10 05 39 - query watchface layout
fe ea 10 06 25 ff - query device function support
fe ea 10 05 85 - query physiological period
fe ea 10 05 32 - sync sleep
fe ea 10 05 34 - query dynamic rate
fe ea 10 06 35 04 - query heart rate
fe ea 10 06 59 00 - query steps
fe ea 10 06 59 02 - query steps
fe ea 10 06 1e 01 - send device version
fe ea 10 06 1b 00 - send device language
fe ea 10 06 54 4f - send step length
fe ea 10 20 43 00 00 04 00 20 00 20 00 20 00 20 00 6e 00 75 00 6c 00 6c 00 4d 00 6 - send today weather
fe ea 10 1a 42 00 01 08 00 00 06 00 fd 05 00 ff 06 03 02 0a 03 05 0b 00 04 0b - send future weather
fe ea 10 06 35 05 - query heart rate
fe ea 10 06 35 06 - query heart rate
fe ea 10 06 35 07 - query heart rate
fe ea 10 05 29 - query watchface
fe ea 10 05 84 - query support watchface
fe ea 10 05 29 - query watchface
fe ea 10 05 84 - query support watchface
fe ea 10 05 29 - query watchface
fe ea 10 05 84 - query support watchface
fe ea 10 05 29 - query watchface
fe ea 10 05 84 - query support watchface
fe ea 10 06 19 03 - send display watchface
fe ea 10 06 19 04 - send display watchface
fe ea 10 06 19 03 - send display watchface
fe ea 10 06 19 02 - send display watchface
fe ea 10 06 19 01 - send display watchface
fe ea 10 06 19 04 - send display watchface
fe ea 10 09 74 00 00 4b f6 - start watchface transfer, 0x4bf6 file size
fe ea 10 0a 31 5d 98 9b 41 08 - sync time
fe ea 10 06 1e 01 - send device version
fe ea 10 06 1b 00 - send device language
fe ea 10 06 54 4f - send step length
fe 74 97 00 81 05 14 1a 00 00 00 00 f0 18 00 40 35 6b 17 22 0a 41 4e 6b 17 22 0a 4... - 256 bytes + 4 bytes header
fe d1 07 00 d0 39 00 00 13 3b 00 00 e0 3c 00 00 19 3f 00 00 8f 40 00 00 1c 43 00 0
fe 33 c6 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 0
fe 33 c6 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 0
fe f5 2c 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 0
fe 33 c6 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 0
fe cb 64 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 0
fe c8 68 00 00 ff 00 00 ff 00 00 ff 00 00 ff 00 00 ff 00 00 ff 00 00 ff 00 00 ff 0
fe 92 99 00 be 02 ff df 02 8c 51 01 00 00 36 08 41 01 ce 79 01 ff ff 01 f7 be 02 f
fe 39 76 00 01 ce 79 01 00 00 2f 8c 51 01 ff df 01 f7 be 02 f7 9e 01 ef 7d 02 de f
fe 0c 2f 00 ef 7d 01 de fb 01 ef 7d 01 f7 9e 01 f7 be 03 bd f7 01 00 00 29 00 20 0
fe 89 fb 00 9e 01 f7 be 03 ad 75 01 00 00 28 d6 ba 01 f7 be 03 f7 9e 01 e7 1c 01 e
fe 6f 09 00 01 bd f7 0b c6 18 07 c6 38 06 c6 58 01 ce 59 07 ce 38 01 c6 38 05 c6 1
fe fe 78 00 de db 04 d6 ba 01 d6 9a 11 d6 ba 08 d6 da 01 de da 01 de db 03 d6 db 0
fe 27 fc 00 45 01 ad 55 01 e7 1c 01 de fb 01 de db 01 d6 ba 01 de db 01 ff df 01 f
fe c4 c0 00 03 f7 9e 01 e7 3c 01 f7 be 09 e7 1c 01 ce 79 01 d6 9a 01 d6 ba 01 d6 9
fe d1 c1 00 f7 be 03 f7 9e 01 00 00 28 f7 9e 01 f7 be 03 f7 9e 01 e7 3c 01 f7 be 0
fe ae f2 00 9e 01 00 00 28 f7 9e 01 f7 be 03 f7 9e 01 e7 3c 01 f7 be 09 d6 9a 01 c
fe e4 4f 00 f7 be 42 ef 7d 01 10 82 02 ce 79 01 ce 59 01 c6 38 01 d6 9a 01 f7 be 0
fe 6c 46 00 82 02 ce 79 01 ce 59 02 d6 9a 01 f7 be 0a e7 3c 01 f7 9e 01 f7 be 03 f
fe 02 e8 00 01 f7 be 09 de fb 01 de db 03 10 82 02 ef 7d 01 f7 be 42 ef 7d 01 10 8
fe b3 4c 00 f7 be 42 ef 7d 01 10 82 02 de fb 04 f7 be 09 e7 1c 03 e7 3c 01 10 82 0
fe 0d f9 00 be 0a 10 a2 15 f7 be 09 e7 1c 01 94 b2 01 5a cb 01 31 86 01 21 04 01 1
fe c0 82 00 02 ef 7d 01 f7 be 0e de fb 01 42 08 01 10 a2 0a f7 be 0c 84 30 01 18 c
fe 2b bb 00 e7 3c 01 f7 9e 01 f7 be 03 f7 9e 01 00 00 28 f7 9e 01 f7 be 03 f7 9e 0
fe 57 9b 00 82 01 08 61 01 08 62 02 ce 79 01 f7 be 08 18 c3 15 f7 be 04 4a 69 01 1
fe ff 3c 00 02 73 ae 01 f7 be 09 18 c3 0f f7 be 09 63 2c 01 18 c3 07 31 86 01 ad 5
fe ea 1d 00 e7 1c 01 de fb 02 f7 be 09 e7 1c 03 e7 3c 01 10 82 01 08 41 01 4a 8a 0
fe 48 a1 00 ae 01 73 af 02 08 41 01 10 82 01 e7 3c 01 e7 1c 01 de fb 02 f7 be 0a e
fe 10 27 00 02 73 ae 01 f7 be 09 21 04 02 4a 49 01 de fb 01 f7 be 03 21 04 08 f7 b
fe d6 30 00 e7 3c 01 e7 1c 01 de fb 02 f7 be 09 e7 1c 01 df 1c 01 e7 1c 01 e7 3c 0
fe ef 8b 00 05 63 2c 01 21 24 07 63 0c 01 f7 be 07 73 8e 01 c6 38 02 08 41 01 10 8
fe 5a c8 00 e7 3c 01 f7 be 09 e7 1c 03 e7 3c 01 10 82 01 08 41 01 a5 14 02 52 8a 0
fe 40 ea 00 3c 01 e7 1c 01 de fb 02 f7 be 0a e7 3c 01 f7 9e 01 f7 be 03 f7 9e 01 0
fe 07 7b 00 07 e7 1c 01 ff ff 05 a5 34 01 5a cb 10 d6 ba 01 ff ff 09 73 ae 01 52 a
fe 09 61 00 f7 9e 01 f7 be 03 f7 9e 01 e7 3c 01 f7 be 09 e7 1c 03 e7 3c 01 10 82 0
fe 2e 0e 00 82 01 e7 3c 01 e7 1c 01 de fb 02 f7 be 09 e7 1c 03 e7 3c 01 10 82 01 0
fe 7f 94 00 01 00 00 28 f7 9e 01 f7 be 03 f7 9e 01 e7 3c 01 f7 be 09 e7 1c 03 e7 3
fe d6 9f 00 e7 3c 01 e7 1c 01 de fb 02 f7 be 09 e7 1c 03 e7 3c 01 10 82 01 08 41 0
fe ee 2d 00 41 01 10 82 01 e7 3c 01 e7 1c 01 de fb 02 f7 be 0a e7 3c 01 f7 9e 01 f
fe 6a f8 00 09 39 c7 16 f7 be 07 73 ae 01 52 cb 02 08 41 01 10 82 01 e7 3c 01 e7 1
fe 53 cd 00 e7 3c 01 f7 9e 01 f7 be 03 f7 9e 01 00 00 28 f7 9e 01 f7 be 03 f7 9e 0
fe 02 8d 00 61 01 8c 71 02 ce 79 01 f7 be 09 a5 14 01 39 e7 10 42 08 01 bd f7 01 f
fe 8d 31 00 09 e7 1c 03 e7 3c 01 10 82 02 f7 be 0e de db 01 84 10 01 42 08 01 39 e
fe 3e dd 00 10 82 02 e7 1c 02 de fb 02 f7 be 09 e7 1b 01 e6 fb 01 e7 1c 01 e7 3c 0
fe 2e 3e 00 01 de fb 03 f7 be 09 de fb 02 e6 fc 01 e7 1c 01 10 82 02 f7 be 44 10 8
fe 08 43 00 e7 1c 01 10 82 02 f7 be 44 10 82 02 e7 1c 01 de fb 03 f7 be 0a e7 3c 0
fe 46 50 00 be 09 de fb 02 e7 1c 01 e7 3c 01 10 82 02 ef 5d 01 f7 be 42 ef 5d 01 1
fe cb 0f 00 01 de db 01 21 24 01 10 82 02 e7 3c 01 e7 1c 01 de fb 02 f7 be 0a e7 3
fe f2 e5 00 e7 1c 02 f7 be 09 e7 1c 02 e7 3c 01 ef 5d 01 4a 29 01 10 82 01 00 20 0
fe 74 39 00 9e 01 f7 be 03 f7 9e 01 00 00 28 f7 9e 01 f7 be 03 f7 9e 01 e7 3c 01 f
fe 0d d1 00 01 00 00 01 00 20 01 08 61 01 10 82 02 b5 96 01 ef 7d 01 ef 5d 02 e7 3
fe 93 b5 00 e7 3c 01 84 10 01 4a 49 01 18 c3 01 10 82 3a 18 c3 01 4a 49 01 84 10 0
fe 4d c0 00 be 0f ef 7d 01 ef 5d 02 ef 7d 01 f7 9e 03 f7 be 3c f7 9e 03 ef 7d 01 e
fe 84 18 00 01 ef 5d 01 f7 9e 01 f7 be b4 f7 9e 01 ef 5d 01 f7 9e 01 f7 be 03 f7 9
fe eb 79 00 18 01 00 00 30 08 61 01 d6 ba 01 ef 7d 02 f7 9e 01 f7 be 02 ff df 02 e
fe 84 97 00 01 ef 7d 02 ef 5d a6 ef 7d 02 f7 9e 01 ef 7d 01 b5 b6 01 31 86 01 00 0
fe 72 6b 00 e7 1c 01 9c d3 01 63 0c 01 31 a6 01 18 e3 01 10 a2 01 18 e3 01 31 86 0
fe 55 77 00 ff 07 42 28 08 00 00 2e 5a cb 08 ff ff 07 5a cb 08 29 65 08 f7 be 07 2
fe c4 9a 00 06 08 21 f7 be 07 f7 9e 01 73 ae 01 10 a2 08 f7 be 0c ef 5d 01 5a cb 0
fe a7 b8 00 a6 08 f7 be 0f 31 a6 08 f7 be 0f 39 c7 08 f7 be 0f 39 c7 08 f7 be 0f 3
fe bf fc 00 29 45 01 18 e3 06 21 24 01 f7 be 01 bd f7 07 c6 18 01 f7 be 06 31 86 0
fe 71 fe 00 db 01 f7 be 0c 39 c7 16 f7 be 01 39 c7 16 f7 be 01 39 e7 16 f7 be 01 3
fe 3b 8a 00 18 e3 07 7b cf 01 f7 be 0e a5 14 01 18 e3 07 7b ef 01 f7 be 0e 52 aa 0
fe 79 0e 00 aa 01 39 c7 13 4a 49 01 ef 7d 01 f7 be 01 bd f7 01 39 e7 13 ad 75 01 f
fe 38 82 00 21 04 07 c6 18 01 f7 be 0d c6 18 01 21 04 07 52 aa 01 f7 be 0e 5a cb 0
fe 4b 4a 00 07 e7 3c 01 94 92 01 52 8a 01 31 86 01 21 24 02 31 86 01 4a 49 01 84 1
fe 48 35 00 ce 59 01 39 c7 15 ce 79 01 f7 be 01 5a cb 01 39 c7 13 6b 4d 01 f7 be 0
fe f8 aa 00 01 18 e3 07 52 8a 01 f7 be 0d b5 b6 01 18 e3 08 c6 18 01 f7 be 0d 4a 6
fe 25 ab 00 ce 79 01 31 a6 07 42 28 01 63 0c 01 31 a6 07 73 ae 01 f7 be 05 73 8e 0
fe 3d bf 00 07 73 ae 01 f7 be 01 21 04 07 f7 be 06 42 28 01 21 04 07 ce 79 01 f7 b
fe 76 8a 00 39 e7 07 a5 34 01 f7 be 0d ef 7d 01 4a 49 01 39 e7 06 42 28 01 ef 5d 0
fe 50 fb 00 05 d6 ba 01 18 e3 07 6b 6d 02 18 e3 07 f7 9e 01 f7 be 05 f7 9e 01 18 e
fe 51 c5 00 f7 9e 01 31 a6 08 39 e7 01 31 a6 07 d6 ba 01 f7 be 05 d6 ba 01 31 a6 0
fe a8 b9 00 07 ad 75 01 21 04 01 10 a2 0d 21 04 01 ad 75 01 f7 be 05 ad 55 01 18 c
fe 4d 37 f6 ff df 01 52 aa 01 42 28 13 52 8a 01 ff df 01 00 00 2e ff ff 03 a5 34 0
fe ea 10 09 74 00 00 00 00 - transfer ok?
fe ea 10 06 19 04 - display watchface
fe ea 10 05 29 - query watchface
fe ea 10 05 84 - query watchface support
fe ea 10 0a 31 5d 98 9b 56 08 - sync time
fe ea 10 09 12 aa 41 14 00 - set user info
fe ea 10 09 16 00 00 1f 40 - set step goal
fe ea 10 06 1a 00 - send metric system
fe ea 10 06 17 01 - send time system
fe ea 10 05 2f - not interesting
fe ea 10 05 2b - not interesting
fe ea 10 05 39 - query watchface layout
fe ea 10 06 25 ff - query device function support
fe ea 10 05 85
fe ea 10 05 32
fe ea 10 06 33 01
fe ea 10 06 33 02
fe ea 10 06 33 03
fe ea 10 06 33 04
fe ea 10 05 34
fe ea 10 06 35 00
fe ea 10 06 35 04
fe ea 10 06 59 00
fe ea 10 06 59 02
fe ea 10 06 1e 01
fe ea 10 06 1b 00
fe ea 10 06 54 4f
fe ea 10 20 43 00 00 04 00 20 00 20 00 20 00 20 00 6e 00 75 00 6c 00 6c 00 4d 00 6
fe ea 10 1a 42 00 01 08 00 00 06 00 fd 05 00 ff 06 03 02 0a 03 05 0b 00 04 0b
fe ea 10 06 35 01
fe ea 10 06 35 05
fe ea 10 06 35 02
fe ea 10 06 35 06
fe ea 10 06 35 03
fe ea 10 06 35 07
fe ea 10 05 37
```

# CRC Calculation

```js
const crc16 = (bytes) => {
  let result = 65258;
  
  for (const b of bytes) {
    let b2 = (((result & 0xff) << 8) | ((0xFF00 & result) >> 8)) ^ (b & 0xff);
    let b3 = b2 ^ ((b2 & 0xff) >> 4);
    let b4 = b3 ^ (((b3 & 0xff) << 8) << 4);
    result = b4 ^ (((b4 & 0xff) << 4) << 1);
  }
  
  return (result&0xffff).toString(16).padStart(4, '0');
}
```