# EMG Interface for Muscle Activity Monitoring

### Author: **Luciana Falcon**  

---

## Description  

The project consists of the development of an **embedded system capable of acquiring, processing, and visualizing bioelectrical muscle signals (EMG)** using **non-invasive recording electrodes**.  

The system enables the **detection of muscle contractions** through digital processing implemented in the **STM32 microcontroller**, displaying the results on a **local display** and transmitting the data via **Bluetooth** to a PC or mobile device for analysis or visualization.  

The main objective is **to design a complete embedded system** that integrates all stages of biological signal processing:
- **Analog sensing** (electrodes + amplifier).  
- **Signal conditioning and digitization** (microcontroller ADC).  
- **Processing and event detection** (embedded software).  
- **Communication and visualization** (Bluetooth + display).  

---

## Project Scope  

- Acquisition of EMG signals using non-invasive electrodes and amplification through the AD8232 module.  
- Detection of muscle contractions through digital processing and threshold detection.  
- Real-time visualization of muscle activation level on an OLED display.  
- Transmission of data via Bluetooth to a PC or mobile device.  
- Activation of an LED or audible alert when the contraction threshold is exceeded.  

---

## Requirements  

### Development Platform  
- **Board used:** NUCLEO-F103RB  
- **Microcontroller:** STM32  

### Firmware  
Implementation using a **Super Loop (bare-metal, event-triggered)** architecture with periodic tasks:
- ADC / I2C reading.  
- Digital processing and peak detection.  
- UART–BLE (Bluetooth) communication.  
- OLED display update.  

---

### Hardware  

- **Dip Switch / Button:** Allows starting or stopping EMG signal acquisition.  
- **Buzzer:** Emits an audible signal when the contraction threshold is exceeded.  
- **OLED Display (I2C):** Displays the contraction level in real time.  
- **Bluetooth Module HM-10:** Transmits EMG data to a PC or mobile device.  
- **External EEPROM / Internal Flash Memory:** Stores historical data or calibration parameters.  
- **Analog sensor (AD8232 + electrodes):** Captures EMG signals in the millivolt range and feeds them to the STM32 ADC.  

---

## Block Diagram  

