# ATtiny85 Millisecond Timer Driver

Simple `Timer0`-based millisecond timebase for AVR (tested target: ATtiny85).

## What It Does

- Configures `Timer0` in CTC mode.
- Uses compare-match interrupt to increment a global millisecond counter.
- Exposes:
  - `timeInit()` to start the timer.
  - `getTime()` to read elapsed milliseconds.

## Timing Configuration

Current implementation in `timer.c` assumes:

- `F_CPU = 16000000UL`
- Timer0 prescaler: `64`
- Timer clock: `16,000,000 / 64 = 250,000 Hz`
- Compare value: `OCR0A = 249`
- Interrupt period: `250 ticks = 1 ms`

## Files

- `timer.c` - timer setup, ISR, and millisecond counter logic.
- `timer.h` - public API (`timeInit`, `getTime`).

## Public API

```c
void timeInit(void);
uint32_t getTime(void);
```

## Usage Example

```c
#include "timer.h"

int main(void)
{
    timeInit();

    uint32_t start = getTime();
    while ((getTime() - start) < 1000) {
        // wait 1 second
    }

    for (;;) {}
}
```

## Build Example (avr-gcc)

```bash
avr-gcc -mmcu=attiny85 -DF_CPU=16000000UL -Os -std=gnu11 \
  -o app.elf main.c timer.c
avr-objcopy -O ihex app.elf app.hex
```

## Flash Example (avrdude)

```bash
avrdude -c usbtiny -p t85 -U flash:w:app.hex
```

Replace programmer settings (`-c ...`) with your hardware.

## Notes

- `getTime()` briefly disables interrupts to read the 32-bit counter atomically on an 8-bit MCU.
- `timeInit()` also disables/enables global interrupts during timer setup.
- If your CPU clock is not 16 MHz, update `F_CPU` and timer compare settings accordingly.
