# Design-and-Implementation-of-an-Automatic-Street-Lighting-System-Using-STM32

## Aim

To design and implement an **Automatic Street Lighting System using STM32 Nucleo-L031K6** that senses the surrounding light intensity and automatically switches the streetlight **ON during dark conditions** and **OFF during bright conditions**.

---

## Components Required

- STM32 Nucleo-L031K6
- Potentiometer to simulate an LDR/light sensor
- LED to represent the streetlight
- 220 Ω resistor
- Wokwi Simulator
- Connecting wires
- Serial Monitor

---

## Theory

An **Automatic Street Lighting System** automatically controls a streetlight based on the surrounding light intensity.

In a practical system, an **LDR (Light Dependent Resistor)** is used to detect the ambient light level. In the Wokwi simulation, a **potentiometer** is used to simulate the changing output of the light sensor.

The analog output from the potentiometer is connected to **PA0**, which is configured as an ADC input of the STM32 Nucleo-L031K6.

The STM32 uses a **12-bit Analog-to-Digital Converter (ADC)** to convert the analog input voltage into a digital value between **0 and 4095**.

For this experiment:

- **Higher ADC value → Dark condition → Streetlight ON**
- **Lower ADC value → Bright condition → Streetlight OFF**

Two separate threshold values are used to prevent frequent switching of the streetlight when the light intensity is close to the switching point.

---

## Pin Configuration

| Component | STM32 Pin | Function |
|---|---|---|
| Potentiometer SIG | PA0 | ADC Input |
| Potentiometer VCC | 3.3V | Power Supply |
| Potentiometer GND | GND | Ground |
| Streetlight LED | PB3 | Digital Output |
| USART2 TX | PA2 | Serial Data Transmission |
| USART2 RX | PA15 | Serial Data Reception |

---

## Block Diagram

~~~text
        Potentiometer
       (LDR Simulation)
             |
             | Analog Signal
             v
          PA0 / ADC
             |
             v
 +--------------------------+
 | STM32 Nucleo-L031K6      |
 |                          |
 | Read Light Intensity     |
 |           |              |
 |           v              |
 | Compare with Threshold   |
 +-----------+--------------+
             |
             v
         PB3 Output
             |
             v
      Streetlight LED
             
             |
          USART2
             |
             v
       Serial Monitor
~~~

---

## Threshold Conditions

| ADC Value | Lighting Condition | Streetlight Status |
|---:|---|---|
| 0–2200 | Bright | OFF |
| 2201–2799 | Intermediate | Maintain Previous State |
| 2800–4095 | Dark | ON |

---

## Algorithm

1. Start the program.
2. Initialize the STM32 HAL library.
3. Configure the system clock.
4. Configure **PA0** as an analog input.
5. Configure **PB3** as a digital output.
6. Initialize the ADC peripheral.
7. Initialize USART2 for serial communication.
8. Read the analog value from the light sensor.
9. Convert the ADC value into a darkness percentage.
10. Compare the ADC value with the predefined threshold values.
11. If the ADC value is **2800 or greater**, switch the streetlight **ON**.
12. If the ADC value is **2200 or lower**, switch the streetlight **OFF**.
13. If the ADC value is between **2201 and 2799**, maintain the previous streetlight state.
14. Display the ADC value, darkness percentage, and streetlight status on the Serial Monitor.
15. Wait for one second.
16. Repeat the process continuously.

---

## Circuit Connections
<img width="527" height="266" alt="image" src="https://github.com/user-attachments/assets/17911000-631c-44ee-9af8-c79ed56dfea2" />


### Potentiometer – LDR Simulation

| Potentiometer Pin | STM32 Connection |
|---|---|
| VCC | 3.3V |
| GND | GND |
| SIG / Middle Pin | PA0 |

### Streetlight LED

| LED Terminal | STM32 Connection |
|---|---|
| Anode (+) | PB3 through 220 Ω resistor |
| Cathode (-) | GND |

---

## Circuit Diagram

~~~text
             Potentiometer
            (LDR Simulation)

          +---------------+
3.3V -----| VCC           |
GND  -----| GND           |
          | SIG           |
          +------+--------+
                 |
                 |
                PA0
                 |
                 v
       +-------------------------+
       | STM32 Nucleo-L031K6     |
       |                         |
       | PA0  -> ADC Input       |
       |                         |
       | PB3  -> LED Output      |
       |                         |
       | PA2  -> USART2 TX       |
       | PA15 -> USART2 RX       |
       +-----------+-------------+
                   |
                  PB3
                   |
                 220 Ω
                   |
                   v
                  LED
                   |
                  GND

       STM32 USART2
             |
             v
      Wokwi Serial Monitor
~~~

---

## Procedure

1. Open **Wokwi Simulator**.
2. Select the **STM32 Nucleo-L031K6** board.
3. Add a **potentiometer** to simulate the LDR/light sensor.
4. Connect the potentiometer **VCC** pin to **3.3V**.
5. Connect the potentiometer **GND** pin to **GND**.
6. Connect the potentiometer **SIG** pin to **PA0**.
7. Connect the streetlight LED to **PB3** through a **220 Ω resistor**.
8. Enter the STM32 HAL program for the Automatic Street Lighting System.
9. Compile the program.
10. Start the simulation.
11. Open the **Serial Monitor**.
12. Rotate the potentiometer to simulate different light conditions.
13. Observe the LED and the corresponding values on the Serial Monitor.
14. Verify that the LED switches ON during dark conditions and OFF during bright conditions.

---

## Expected Output

### Bright Condition

~~~text
ADC Value: 1500
Darkness: 36%
Streetlight: OFF
~~~

### Dark Condition

~~~text
ADC Value: 3300
Darkness: 80%
Streetlight: ON
~~~

---

## Working

The potentiometer is used to simulate the operation of an **LDR light sensor**. It produces an analog voltage according to its position.

The analog signal is applied to **PA0**, which is configured as the ADC input of the STM32.

The STM32 continuously reads the ADC value and compares it with predefined threshold values.

When the ADC value is **2800 or greater**, the system considers the surroundings to be dark and switches the streetlight **ON**.

When the ADC value is **2200 or lower**, the system considers the surroundings to be bright and switches the streetlight **OFF**.

When the ADC value is between **2201 and 2799**, the previous streetlight state is maintained. This prevents rapid ON/OFF switching near the threshold.

The ADC value, darkness percentage, and streetlight status are also transmitted through **USART2** and displayed on the **Wokwi Serial Monitor**.

---

## Applications

- Automatic street lighting
- Highway lighting systems
- Campus lighting
- Garden lighting
- Parking-area lighting
- Smart city lighting systems
- Energy-efficient outdoor lighting

---

## Result

Thus, the **Automatic Street Lighting System using STM32 Nucleo-L031K6** was designed and implemented successfully. The streetlight automatically switches **ON during dark conditions** and **OFF during bright conditions** based on the simulated light-sensor input.
