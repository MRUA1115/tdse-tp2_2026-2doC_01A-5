# TP2 - Actividad 05 - 8vo Proyecto - 2 Actuator Statechart

Proyecto `tdse-tp2_05-model_integration`, extendido desde `TP2_Actividad_04`
para soportar **2 actuadores (LEDs)** en lugar de 1.

**Placa utilizada:** NUCLEO-F103RB.

## Hardware adicional requerido
- 1 (UN) Módulo LED RGB - Arduino, o 1 (UN) Módulo LED Mini Semáforo -
  Arduino.

## Statechart implementado (`task_actuator.c` / `task_actuator_attribute.h`)

Mismo diagrama oficial `task_actuator.jpg` de la Actividad 04 (3 estados:
OFF / ON / BLINK), aplicado a **2 instancias independientes**
(`ID_LED_A`, `ID_LED_B`), cada una con su propio `state`/`event`/`tick` en
`task_actuator_dta_list[]`, recorridas con un `for` en
`task_actuator_update()`.

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

## Cambios respecto de la Actividad 04

- `board.h`: se agregó `LED_B_PIN`/`LED_B_PORT` (PA10, pin D2) — **pendiente
  confirmar en la placa física que este pin esté libre** antes de conectar
  el módulo LED externo. Verificar también el esquema eléctrico de la
  NUCLEO-F103RB (activo alto/bajo según el pin elegido).
- `task_actuator_attribute.h`: se agregó el identificador `ID_LED_B` al enum
  `task_actuator_id_t`, y se actualizaron eventos/estados a los 3 reales
  del diagrama (`EV_LED_OFF/ON/BLINK`, `ST_LED_OFF/ON/BLINK`).
- `task_actuator.c`: se agregó una entrada más al array
  `task_actuator_cfg_list[]` para el LED B. Como `ACTUATOR_CFG_QTY` se
  calcula dinámicamente con `sizeof()`, el resto del código (`init`,
  `update`) no requiere cambios; el `switch/case` del statechart sí se
  actualizó a los 3 estados reales (ver tabla arriba).
- `task_system.c`: se modificaron las 2 transiciones del System para
  notificar a **ambos** LEDs (`ID_LED_A` e `ID_LED_B`) en cada cambio de
  estado, usando los eventos reales `EV_LED_ON`/`EV_LED_OFF`.

## Pendiente
- Confirmar en la placa NUCLEO-F103RB qué pin del conector digital (D0-D15)
  está realmente libre para `LED_B_PIN`, y ajustar en `board.h` si hiciera
  falta.
- Configurar el pin elegido en el archivo `.ioc` como `GPIO_Output`
  push-pull.
- Conectar físicamente el módulo LED RGB/mini semáforo y verificar que
  ambos LEDs enciendan/apaguen en simultáneo junto con LD2.
- Compilar y depurar en STM32CubeIDE.
- Luego de varias ejecuciones de `app_update()`, leer y almacenar los
  valores de `task_dta_list[0]` (Task Sensor) en `tdse-tp2_05-actuator.md`,
  según lo pedido por la guía.
