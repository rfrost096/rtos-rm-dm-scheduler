# FreeRTOS Real-Time Scheduler Extension

**Code available upon request** (Starter code is protected IP)

This project extends the FreeRTOS kernel on an ATmega2560 (Arduino Mega) with custom scheduling capabilities, advanced profiling, and real-time execution tracking. It implements custom scheduling policies and provides a robust framework for monitoring task deadlines and execution times.

## Key Features

*   **Custom Scheduling Algorithms:** Implements static priority scheduling based on Rate Monotonic (RM) and Deadline Monotonic (DM) policies.
*   **Execution Time Tracking:** Monitors Worst-Case Execution Time (WCET) on a tick-by-tick basis, detecting tasks that exceed their allocated execution time and suspending them safely to protect overall system schedulability.
*   **Deadline Monitoring:** Actively tracks absolute deadlines for periodic tasks. It includes mechanisms to detect missed deadlines, securely terminate the violating task, and restart it in the subsequent frame while logging the event.
*   **High-Speed Serial Profiling:** Features a custom, low-overhead event tracking system. It records tick-by-tick CPU occupation, execution times, and scheduling overhead. 
*   **Companion Diagnostic Tool:** Includes a C++ application (`reader.cpp`) designed to run on a host PC. This application catches the high-speed binary serial output from the microcontroller and decodes it into a human-readable, chronological event log for analysis.

## High-Level Design Architecture

To achieve these features while maintaining the integrity of the base FreeRTOS system and adhering to the constraints of an 8-bit microcontroller, several key design decisions were made:

1.  **Supplemental State Tracking:** Instead of deeply modifying core FreeRTOS data structures (which can complicate upgrades and debugging), scheduling parameters (such as execution states, dynamic priorities, absolute deadlines, and maximum execution times) are maintained in a supplementary Task Control Block (TCB) array. This array is synchronized with FreeRTOS task handles.
2.  **Dedicated Scheduler Task:** To minimize the latency and overhead introduced in high-priority hardware interrupts, complex deadline checking and task recreation logic is offloaded from the system tick hook (`vApplicationTickHook`). Instead, a dedicated high-priority Scheduler Task is utilized. The tick hook simply wakes this scheduler at a defined periodic rate (e.g., every 100ms) to evaluate system health, process potential deadline misses, and manage suspended tasks.
3.  **Lock-Free(ish) Serial Logging:** To prevent the profiling system from significantly altering the timing behavior it is meant to measure, the serial logging uses a highly optimized circular buffer. Events are recorded rapidly during critical sections. The actual transmission of this data is handled by a low-priority serial task that streams raw binary data at a high baud rate (115200) to ensure the buffer clears quickly without blocking vital system operations.

## Build and Usage Instructions

This project is built using PlatformIO.

### Firmware (ATmega2560)

1.  Initialize and compile the firmware:
    ```bash
    pio run
    ```
2.  Upload to the target device (adjust the port as necessary):
    ```bash
    pio run --target upload --upload-port /dev/ttyACM0
    ```

### Companion Serial Reader

1.  Compile the C++ reader application on your host machine:
    ```bash
    g++ -O3 -Wall reader/reader.cpp -o reader_app
    ```
    *(Note: Ensure the path to `reader.cpp` matches your local directory structure).*
2.  Execute the reader, pointing it to the correct serial port:
    ```bash
    ./reader_app /dev/ttyACM0
    ```

## Disclaimer

This repository is maintained as an archived academic project to demonstrate embedded systems programming, real-time operating system concepts, and C++ application development. Specific assignment details and lower-level algorithmic implementations are kept abstract to protect intellectual property.
