# Day 2 — GPIO input/output, buttons, pull-ups and pull-downs

## Goal

Understand how a physical button becomes a digital input to the MCU, configure GPIO pins directly, and use a button on PA0 to control an external LED on PA1.

## Hardware setup

- STM32 Blue Pill (STM32F103C6T6).
- PA0: pushbutton input.
- PA1: external LED output.
- The exercise was tested with both pull-up and pull-down wiring.

## Topics studied

- Input versus output GPIO configuration.
- STM32F1 `CRL` pin configuration fields.
- `IDR` reads the actual logic level at the input pin.
- Pull-up and pull-down resistors prevent an input from floating when the button is released.
- On STM32F1, input pull-up/pull-down mode is configured with MODE = 00 and CNF = 10 (nibble `0x8`); the corresponding ODR bit selects pull direction.
- Output push-pull configuration for an LED.
- Bit masks and bitwise AND to test an individual input bit.

## Button wiring and logic

### Pull-up

- Button connects PA0 to GND.
- Released = HIGH.
- Pressed = LOW (active-low).

To select pull-up in input pull-up/pull-down mode:

```c
GPIOA->ODR |= (1U << 0);
```

### Pull-down

- Button connects PA0 to 3.3 V.
- Released = LOW.
- Pressed = HIGH (active-high).

To select pull-down:

```c
GPIOA->ODR &= ~(1U << 0);
```

**Current physical setup for Day 3:** pull-down. PA0 is LOW when released and HIGH when pressed.

## Important code patterns

Configure PA0 as input pull-down:

```c
GPIOA->CRL &= ~(0xFU << 0);
GPIOA->CRL |=  (0x8U << 0);
GPIOA->ODR &= ~(1U << 0);
```

Configure PA1 as output push-pull:

```c
GPIOA->CRL &= ~(0xFU << 4);
GPIOA->CRL |=  (0x1U << 4);
```

Read PA0:

```c
if ((GPIOA->IDR & (1U << 0)) != 0U)
{
    // PA0 is HIGH
}
```

Control PA1 through BSRR:

```c
GPIOA->BSRR = (1U << 1);   // Set PA1 HIGH
GPIOA->BSRR = (1U << 17);  // Reset PA1 LOW
```

## Mistakes, questions and debugging lessons

1. **Initially confused IDR with the configuration registers.**  
   IDR reads the pin level; CRL/CRH configure the pin mode and configuration.

2. **Bit-mask result confusion.**  
   For example, if IDR contains `0000 0000 0000 0100`, then:
   `IDR & (1U << 2)` gives decimal 4, not decimal 1. The important point in an `if` is that the result is non-zero when the selected bit is HIGH.

3. **Pull-down initially did not work because the button wiring was still connected to GND.**  
   For pull-down wiring, the button must connect PA0 to 3.3 V when pressed. Moving the connection to 3.3 V fixed the behaviour.

4. **A line like `GPIOA->ODR |= (0U << 0)` does nothing.**  
   Shifting zero still gives zero. To clear the PA0 ODR bit, use `GPIOA->ODR &= ~(1U << 0)`.

5. **BSRR is not an ordinary storage register.**  
   Use assignment (`=`) to issue a set/reset action rather than read-modify-write with `|=`.

## Result

Successfully used PA0 as a button input and PA1 as an LED output with both pull-up and pull-down configurations. Understood active-low versus active-high button logic.

## Takeaway

The important skill is not memorising one wiring arrangement: it is being able to determine the input's idle state, pressed state, pull direction and matching software condition.
