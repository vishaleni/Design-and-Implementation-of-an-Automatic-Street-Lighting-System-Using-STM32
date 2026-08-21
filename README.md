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

## program
// Automatic Street Lighting System
// STM32 Nucleo-L031K6 using Wokwi
//
// Potentiometer on PA0 simulates the LDR.
// LED on PB3/D13 represents the streetlight.
// USART2 displays light level and lamp status.

#include <stdio.h>
#include <stdint.h>
#include <stm32l0xx_hal.h>

/* Streetlight LED */
#define STREET_LED_PORT          GPIOB
#define STREET_LED_PIN           GPIO_PIN_3
#define STREET_LED_CLK_ENABLE()  __HAL_RCC_GPIOB_CLK_ENABLE()

/* Light sensor input */
#define LIGHT_SENSOR_PORT        GPIOA
#define LIGHT_SENSOR_PIN         GPIO_PIN_0
#define LIGHT_SENSOR_CHANNEL     ADC_CHANNEL_0

/* USART2 pins */
#define VCP_TX_PIN               GPIO_PIN_2
#define VCP_RX_PIN               GPIO_PIN_15

/*
 * For this simulation:
 * Higher ADC value = darker condition
 * Lower ADC value  = brighter condition
 */
#define LIGHT_ON_THRESHOLD       2800
#define LIGHT_OFF_THRESHOLD      2200

UART_HandleTypeDef huart2;
ADC_HandleTypeDef hadc1;

void SystemClock_Config(void);
static void MX_GPIO_Init(void);
static void MX_ADC1_Init(void);
static void MX_USART2_UART_Init(void);
static uint32_t Read_Light_Level(void);
void Error_Handler(void);

int main(void)
{
  uint32_t lightValue;
  uint32_t lightPercentage;
  uint8_t streetLightStatus = 0;

  HAL_Init();
  SystemClock_Config();

  MX_GPIO_Init();
  MX_ADC1_Init();
  MX_USART2_UART_Init();

  printf("\r\n====================================\r\n");
  printf("Automatic Street Lighting System\r\n");
  printf("====================================\r\n");
  printf("PA0 : Light sensor input\r\n");
  printf("PB3 : Streetlight LED\r\n\r\n");

  if (HAL_ADCEx_Calibration_Start(
          &hadc1,
          ADC_SINGLE_ENDED) != HAL_OK)
  {
    Error_Handler();
  }

  while (1)
  {
    lightValue = Read_Light_Level();

    /*
     * Convert ADC reading into percentage.
     * 0    = 0 percent darkness
     * 4095 = 100 percent darkness
     */
    lightPercentage =
        (lightValue * 100U) / 4095U;

    /*
     * Dark condition: switch streetlight ON.
     */
    if ((lightValue >= LIGHT_ON_THRESHOLD) &&
        (streetLightStatus == 0))
    {
      streetLightStatus = 1;

      HAL_GPIO_WritePin(
          STREET_LED_PORT,
          STREET_LED_PIN,
          GPIO_PIN_SET);

      printf("Dark condition: Streetlight ON\r\n");
    }

    /*
     * Bright condition: switch streetlight OFF.
     */
    else if ((lightValue <= LIGHT_OFF_THRESHOLD) &&
             (streetLightStatus == 1))
    {
      streetLightStatus = 0;

      HAL_GPIO_WritePin(
          STREET_LED_PORT,
          STREET_LED_PIN,
          GPIO_PIN_RESET);

      printf("Bright condition: Streetlight OFF\r\n");
    }

    printf("ADC: %lu | Darkness: %lu%% | Light: %s\r\n",
           (unsigned long)lightValue,
           (unsigned long)lightPercentage,
           streetLightStatus ? "ON" : "OFF");

    printf("------------------------------------\r\n");

    HAL_Delay(1000);
  }
}

/* Read analog light-sensor value */
static uint32_t Read_Light_Level(void)
{
  uint32_t adcValue;

  if (HAL_ADC_Start(&hadc1) != HAL_OK)
  {
    Error_Handler();
  }

  if (HAL_ADC_PollForConversion(&hadc1, 100) != HAL_OK)
  {
    Error_Handler();
  }

  adcValue = HAL_ADC_GetValue(&hadc1);

  HAL_ADC_Stop(&hadc1);

  return adcValue;
}

