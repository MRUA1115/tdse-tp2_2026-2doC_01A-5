# TP2 - Actividad 04 - 7mo Proyecto - 1 Actuator Statechart

Proyecto `tdse-tp2_04-model_integration`, copiado desde `TP2_Actividad_03`,
con foco en el modelo **Actuator** para **1 LED** (LD2 de la placa NUCLEO).

**Placa utilizada:** NUCLEO-F103RB.

## Statechart implementado (`task_actuator.c` / `task_actuator_attribute.h`)

Corresponde al modelo `Actuator Statechart` diseñado en el TP1.

| Estado | Evento | Acción | Próximo estado |
| :---: | :---: | :---: | :---: |
| ST_LED_IDLE | EV_LED_ACTIVE | HAL_GPIO_WritePin(..., led_on) | ST_LED_ACTIVE |
| ST_LED_ACTIVE | EV_LED_IDLE | HAL_GPIO_WritePin(..., led_off) | ST_LED_IDLE |

El Actuator recibe eventos desde el Task System (Actividad 03) para encender
o apagar el LED LD2 integrado en la placa.

## Pendiente
- Confirmar en la placa física el correcto encendido/apagado del LED LD2 al
  presionar cualquiera de los 3 botones.
- Compilar y depurar en STM32CubeIDE.
