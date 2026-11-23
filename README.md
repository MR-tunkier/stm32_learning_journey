# STM32 Learning Journey

Hi! I'm an engineering student building strong fundamentals in embedded systems and hardware.

This repository is my day-by-day learning diary. I'm documenting what I study, what I build, the mistakes I make, the bugs I encounter, and what I learn while debugging them.

## Hardware and tools

- **MCU board:** STM32 Blue Pill, STM32F103C6T6
- **IDE:** STM32CubeIDE
- **Programming approach so far:** C with CMSIS register-level GPIO access, while retaining the CubeIDE/HAL startup and project infrastructure
- **Programmer/debugger:** ST-Link V2
- **Breadboard exercises:** onboard PC13 LED, external LEDs, and pushbuttons

## Learning log

| Day | Main topics | Practical result |
|---|---|---|
| [Day 1](docs/day-01.md) | Reference manual navigation, bus overview, GPIO registers, RCC GPIO clock enable, BSRR | Blinked the active-low PC13 onboard LED using register-level code |
| [Day 2](docs/day-02.md) | GPIO input/output, CRL configuration, IDR, pull-up and pull-down, button wiring | Read a button on PA0 and controlled an LED on PA1 |
| [Day 3](docs/day-03.md) | Mechanical switch bounce, blocking and non-blocking debounce, HAL tick, state flags | Implemented and tested timestamp-based button debounce |

## How I am learning

- Understand the register and the hardware behaviour before copying code.
- Read the STM32 reference manual and datasheet alongside practical work.
- Build on existing hardware before buying more components.
- Record mistakes and debugging lessons, not just successful results.
- Gradually move from peripheral basics towards timers, interrupts, communication protocols, RTOS, digital design, computer architecture and other hardware fields.

## Current status

Completed the first three learning days. Next planned topic: **Finite State Machines (FSMs)**, followed by a more robust button implementation that debounces both press and release.

This is a learning log, not a claim of expertise. The goal is to understand the fundamentals well enough to work with unfamiliar microcontrollers and hardware platforms later.