/* GPIO initialization */
static void MX_GPIO_Init(void)
{
  GPIO_InitTypeDef GPIO_InitStruct = {0};

  STREET_LED_CLK_ENABLE();
  __HAL_RCC_GPIOA_CLK_ENABLE();

  /* Configure PB3 as LED output */
  GPIO_InitStruct.Pin = STREET_LED_PIN;
  GPIO_InitStruct.Mode = GPIO_MODE_OUTPUT_PP;
  GPIO_InitStruct.Pull = GPIO_NOPULL;
  GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_LOW;

  HAL_GPIO_Init(STREET_LED_PORT, &GPIO_InitStruct);

  /* Initially switch streetlight OFF */
  HAL_GPIO_WritePin(
      STREET_LED_PORT,
      STREET_LED_PIN,
      GPIO_PIN_RESET);

  /* Configure PA0 as analog input */
  GPIO_InitStruct.Pin = LIGHT_SENSOR_PIN;
  GPIO_InitStruct.Mode = GPIO_MODE_ANALOG;
  GPIO_InitStruct.Pull = GPIO_NOPULL;

  HAL_GPIO_Init(LIGHT_SENSOR_PORT, &GPIO_InitStruct);
}

/* ADC initialization */
static void MX_ADC1_Init(void)
{
  ADC_ChannelConfTypeDef channelConfig = {0};

  __HAL_RCC_ADC1_CLK_ENABLE();

  hadc1.Instance = ADC1;
  hadc1.Init.OversamplingMode = DISABLE;
  hadc1.Init.ClockPrescaler = ADC_CLOCK_SYNC_PCLK_DIV2;
  hadc1.Init.Resolution = ADC_RESOLUTION_12B;
  hadc1.Init.SamplingTime = ADC_SAMPLETIME_39CYCLES_5;
  hadc1.Init.ScanConvMode = ADC_SCAN_DIRECTION_FORWARD;
  hadc1.Init.DataAlign = ADC_DATAALIGN_RIGHT;
  hadc1.Init.ContinuousConvMode = DISABLE;
  hadc1.Init.DiscontinuousConvMode = DISABLE;
  hadc1.Init.ExternalTrigConvEdge =
      ADC_EXTERNALTRIGCONVEDGE_NONE;
  hadc1.Init.ExternalTrigConv = ADC_SOFTWARE_START;
  hadc1.Init.DMAContinuousRequests = DISABLE;
  hadc1.Init.EOCSelection = ADC_EOC_SINGLE_CONV;
  hadc1.Init.Overrun = ADC_OVR_DATA_PRESERVED;
  hadc1.Init.LowPowerAutoWait = DISABLE;
  hadc1.Init.LowPowerFrequencyMode = DISABLE;
  hadc1.Init.LowPowerAutoPowerOff = DISABLE;

  if (HAL_ADC_Init(&hadc1) != HAL_OK)
  {
    Error_Handler();
  }

  channelConfig.Channel = LIGHT_SENSOR_CHANNEL;
  channelConfig.Rank = ADC_RANK_CHANNEL_NUMBER;

  if (HAL_ADC_ConfigChannel(
          &hadc1,
          &channelConfig) != HAL_OK)
  {
    Error_Handler();
  }
}

