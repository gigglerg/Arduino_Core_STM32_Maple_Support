# Arduino Core STM32 Maple Support
STM32dunio Arduino Core STM32 at release 1.4.0 or below did not have Maple Mini support beyond basic programming and debug via external devices like STLink or a TTL UART or other programmer.

[The original Maple Mini release for Ardino](https://wiki.stm32duino.com/index.php?title=Maple_Mini) install had DFU bootloader support and CDC serial for debugging and feedback via the Arduino IDE Monitor.  These are not planned for release until 1.5.0 so this GitHub is a temporary patch for Windows installs.

## Creation reason
Maple Mini support in Arduino IDE is great until you want to use other device peripherals that are not part of the MapleLabs headers.  In this situation you will quickly find you're stuck with including parts of STM32 HAL headers into your project or write your own.  STM32dunio Arduino Core STM32 offers the best choice for future Maple Mini development so having the original support for programming and debug are important.

## Features
* DFU support
 * Original MapleLabs
 * Bootloader 2.0
* USB CDC support.  Full speed, device only, Bulk pipe
 * Flush control options
  * Manual.  You decide when to send data to host
  * Semi-automatic.  Done for you based upon millis() timeout and write data
  * Automatic.  Least efficient and will send single characters (like original CDC implementation)
 * Maple automatic reset support (user code has to call CDC serial functions)
 * Has original MapleLabs CDC descriptors for seamless use
* Flash device support
 * F103CB, 128K (like original)
 * F103C8, 64K (like blue pill etc.)

## Known issues
* Automatic reset.  Implementation for magic character sequence handled in the Arduino CDC class CDCSerial rather than USB ISR.  For automatic reset to work, user code must invoke repeatedly CDC serial functions, like available() or flush() or read() etc.
* CDCSerial::flush will only send 1 USB BULK IN frame per mS which may, in certain situations mean the caller will have invoke more than once

## Installation
I used Cygwin but other shell options are available.  Move into the default Windows package manager installer location, like:

```bash
$cd /cygdrive/<drive>/Users/<user>/AppData/Local/Arduino15/packages/STM32/hardware/stm32/
$patch -p1 < giggler141.patch
```

The changes I made to 1.4.0 are distributed in this Github as a GNU patch.


## MD5
831e452957b94557a4633c694ffb9d95 *giggler141.patch


## Using
It is all hidden under the hood of IDE so as an example you'll recognise something like this simple script.  You will have to select Board part number (MCU), Hardware UART use, USB interface, Upload method.

```c
#define LED 29

// the setup function runs once when you press reset or power the board
void setup() {
  // initialize digital pin LED_BUILTIN as an output.
  pinMode(LED, OUTPUT);
  Serial.begin(9600); // virtual serial so no baud rate required
  while (!Serial) { // start output serial data only after connection to PC is established
    digitalWrite(LED, 1-digitalRead(LED));
    delay(100);
  }

  Serial.println("Testing");
  Serial.println("Done");
}

// the loop function runs over and over again forever
void loop() {
  uint32_t av = Serial.available();
  if (av) {
      int b = Serial.read();
      if (b != -1) {
        digitalWrite(LED, 1-digitalRead(LED));
        Serial.write(b);
      }
  }
}
```

## Useful information
Set macro **CDC_DISCONNECT_PIN** with the Arduino pin number of your boards software USB disconnect i/o (-1 no circuit.  Maple Mini default is 34).

Set macro **CDC_FLUSH_TYPE** to 0, 1, 2, 4 (enum eFLUSH_xxxx) from CDCSerial (default CDCSerial::eFLUSH_AUTOFORCE, 2).

```c
typedef enum {
	eFLUSH_MANUAL		= 0x0,	// caller does everything
	eFLUSH_AUTOTIMED	= 0x1,	// same as auto + if next write time>1mS then flush after write.  also flush on reads
	eFLUSH_AUTOFORCE	= 0x2,	// when data send it
	eFLUSH_AUTO			= 0x4,	// when USB packet worth of data, it is sent
}eFLUSH;
```

Set macro **CDC_START_DELAY1** and **CDC_START_DELAY2** in milli seconds.  These are small delays at start up within the CDCSerial constructor, tweak to suit your requirements.


## Image helpers
![MCU selection for flash size](/support/mcu.png "MCU selection for flash size")
![Hardware serial](/support/hwserial.png "Hardware serial selection")
![CDC use](/support/cdc.png "CDC Serial use")
![Bootloader](/support/bootloader.png "Bootloader choices")
