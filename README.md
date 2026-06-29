PWM_01
======

Overview
--------
This project measures an incoming PWM signal's frequency and duty cycle using TIM2 input-capture on an STM32F103 MCU. Captured values are stored in `Frequency` and `Duty_Cycle` in `Core/Src/main.c`.

Key points
----------
- Timer: `TIM2` configured as input-capture (CH1 rising = period, CH2 falling = pulse width).
- GPIO: `GPIOA` clock is enabled; connect the PWM signal to PA0 (TIM2_CH1) and PA1 (TIM2_CH2).
- Timer clock: code assumes a 72 MHz timer clock (see `TIM2_CLK` in `main.c`).

Build
-----
Prerequisites: `arm-none-eabi-gcc`, `cmake`, and `ninja` (or another generator). A toolchain file is provided at `cmake/gcc-arm-none-eabi.cmake`.

Example (from project root):

```bash
mkdir -p build
cd build
cmake -S .. -B . -G Ninja -DCMAKE_TOOLCHAIN_FILE=cmake/gcc-arm-none-eabi.cmake
cmake --build .
```

The resulting ELF/BIN is placed under the chosen build directory (check `build/Debug` if you used the existing preset).

Flash
-----
You can flash with OpenOCD, ST-Link tools, or other programmers. Example using `st-flash` (adjust path/name as needed):

```bash
st-flash write build/Debug/PWM_01.bin 0x08000000
```

Or use OpenOCD + GDB for debugging.

Measurement details
-------------------
- The code starts input capture on TIM2 CH1 (interrupt) and CH2.
- `HAL_TIM_IC_CaptureCallback` reads:
  - `IC_Val1` = captured ticks for the full period (CH1)
  - `IC_Val2` = captured ticks for the high pulse (CH2)
- Computations:
  - `Duty_Cycle = (IC_Val2 / IC_Val1) * 100.0f`
  - `Frequency = TIM2_CLK / IC_Val1` where `TIM2_CLK` = 72,000,000 Hz

Observing results
-----------------
- The measured values are stored in `volatile` globals in `Core/Src/main.c`. Use a debugger (watch/inspect) or add UART/ITM prints to output them at runtime.

Notes / Caveats
---------------
- `htim2.Init.Prescaler = 0` so timer tick = 1 / 72 MHz. For very high frequencies the 16-bit counter may overflow; consider adding a prescaler.
- The project enables `GPIOA` clock; ensure the signal is wired to PA0/PA1.
- No runtime display is implemented; add UART or LED indications if needed.

Want changes?
-------------
Tell me if you want UART prints, a simple LED indicator, or a sample flash/debug command tailored to your programmer.
