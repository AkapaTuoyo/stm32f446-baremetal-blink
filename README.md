# Bare-Metal STM32F446RE LED Blink (Embedded System Hello World)

A bare-metal embedded C project that blinks the LD2 LED on an STM32 Nucleo-F446RE
board using custom GPIO and RCC register-overlay structs

## What this project does

This Projects enables GPIOA Peripheral clock, configures pin PA5 which controls LD2 as a general-purpose,
and toggles it in an infinite loop to make LD2 blink.

## Hardware

- STM32 Nucleo-F446RE development board
- Micro USB cable

## Tooling

- STM32CubeIDE
- Compiler: arm-none-eabi-gcc (default in CubeIDE)
- Flashing: ST-LINK over USB

## Key concepts demonstrated

- **Memory-mapped peripherals.** STM32 peripheral registers live at fixed
  addresses in the memory map (e.g. GPIOA at 0x40020000). Reading or writing
  these addresses directly controls hardware.
- **Register overlay using a C struct.** Defining a struct whose fields match
  the peripheral's register layout, then casting the base address to a pointer
  of that struct type, lets you access registers as `GPIOA->MODER` instead of
  doing manual pointer arithmetic.
- **`volatile` qualifier.** The essence of volatile qualifier is to let the compiler know that this value can change at any point  due to interaction to the hardware,
 and also tell the compiler don't optimize away or cache reads/writes
- **Clock gating via RCC.** Peripherals on the STM32 are unclocked by default to save power. 
No clock signal means the peripheral's internal logic is frozen. Without a clock their registers don't function writes are silently lost
- **Reserved fields in peripheral structs.** The Hardware
  register map has reserved spots that must be matched in the struct layout, or all
  subsequent fields are misaligned


## Bugs I hit

### 1. Trailing semicolon in `#define`

I initially wrote `#define GPIOA ((GPIO_Def*)0x40020000);` with a trailing
semicolon. Because `#define` is text substitution, every use of `GPIOA` had
the semicolon pasted in, breaking the syntax of every subsequent expression.
Lesson: macros don't end in semicolons 

### 2. Missing reserved field in the RCC struct

I initially missed the reversed spots between different registers, 
every write after that offset went to the wrong register

### 3. Toggle inside the delay loop

Initially placed `GPIOA->ODR ^= (1<<5);` inside the inner delay
loop, causing PA5 to toggle 100,000+ times per loop iteration far too fast
to see. Fix was to put the toggle outside, leaving the inner loop empty as
a busy-wait delay


## What I'd improve

 - Replace the busy-wait delay with a SysTick-based delay so timing is independent of CPU clock , 
 - Add the user button on PC13 as an on/off trigger whta

## References

- RM0390 — STM32F446xx Reference Manual
- UM1724 — STM32 Nucleo-64 boards user manual
- DS10693 — STM32F446xx datasheet

## Photo
[LD2 blinking on Nucleo F446RE](docs/led_blinking.gif)