![embebidos](https://github.com/user-attachments/assets/35658773-ff54-48d2-b060-35d2d7419d01)

---

## Power Consumption and Load Factor

Table 1.1 presents the system power consumption analysis under different operating scenarios. Measurements were performed by powering the node from a regulated 5 V supply,
using a multimeter in series to record the current consumed by the STM32 and its peripherals in each case. Each scenario was measured during multiple executions to obtain a representative value.

| Scenario               | Active Peripherals                     | Test Description                                                                      | Consumption (mA) |
|------------------------|------------------------------------------|--------------------------------------------------------------------------------------|------------------|
| **Baseline**           | None                                     | Microcontroller running minimal loop, with no peripherals enabled.                  |    12.9          |
| **ADC active**         | ADC1                                     | Continuous conversion of EMG signal from the AD8232 module.                         |    14.1          |
| **I2C active**         | I2C1 (OLED + EEPROM)                     | Communication with OLED and I2C memory access, screen refresh.                      |    13.4          |
| **UART active**        | USART1 (BLE HM-10)                       | Periodic transmission of contraction level to the HM-10 Bluetooth module.           |    11.4          |
| **Buzzer**             | GPIO                                     | Buzzer activation when contraction level exceeds the configured threshold.          |    13.2          |
| **Full Super Loop**    | ADC + I2C + UART + GPIO                  | Full operation: EMG acquisition, processing, BLE transmission and OLED active.     |    14.5          |

<p align="center"><b>Table 1.1 — Power consumption measurement scenarios.</b></p>

Table 1.2 lists the periodic tasks present in the super-loop. Their execution times were estimated using instrumented measurements with the internal STM32 timer,
along with their corresponding periods.

| Task                         | Description                                | CPU Time (Ci) | Period (Ti) | Ci/Ti   |
|------------------------------|--------------------------------------------|---------------|-------------|---------|
| **EMG Acquisition (ADC)**    | Continuous sampling at 1 kHz               | 0.02 ms       | 1 ms        | 0.020   |
| **EMG Processing**           | Filtering, RMS and level calculation       | 0.40 ms       | 20 ms       | 0.020   |
| **OLED Display (I2C)**       | Screen refresh                             | 1.50 ms       | 100 ms      | 0.015   |
| **Bluetooth BLE (UART)**     | Transmission of processed value            | 0.30 ms       | 20 ms       | 0.015   |
| **Threshold Detection**      | Comparison and buzzer activation           | 0.05 ms       | 20 ms       | 0.0025  |
| **Button Reading**           | Periodic digital input reading             | 0.02 ms       | 50 ms       | 0.0004  |

<p align="center"><b>Table 1.2 — Periodic tasks considered for load factor calculation.</b></p>

**Total system utilization factor:**  
Based on the execution times and periods defined in Table 1.2, the total system utilization factor is calculated as the sum of Ci/Ti for all periodic tasks.

u = 0.020 + 0.020 + 0.015 + 0.015 + 0.0025 + 0.0004 = **0.0729 → 7.29%**

---

## Requirements Elicitation and Use Cases

Table 2.1 summarizes the essential functions that the system must perform to meet the project objectives.
Each requirement is assigned a unique identifier to ensure traceability during design and implementation.

| Code | Functional Requirement |
|------|------------------------|
| **FR1** | The system must acquire the EMG signal using electrodes connected to the AD8232 module. |
| **FR2** | The system must digitize the signal using the STM32 ADC at at least 1 kHz. |
| **FR3** | The system must process the signal (RMS, filtering, threshold). |
| **FR4** | The system must display the muscle activity level on the OLED screen. |
| **FR5** | The system must transmit the processed data via Bluetooth to a PC or mobile device. |
| **FR6** | The system must activate a buzzer when muscle activity exceeds a configurable threshold. |
| **FR7** | The system must allow starting/stopping monitoring using a button. |

<p align="center"><b>Table 2.1 — Functional Requirements FR.</b></p>

Table 2.2 presents performance constraints and operating conditions that the system must satisfy to guarantee efficiency,
proper timing response and communication stability.

| Code | Non-Functional Requirement |
|------|----------------------------|
| **NFR1** | The system must maintain consumption below 20 mA under standard operation. |
| **NFR2** | The Bluetooth interface must ensure continuous communication. |
| **NFR3** | The display must update at least every 100 ms. |
| **NFR4** | Contraction detection must occur with latency below 50 ms. |
| **NFR5** | The firmware must be implemented using a super-loop architecture. |

<p align="center"><b>Table 2.2 — Non-Functional Requirements NFR.</b></p>

Tables 3.1 to 4.2 present the two use cases of the system.

| Item | Description |
|------|-------------|
| **Actors** | User, EMG System. |
| **Preconditions** | The device is powered on and electrodes are properly placed. |

<p align="center"><b>Table 3.1 — Use Case 1: Muscle Activity Monitoring.</b></p>

| Step | Action |
|------|--------|
| **1** | The user presses the button to start monitoring. |
| **2** | The system begins sampling the EMG signal. |
| **3** | The signal is processed and the muscle activation level is calculated. |
| **4** | The processed value is sent to the display and via Bluetooth. |
| **5** | If the level exceeds the threshold, the buzzer is activated. |

<p align="center"><b>Table 3.2 — Main Flow.</b></p>

| Step | Action |
|------|--------|
| **A1** | The user presses the button again and the system stops acquisition. |

<p align="center"><b>Table 3.3 — Alternative Flow.</b></p>

| Item | Description |
|------|-------------|
| **Actor** | EMG System. |
| **Preconditions** | The system is operating and the EMG signal is being acquired. |
| **Postconditions** | The buzzer is activated or deactivated according to the muscle activation level. |
| **General Description** | The system continuously analyzes the RMS level of the EMG signal and generates an alarm when a defined threshold is exceeded. |

<p align="center"><b>Table 4.1 — Use Case 2: Activation Threshold Alarm.</b></p>

| Step | Action |
|------|--------|
| **1** | The system calculates the RMS of the EMG signal in each processing window. |
| **2** | It compares the level with the configured threshold. |
| **3** | If exceeded, it activates the buzzer. |
| **4** | When the RMS value drops below the threshold, the system deactivates the buzzer. |

<p align="center"><b>Table 4.2 — Flow.</b></p>

---
