# Day 3 — Mechanical switch bounce and software debounce

## Goal

Understand why a mechanical pushbutton can generate unstable transitions and implement a one-toggle-per-press behaviour. Start with a blocking delay-based solution, then replace it with a non-blocking timestamp-based approach.

## Hardware setup

Same basic wiring as Day 2:

- PA0: pushbutton input, configured as internal pull-down.
- PA1: external LED output.
- Button connects PA0 to 3.3 V when pressed.
- Released = LOW; pressed = HIGH.

## Topics studied

### 1. Mechanical switch bounce

A physical switch does not always produce one clean transition. Its contacts can rapidly connect and disconnect for a short time during pressing or releasing.

A simplified press-bounce sequence might look like:

`LOW → HIGH → LOW → HIGH → LOW → HIGH`

Without debouncing, the MCU may interpret one physical press as multiple events.

### 2. Blocking debounce with HAL_Delay()

Basic strategy:

1. Detect a possible press.
2. Wait 20 ms using `HAL_Delay(20)`.
3. Read the input again.
4. Accept the press only if it is still active.

This is easy to understand, but it blocks the main application loop during the delay.

### 3. One event per physical press

Introduced a `button_ready` flag:

- `1`: a new press may be accepted.
- `0`: a press has already been accepted; wait for release before accepting another.

For the current pull-down wiring, HIGH means pressed and LOW means released.

### 4. HAL tick: HAL_IncTick() and HAL_GetTick()

- In the usual STM32 HAL setup, SysTick generates a periodic interrupt (commonly every 1 ms).
- The HAL SysTick handler calls `HAL_IncTick()`, which increments the HAL tick counter.
- `HAL_GetTick()` reads and returns the current tick count.
- The application normally does not call `HAL_IncTick()` manually.
- Saving `previous_time = HAL_GetTick()` copies the current timestamp; it does not reset or pause the counter.

Conceptual example:

```c
uint32_t previous_time = HAL_GetTick();

if (HAL_GetTick() - previous_time >= 20U)
{
    // At least 20 ms elapsed
}
```

Unsigned subtraction is preferred for measuring elapsed intervals because it also behaves correctly across the tick counter's wraparound, for intervals much shorter than the full counter period.

### 5. Non-blocking debounce

Instead of waiting inside `HAL_Delay(20)`, record the time when a possible press is first seen. The main loop can continue doing other work and check whether 20 ms have elapsed on later iterations.

Variables:

```c
uint32_t previous_time = 0;
uint8_t button_ready = 1;
uint8_t checking_press = 0;
```

- `previous_time`: timestamp of the possible press.
- `button_ready`: whether a new press may be accepted.
- `checking_press`: whether a possible press is awaiting confirmation.

## Working implementation used in the exercise

This is the basic press-debounce implementation tested on the existing PA0/PA1 setup:

```c
uint32_t previous_time = 0;
uint8_t button_ready = 1;
uint8_t checking_press = 0;

// Enable GPIOA clock
RCC->APB2ENR |= (1U << 2);

// PA0: input pull-down
GPIOA->CRL &= ~(0xFU << 0);
GPIOA->CRL |=  (0x8U << 0);
GPIOA->ODR &= ~(1U << 0);

// PA1: output push-pull
GPIOA->CRL &= ~(0xFU << 4);
GPIOA->CRL |=  (0x1U << 4);

while (1)
{
    // A. Detect a possible press
    if (button_ready && !checking_press &&
        ((GPIOA->IDR & (1U << 0)) != 0U))
    {
        previous_time = HAL_GetTick();
        checking_press = 1;
    }

    // B. Confirm the press after 20 ms
    if (checking_press &&
        (HAL_GetTick() - previous_time >= 20U))
    {
        if ((GPIOA->IDR & (1U << 0)) != 0U)
        {
            GPIOA->ODR ^= (1U << 1);  // Toggle PA1
            button_ready = 0;
        }

        checking_press = 0;
    }

    // C. Detect release and re-arm
    if ((GPIOA->IDR & (1U << 0)) == 0U)
    {
        button_ready = 1;
        checking_press = 0;
    }
}
```

Keep the CubeIDE project's existing startup and clock initialization code. The snippet above shows the GPIO/application logic used in the exercise, not a replacement for the complete generated project.

## Mistakes, questions and debugging lessons

1. **Pull direction and physical wiring must match.**  
   The actual setup is pull-down, not pull-up: PA0 is connected to 3.3 V by the button. Pressed is HIGH.

2. **The first debounce attempt had brace/placement issues.**  
   We reviewed the structure of the independent checks and corrected the loop so press detection, elapsed-time confirmation and release handling execute in the intended order.

3. **A press candidate must be cancelled if the input is released before the confirmation interval.**  
   Clearing `checking_press` on release prevents an old pending candidate from being confirmed later.

4. **`button_ready` and `checking_press` have different jobs.**  
   `checking_press` tracks a pending debounce check. `button_ready` prevents repeated accepted events while the button remains held.

5. **HAL_IncTick() is not an application timer-start function.**  
   It increments the HAL tick counter and is normally called by the HAL SysTick interrupt handler. `HAL_GetTick()` reads the counter.

6. **HAL_GetTick() does not block.**  
   The main loop can continue executing other tasks between the timestamp capture and the 20 ms confirmation. The CPU still executes instructions sequentially; this is not simultaneous execution of all tasks.

## Current limitation / next improvement

This beginner implementation debounces the press confirmation but does not independently debounce the release transition. A brief LOW during release bounce can re-arm the button early. The next improvement is to implement a proper state machine and debounce both press and release.

## Result

The user reported the non-blocking implementation working with the same PA0 button and PA1 LED wiring used on Day 2.

## Takeaway

This exercise introduced elapsed-time measurement, flags, event handling and non-blocking program structure. The next step is to make the logic more robust and easier to extend using a Finite State Machine (FSM).
