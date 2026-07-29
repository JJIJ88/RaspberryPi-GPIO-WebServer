# RaspberryPi-GPIO-WebServer
Lightweight easy to use webserver that allows easy use of the GPIO for beginners and intermediate users. 



# Raspberry Pi Industrial Edge Automation Hub

A high-performance, lightweight, multi-threaded industrial control and automation platform built for single-board computing arrays. The application provides a full-stack dashboard ecosystem that interfaces directly with physical GPIO lines, handles real-time telemetry streaming, and features an isolated C# code compiler for running dynamic automation logic loops on the fly.

## Core Architectural Architecture

┌─────────────────────────────────────────────────────────────┐
│                  Blazor Server Front-End                    │
│  ┌───────────────────────┬──────────────────────────────┐   │
│  │   Telemetry Monitor   │    Dynamic Config Editor     │   │
│  └───────────▲───────────┴──────────────┬───────────────┘   │
└──────────────┼──────────────────────────┼───────────────────┘
│ Event Engine Notification│ Database Updates
│ (OnPinStateChanged)      ▼
┌──────────────┴──────────────────────────────────────────────┐
│                  HardwareStateBook (In-Memory)              │
│     Tracks current pin levels, PWM metrics & card states     │
└──────────────▲──────────────────────────────────────────────┘
│                                  ▲
│ High-Speed Telemetry Sync        │ Context Interlocks
│                                  │
┌──────────────┴──────────────────────────────────┴───────────┐
│             GpioPollingService (25ms Background Loop)       │
│  Reads inputs, tracks physical signals & arms active state  │
└────────────────────────────────┬────────────────────────────┘
│ Conditions Met?
▼
┌──────────────────────────────┐
│     CardExecutionFactory     │
│  Runs Roslyn C# Sandbox      │
│  Executes PWM Pulse Trains   │
│  Applies Simple Output Writes│
└──────────────┬───────────────┘
│
▼
[ Physical Hardware Pins ]


---

## Technical Synopsis

The application is structured around a non-blocking, asynchronous execution loop optimized to cross-compile cleanly to `.NET 10` running on `linux-arm` environments:

### 1. High-Speed Background Telemetry Pipeline
* **`GpioPollingService`**: A low-overhead `BackgroundService` thread operating on a deterministic **250ms polling loop**. It directly monitors selected hardware inputs, validates signal noise thresholds, updates the thread-safe global telemetry registry, and evaluates system interlocks.
* **`HardwareStateBook`**: An in-memory concurrent data register tracking high-frequency changes (`PinValue`, `PwmDutyCycle`, and module processing states) across the app domain. It utilizes virtual tracking indexes (`-cardId`) to broadcast decoupled events to the dashboard layout without causing interface typing lag or layout blockages.

### 2. Multi-Mode Modular Hardware Drivers
The ecosystem maps system slots directly to standalone hardware driver modules via a SQLite database configuration map:
* **Simple I/O Controllers**: Dynamically switches pin directions between standard Inputs and Outputs. Features a **Switching Logic Selector** matching industrial PLC standards, allowing physical inputs to evaluate logically as `True (ON)` on either positive voltage edge (`HighIsTrue`) or inverted ground level drops (`LowIsTrue`).
* **PWM Pulse Modulators**: Bypasses traditional loop timing limits by spawning isolated worker loops that continuously cycle pins between high and low positions based on targeted frequencies (Hz) and precise duty cycle percentage widths.
* **Custom C# Script Sandboxes**: Features a real-time code editor that compiles C# plaintext scripts on demand via the **Roslyn Compiler Engine (`Microsoft.CodeAnalysis.CSharp`)**. Plaintext changes instantly clear the IL assembly generation cache, bypassing tooling conflicts to run updated scripts seamlessly.

### 3. Reactive Industrial Control Interface
* Built with Blazor Server, the interface provides comprehensive manual and event-driven configuration management.
* Displays dynamic **ARMED** (amber monitoring) indicators when a hardware module is waiting on sensor line conditions, shifting instantly to a flashing neon-green **ACTIVE** runtime badge when the execution matrix is processing operations.

  *******************************
    Setup
  *******************************

  1.) Copy Folder to RaspberryPi
  2.) Perform chmod +x on the RaspberryPi application in the folder
  3.) Run the applicaiton ( ./RaspberryPi ) or register as a service.

  Default Security:
  Username: Admin
  Password: Admin

  http://localhost:5008
  or
  http://[IP]:5008 from any device reachable
