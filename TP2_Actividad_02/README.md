# TP2 - Actividad 02 - 5to Proyecto - 3 Sensor Statechart

Proyecto `tdse-tp2_02-model_integration`, extendido desde
`TP2_Actividad_01` para soportar **3 sensores** en lugar de 1.

**Placa utilizada:** NUCLEO-F103RB.

## Hardware adicional requerido
- 1 (UN) Teclado membrana 1x4 teclas - Arduino, o 1 (UN) Dip switch de 4
  contactos.

## Cambios respecto de la Actividad 01

- `board.h`: se agregaron `BTN_B_PIN`/`BTN_B_PORT` (PA3, pin D1) y
  `BTN_C_PIN`/`BTN_C_PORT` (PA2, pin D0) — **pendiente confirmar en la placa
  física que estos pines estén libres** antes de conectar el hardware.
- `task_sensor_attribute.h`: se agregaron los identificadores `ID_BTN_B` e
  `ID_BTN_C` al enum `task_sensor_id_t`.
- `task_sensor.c`: se agregaron 2 entradas más al array
  `task_sensor_cfg_list[]`, una por cada botón nuevo. Como `SENSOR_CFG_QTY`
  se calcula dinámicamente con `sizeof()`, el resto del código (`init`,
  `update`, `statechart`) no requiere cambios — ya itera sobre todos los
  sensores configurados.
- Los 3 sensores excitan el mismo `Task System` (1 solo sistema), reutilizando
  los eventos genéricos `EV_SYS_IDLE`/`EV_SYS_ACTIVE`.

## Pendiente
- Confirmar en la placa NUCLEO-F103RB qué pines del conector digital (D0-D15)
  están realmente libres, y ajustar `BTN_B_PIN`/`BTN_C_PIN` en `board.h` si
  hiciera falta.
- Configurar los pines elegidos en el archivo `.ioc` como `GPIO_Input` con
  pull-up interno.
- Conectar físicamente el dip switch/teclado membrana y verificar el
  funcionamiento con los 3 botones.
- Compilar y depurar en STM32CubeIDE.
