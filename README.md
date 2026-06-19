# Performance and Power Profiling of FreeRTOS Task Scheduling Under DFS and Tickless Idle

This repository contains the source code, hardware configuration setups, and experimental evaluation scripts for profiling the **FreeRTOS** scheduling infrastructure on an ARM Cortex-M4 microcontroller (**STM32F411CEU6 Black Pill**). 

The core focus of this project is to analyze the multi-layered impacts of Dynamic Frequency Scaling (DFS), task workload burdens, and low-power Tickless Idle states on both context switch latency and theoretical current draw.

## Key Features & Implementation
* **Hardware-Driven Benchmarking:** Utilizing the internal ARM **Data Watchpoint and Trace (DWT)** cycle counter register (`DWT->CYCCNT`) for nanosecond-precision, zero-overhead execution timing.
* **Dynamic Frequency Scaling (DFS):** Runtime configuration of the system clock (HCLK) via Phase-Locked Loop (PLL) tuning across 8 MHz, 16 MHz, and 100 MHz.
* **Low-Power Optimization:** Implementation and analysis of `configUSE_TICKLESS_IDLE` to suppress standard periodic 1 kHz SysTick interrupts during operational inactivity.
* **Determinism Validation:** Empirical evaluation verifying the $O(1)$ temporal predictability of the FreeRTOS ready-list scheduler across scaled active task loads (from 2 to 8 tasks).

## Experimental Results Summary
Our empirical findings expose a critical real-time paradox:
* **Energy Savings:** Reducing the processor clock from 100 MHz down to 8 MHz achieves up to an **88% reduction** in active dynamic current draw.
* **Latency Penalty:** The physical context switch latency expands exponentially from **$1.82~\mu s$ (at 100 MHz)** to **$21.50~\mu s$ (at 8 MHz)**, rendering aggressive clock throttling hazardous for high-frequency hard real-time systems.
* **$O(1)$ Predictability:** Scaling the active task payload induces **zero additional scheduling overhead** in terms of clock cycles (anchored tightly at 176 cycles at 16 MHz).

## Hardware & Software Environment
* **Microcontroller:** STMicroelectronics STM32F411CEU6 (ARM Cortex-M4 with FPU)
* **RTOS Kernel:** FreeRTOS (Preemptive, Priority-based Scheduling)
* **Development IDE:** STM32CubeIDE / Keil MDK
* **Instrumentation Model:** Extracted from official STM32F411xC/E production data characterization benchmarks ($V_{DD} = 3.0\text{ V}$, $T_A = 25^\circ\text{C}$).

## Repository Structure
```text
├── Src/
│   ├── main.c              # Core application entry point and task setup
│   ├── freertos.c          # FreeRTOS hook functions and configuration routines
│   └── stm32f4xx_it.c      # Interrupt service routines (SysTick & PendSV hooks)
├── Inc/
│   ├── FreeRTOSConfig.h    # Custom kernel configurations (Tickless Idle settings)
│   └── main.h              # Hardware pin definitions and macro abstractions
└── README.md               # Documentation
