# STM32 PWM LED Dimming

Demonstrates LED brightness control using hardware PWM on STM32 Nucleo F446RE.

## Hardware
- **Board:** STM32 Nucleo F446RE
- **LED:** Connect to PA8 (TIM1_CH1) via 330Ω resistor
- **Power:** USB

## Pulse Width Modulation

PWM is a digital electronics concept. It simulates an analog behavior by controlling the frequency of ON & OFF states. The human eye (response time ~50ms) perceives the *average* brightness, not individual pulses.

**Key principle:** Longer ON time = brighter LED.

```
Duty Cycle = 25%  →  ■□□□  (dim)
Duty Cycle = 50%  →  ■■□□  (medium)
Duty Cycle = 75%  →  ■■■□  (bright)
```

## PWM Configuration Logic

The timer counts from 0 to **ARR**. When count < **CCR**, output is HIGH. When count ≥ CCR, output is LOW.

**Think resolution-first:**
```
System Clock    = 72 MHz (72 million counts/second)
Resolution      = ARR + 1 (number of brightness steps)
PWM Frequency   = System Clock / Resolution
Duty Cycle      = CCR / Resolution
```

**Example (16-bit resolution, 25% duty):**
- Resolution = 65536 steps (16-bit)
- Frequency = 72MHz / 65536 ≈ 1098Hz
- ARR = 65535
- CCR = 16384 → Duty = 16384/65536 = 25%

**Example (8-bit resolution):**
- Resolution = 256 steps (8-bit)
- Frequency = 72MHz / 256 = 281.25kHz
- ARR = 255
- CCR = 64 → Duty = 64/256 = 25%

Higher resolution = smoother dimming, lower frequency.


## Code

```c
int pulse = 10;
int dir = 0;
while (1) {
    //	HAL_GPIO_TogglePin(CUST_LED_GPIO_Port, CUST_LED_Pin);
    //	HAL_Delay(3000);
		HAL_TIM_PWM_Start(&htim1, TIM_CHANNEL_1);

		__HAL_TIM_SET_COMPARE(&htim1, TIM_CHANNEL_1, pulse);
		HAL_Delay(2);

		pulse += dir;
		if(pulse >= 245) dir = -1;
		if(pulse <= 10) dir = 1;
}
```
