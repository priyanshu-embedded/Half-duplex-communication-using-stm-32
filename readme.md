==========================================================
STM32 Half-Duplex UART LED Control Between Two Boards
==========================================================

PROJECT OVERVIEW
----------------
This project demonstrates UART communication in HALF-DUPLEX mode between two STM32 boards.
It is designed so that:

1. Board A:
   - Continuously listens for incoming UART data.
   - If it receives the value 1, transmitted by stm32 second board ,it blinks an LED connected to PA6.
   - Waits for a button press (PA3).
   - When pressed, sends a single byte (value 1) to Board B via UART.
   - After transmission is complete, blinks an LED connected to PA4.

2. Board B:
   - Continuously listens for incoming UART data.
   - If it receives the value 1, transmitted by stm32 second board ,it blinks an LED connected to PA6.
   - Waits for a button press (PA3).
   - When pressed, sends a single byte (value 1) to Board B via UART.
   - After transmission is complete, blinks an LED connected to PA4.



Both boards use USART1 in HALF-DUPLEX mode (single wire TX/RX), and the communication
is interrupt-driven.

----------------------------------------------------------
HARDWARE REQUIREMENTS
----------------------------------------------------------
- Two STM32 boards (tested with STM32F4 series, but adaptable to others).
- Two  push button connected to PA3 (with pull-up enabled).
- Two LED connected to PA4 (Board A) for TX indication.
- Two LED connected to PA6 (Board B) for RX indication.
- Common GND between both boards.
- UART half-duplex connection: PA9 <--> PA9 (USART1 TX pin on both boards).

----------------------------------------------------------
PIN CONFIGURATION
----------------------------------------------------------
Board A:
  PA3  -> Push button input (Pull-up)
  PA4  -> TX complete LED
  PA6  -> RX indication LED
  PA9  -> UART Half-Duplex TX/RX line

Board B:
  PA3  -> Push button input (Pull-up)
  PA4  ->TX complete LED
  PA6  -> RX indication LED
  PA9  -> UART Half-Duplex TX/RX line

----------------------------------------------------------
FIRMWARE BEHAVIOR
----------------------------------------------------------
Main Flow:
1. On startup, both boards initialize UART in half-duplex mode and enable
   the receiver interrupt (`HAL_UART_Receive_IT`).

2. On Board A:
   - An external interrupt on PA3 sets a flag `buttonpress = 1`.
   - The main loop detects this flag and switches the UART to transmitter mode,
     then sends one byte `{1}` using `HAL_UART_Transmit_IT`.
   - When TX completes (`HAL_UART_TxCpltCallback`), the PA4 LED blinks briefly,
     then the UART is switched back to receiver mode.

3. On Board B:
   - When UART RX completes (`HAL_UART_RxCpltCallback`), if the value is 1,
     the PA6 LED blinks briefly.
   - The UART receive interrupt is restarted to continue listening.

----------------------------------------------------------
KNOWN ISSUES / LIMITATIONS
----------------------------------------------------------
1. Simulation in Proteus may not work reliably for half-duplex mode
   due to shared line modeling limitations.
   Workaround: Use full-duplex simulation (TX1->RX2, TX2->RX1)
   or test on real hardware.

2. The code uses `HAL_Delay` inside interrupt callbacks.
   This is generally discouraged, as it blocks other interrupts.
   In production, replace delays with non-blocking timers or flags.

3. Button debounce is not implemented.
   Rapid presses may cause multiple transmissions.

4. DMA is initialized but not actively used in this code.
   The transfer is handled via interrupts.

----------------------------------------------------------
HOW TO USE
----------------------------------------------------------
1. FLASHING THE CODE
   - Build and flash this code separately onto each STM32 board.
   - Use the same code for both boards, but connect the LEDs and buttons
     as per the respective role (Board A or Board B).

2. HARDWARE CONNECTIONS
   - Connect PA9 of Board A to PA9 of Board B (UART line).
   - Connect GND of both boards together.
   - Attach LEDs and button as per pin configuration above.
   - Connect Circuit as given in half duplex communication stm32.pdsprj

3. TESTING
   - Press the button on Board A.
   - Observe PA4 LED on Board A blink after transmission.
   - Observe PA6 LED on Board B blink after reception.

----------------------------------------------------------
FOLDER STRUCTURE
----------------------------------------------------------
/Core/Src/main.c          -> Main application code
/Core/Inc/main.h          -> Header definitions
/Drivers/...              -> HAL driver files
README.txt                -> This documentation

----------------------------------------------------------
LICENSE
----------------------------------------------------------
This code is provided AS-IS without warranty.
Refer to the LICENSE file for terms if applicable.

==========================================================
