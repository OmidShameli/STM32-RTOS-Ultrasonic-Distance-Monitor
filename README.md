# STM32-RTOS-Ultrasonic-Distance-Monitor
Embedded real-time distance measurement system using STM32 and FreeRTOS with interrupt-driven ultrasonic sensing and UART communication 
# 🔊 Ultrasonic Distance Monitor — STM32 + FreeRTOS

A real-time distance measurement system built on STM32 Nucleo F446RE using FreeRTOS, Timer Input Capture, and UART logging.

-----

## 📋 Project Overview

This project measures distance using an HC-SR04 ultrasonic sensor, processes the data in real-time using FreeRTOS tasks, triggers an LED alert for close objects, and logs distance values over UART.

-----

## 🛠️ Hardware

|Component    |Details                                       |
|-------------|----------------------------------------------|
|MCU Board    |STM32 Nucleo F446RE(ARM Cortex-M4 @180MHz Max)|
|Sensor       |HC-SR04 Ultrasonic Distance Sensor            |
|Alert        |Outboard LED (PC0)                            |
|Communication|UART2 via USB (115200 baud)                   |

### Pin Configuration

|Signal|Pin|Notes                    |
|------|---|-------------------------|
|Trig  |PA6|GPIO Output              |
|Echo  |PA7|TIM3_CH2 Input Capture   |
|LED   |PA5|GPIO Output (onboard LED)|


> ⚠️ Important: HC-SR04 Echo output is 5V. A voltage divider (1kΩ + 2kΩ) is required to bring it down to 3.3V for STM32.
Echo (5V) ──── R1 (1kΩ) ────┬──── PA7 (3.3V)
                              │
                           R2 (2kΩ)
                              │
                             GND

-----

## ⚙️ Software Architecture

### FreeRTOS Tasks
┌─────────────────────────────────────────────┐
│  SensorTask (500ms)                          │
│  → Send Trig pulse                           │
│  → Measure Echo via Timer Input Capture      │
│  → Calculate distance                        │
│  → Write to Queue                            │
└──────────────────┬──────────────────────────┘
                   │ xDistanceQueue (size=1)
          ┌────────┴────────┐
          ▼                 ▼
┌─────────────────┐  ┌─────────────────┐
│  AlertTask      │  │  LogTask        │
│  (100ms)        │  │  (1000ms)       │
│  → Read Queue   │  │  → Read Queue   │
│  → LED ON/OFF   │  │  → UART print   │
│  → < 20cm = ON  │  │  → distance cm  │
└─────────────────┘  └─────────────────┘

### Distance Calculation
distance (cm) = (echo_time_µs × 0.034) / 2

Sound travels at ~340 m/s = 0.034 cm/µs. Divide by 2 because the signal travels to the object and back.

-----

## 📁 Key Files
Core/
├── Src/
│   └── main.c          ← All application code
├── Inc/
│   └── main.h

-----

## 🔧 Configuration

|Parameter      |Value                    |
|---------------|-------------------------|
|TIM3 Prescaler |79 (1 tick = 1µs @ 80MHz)|
|TIM3 Channel 2 |Input Capture direct mode|
|UART2 Baud Rate|115200                   |
|Alert Threshold|< 20 cm                  |
|Sensor Timeout |30 ms                    |

-----

## 📦 Dependencies

- STM32CubeIDE
- STM32CubeMX
- FreeRTOS (CMSIS_V2)
- HAL Drivers (STM32F4xx)

-----

## 🚀 How to Run

1. Clone the repository
1. Open project in STM32CubeIDE
1. Build the project (`Ctrl + B`)
1. Flash to Nucleo board (`Run → Debug`)
1. Open PuTTY or any serial monitor:
- Port: COM port of Nucleo
- Baud: 115200
1. Point sensor at an object — distance prints every second

-----

## 💡 Key Concepts Used

|Concept            |Usage                                     |
|-------------------|------------------------------------------|
|Timer Input Capture|Precise Echo pulse measurement (µs)       |
|NVIC Interrupt     |Non-blocking Echo detection               |
|FreeRTOS Queue     |Safe data transfer between tasks          |
|xQueueOverwrite    |Always keep latest distance value         |
|xQueuePeek         |Multiple tasks read same value            |
|Voltage Divider    |5V → 3.3V level shifting for Echo pin     |
|volatile           |Safe shared variable between ISR and tasks|

-----

## 👤 Author

Omid Shameli
Embedded Systems Engineer
[github.com/OmidShameli](https://github.com/OmidShameli)