/* System clock configuration */
void SystemClock_Config(void)
{
  RCC_OscInitTypeDef RCC_OscInitStruct = {0};
  RCC_ClkInitTypeDef RCC_ClkInitStruct = {0};
  RCC_PeriphCLKInitTypeDef PeriphClkInit = {0};

  __HAL_PWR_VOLTAGESCALING_CONFIG(
      PWR_REGULATOR_VOLTAGE_SCALE1);

  RCC_OscInitStruct.OscillatorType =
      RCC_OSCILLATORTYPE_HSI;

  RCC_OscInitStruct.HSIState = RCC_HSI_ON;
  RCC_OscInitStruct.HSICalibrationValue =
      RCC_HSICALIBRATION_DEFAULT;

  RCC_OscInitStruct.PLL.PLLState = RCC_PLL_ON;
  RCC_OscInitStruct.PLL.PLLSource = RCC_PLLSOURCE_HSI;
  RCC_OscInitStruct.PLL.PLLMUL = RCC_PLLMUL_4;
  RCC_OscInitStruct.PLL.PLLDIV = RCC_PLLDIV_2;

  if (HAL_RCC_OscConfig(&RCC_OscInitStruct) != HAL_OK)
  {
    Error_Handler();
  }

  RCC_ClkInitStruct.ClockType =
      RCC_CLOCKTYPE_HCLK |
      RCC_CLOCKTYPE_SYSCLK |
      RCC_CLOCKTYPE_PCLK1 |
      RCC_CLOCKTYPE_PCLK2;

  RCC_ClkInitStruct.SYSCLKSource =
      RCC_SYSCLKSOURCE_PLLCLK;

  RCC_ClkInitStruct.AHBCLKDivider =
      RCC_SYSCLK_DIV1;

  RCC_ClkInitStruct.APB1CLKDivider =
      RCC_HCLK_DIV1;

  RCC_ClkInitStruct.APB2CLKDivider =
      RCC_HCLK_DIV1;

  if (HAL_RCC_ClockConfig(
          &RCC_ClkInitStruct,
          FLASH_LATENCY_1) != HAL_OK)
  {
    Error_Handler();
  }

  PeriphClkInit.PeriphClockSelection =
      RCC_PERIPHCLK_USART2;

  PeriphClkInit.Usart2ClockSelection =
      RCC_USART2CLKSOURCE_PCLK1;

  if (HAL_RCCEx_PeriphCLKConfig(
          &PeriphClkInit) != HAL_OK)
  {
    Error_Handler();
  }
}

/* USART2 initialization */
static void MX_USART2_UART_Init(void)
{
  GPIO_InitTypeDef GPIO_InitStruct = {0};

  __HAL_RCC_GPIOA_CLK_ENABLE();
  __HAL_RCC_USART2_CLK_ENABLE();

  GPIO_InitStruct.Pin = VCP_TX_PIN | VCP_RX_PIN;
  GPIO_InitStruct.Mode = GPIO_MODE_AF_PP;
  GPIO_InitStruct.Pull = GPIO_NOPULL;
  GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_VERY_HIGH;
  GPIO_InitStruct.Alternate = GPIO_AF4_USART2;

  HAL_GPIO_Init(GPIOA, &GPIO_InitStruct);

  huart2.Instance = USART2;
  huart2.Init.BaudRate = 115200;
  huart2.Init.WordLength = UART_WORDLENGTH_8B;
  huart2.Init.StopBits = UART_STOPBITS_1;
  huart2.Init.Parity = UART_PARITY_NONE;
  huart2.Init.Mode = UART_MODE_TX_RX;
  huart2.Init.HwFlowCtl = UART_HWCONTROL_NONE;
  huart2.Init.OverSampling = UART_OVERSAMPLING_16;
  huart2.Init.OneBitSampling =
      UART_ONE_BIT_SAMPLE_DISABLE;

  huart2.AdvancedInit.AdvFeatureInit =
      UART_ADVFEATURE_NO_INIT;

  if (HAL_UART_Init(&huart2) != HAL_OK)
  {
    Error_Handler();
  }
}

/* Error handler */
void Error_Handler(void)
{
  HAL_GPIO_WritePin(
      STREET_LED_PORT,
      STREET_LED_PIN,
      GPIO_PIN_RESET);

  while (1)
  {
  }
}

/* Redirect printf to USART2 */
#define STDOUT_FILENO 1
#define STDERR_FILENO 2

int _write(int file, uint8_t *ptr, int len)
{
  if ((file == STDOUT_FILENO) ||
      (file == STDERR_FILENO))
  {
    HAL_UART_Transmit(
        &huart2,
        ptr,
        len,
        HAL_MAX_DELAY);

    return len;
  }

  return -1;
}


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
