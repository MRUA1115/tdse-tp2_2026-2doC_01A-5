# TP2 - Actividad 04 - 7mo Proyecto - 1 Actuator Statechart

Proyecto `tdse-tp2_04-model_integration`, copiado desde `TP2_Actividad_03`,
con foco en el modelo **Actuator** para **1 LED** (LD2 de la placa NUCLEO).

**Placa utilizada:** NUCLEO-F103RB.

## Statechart implementado (`task_actuator.c` / `task_actuator_attribute.h`)

Corresponde al diagrama oficial `task_actuator.jpg` de la cátedra (3
estados: OFF / ON / BLINK, con parpadeo controlado por tick).

| Estado | Evento [Guarda] | Acción | Próximo estado |
| :---: | :---: | :---: | :---: |
| ST_LED_OFF | EV_LED_ON | HAL_GPIO_WritePin(..., led_on) | ST_LED_ON |
| ST_LED_OFF | EV_LED_BLINK | tick=tick_max; HAL_GPIO_WritePin(..., led_on) | ST_LED_BLINK |
| ST_LED_ON | EV_LED_OFF | HAL_GPIO_WritePin(..., led_off) | ST_LED_OFF |
| ST_LED_ON | EV_LED_BLINK | tick=tick_max; HAL_GPIO_WritePin(..., led_off) | ST_LED_BLINK |
| ST_LED_BLINK | EV_LED_ON | HAL_GPIO_WritePin(..., led_on) | ST_LED_ON |
| ST_LED_BLINK | EV_LED_OFF | HAL_GPIO_WritePin(..., led_off) | ST_LED_OFF |
| ST_LED_BLINK | [tick > 0] | tick-- | ST_LED_BLINK |
| ST_LED_BLINK | [tick == 0] | tick=tick_max; HAL_GPIO_TogglePin(...) | ST_LED_BLINK |

El Actuator recibe eventos desde el Task System (Actividad 04, versión
simplificada del System que solo excita `EV_LED_ON`/`EV_LED_OFF` mientras
el botón esté presionado) para encender o apagar el LED LD2 integrado en
la placa.

## Pendiente
- Confirmar en la placa física el correcto encendido/apagado del LED LD2 al
  presionar el botón B1.
- Compilar y depurar en STM32CubeIDE.
