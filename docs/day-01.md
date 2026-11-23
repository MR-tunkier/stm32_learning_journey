# Day 1 — STM32 reference manual and first register-level GPIO blink

## Goal

Get comfortable navigating the STM32F1 documentation and understand enough of the GPIO and clock-enable registers to blink the onboard LED without using the HAL GPIO functions.

## Topics studied

- Difference between the datasheet, reference manual, errata and application notes.
- Navigating the STM32F1 reference manual (RM0008).
- High-level bus overview: AHB, APB1 and APB2.
- GPIO register roles:
  - `CRL` and `CRH`: configure pin modes and configurations.
  - `IDR`: read input pin levels.
  - `ODR`: output data register; also selects pull-up/pull-down direction for STM32F1 input pull mode.
  - `BSRR`: set or reset output pins atomically.
- Enabling a peripheral clock through `RCC->APB2ENR`.
- PC13 onboard LED is active-low on the Blue Pill: driving PC13 LOW turns it on.

## Practical result

Successfully blinked the onboard PC13 LED using CMSIS register access.

The project still used the CubeIDE-generated startup and HAL initialization infrastructure. The GPIO peripheral itself was controlled directly through registers; this was not a complete from-scratch startup implementation.

Example application code:

```c
RCC->APB2ENR |= (1U << 4);  // Enable GPIOC clock

// PC13: output push-pull, 2 MHz
GPIOC->CRH &= ~(0xFU << 20);
GPIOC->CRH |=  (0x2U << 20);

while (1)
{
    GPIOC->BSRR = (1U << 29); // Reset PC13 LOW: onboard LED ON
    for (volatile uint32_t i = 0; i < 800000; i++);

    GPIOC->BSRR = (1U << 13); // Set PC13 HIGH: onboard LED OFF
    for (volatile uint32_t i = 0; i < 800000; i++);
}
```

## Important register details learned

- GPIOC clock enable is APB2ENR bit 4.
- GPIOA clock enable is APB2ENR bit 2.
- For PC13, the configuration nibble is in `CRH`, at bits 23:20.
- `BSRR` lower half sets pins; upper half resets pins. For PC13, bit 13 sets and bit 29 resets.
- Prefer direct assignment to BSRR (`GPIOC->BSRR = ...`) because it is a write-action register.

## Difficulties and things to revisit

- The clock-tree explanation (HSI, PLL, prescalers and peripheral clocks) felt overwhelming at this stage. We deliberately parked the detailed clock-tree study until it becomes useful for timers, PWM and UART.
- Keep the distinction clear: enabling a GPIO peripheral clock is not the same thing as configuring the system clock tree.
- The AHB/APB bus overview is an initial map; revisit it when studying peripheral timing and clock configuration.

## Takeaway

I can navigate the relevant parts of RM0008, enable GPIOC, configure PC13 and control the onboard LED through registers rather than HAL GPIO calls.
