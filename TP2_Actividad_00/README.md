# TP2 - Actividad 00 - Problem approach

Proyecto base `model_integration`, descargado de la cátedra, con la
arquitectura completa de tareas ya funcional:

- **Task Sensor** (`task_sensor.c`): 1 sensor (botón B1 de la placa NUCLEO),
  statechart con 2 estados (`ST_BTN_IDLE`, `ST_BTN_ACTIVE`).
- **Task System** (`task_system.c`): 1 sistema (modo `NORMAL`), statechart
  con 2 estados (`ST_SYS_IDLE`, `ST_SYS_ACTIVE`), recibe eventos del Sensor
  mediante una cola de eventos (`event_task_system_queue`).
- **Task Actuator** (`task_actuator.c`): 1 actuador (LED LD2 de la placa
  NUCLEO), statechart con 2 estados (`ST_LED_IDLE`, `ST_LED_ACTIVE`).

**Placa utilizada:** NUCLEO-F103RB.

## Arquitectura de la aplicación (`app.c`)

- Cyclic Executive con período de actualización de 1 mS (`SysTick_Handler` /
  `HAL_SYSTICK_Callback`), disparado desde `g_app_tick_cnt`.
- `app_init()`: inicializa las 3 tareas (Sensor, System, Actuator) y las
  interrupciones de la aplicación.
- `app_update()`: recorre el array `task_cfg_list[]` ejecutando
  `task_x_update()` de cada tarea en orden (Sensor → System → Actuator),
  midiendo NOE (número de ejecuciones), LET (tiempo de la última ejecución),
  BCET y WCET (mejor/peor caso de tiempo de ejecución) por tarea.

## Análisis de código fuente (Gemini)

Se generaron los siguientes archivos con el análisis pedido por la guía:

| Archivo | Contenido |
| :--- | :--- |
| `tdse-tp2_00.md` | Respuesta genérica sobre el enfoque del TP (Paso 07) |
| `tdse-tp2_00-main.md` | Análisis de `main.c`, `stm32f1xx_it.c`, `startup_stm32f103rbtx.s` (Paso 09) |
| `tdse-tp2_00-app.md` | Análisis de `app.c`, `app_it.c`, `logger.c/h`, `systick.c`, `dwt.h` (Paso 12) |
| `tdse-tp2_00-sensor.md` | Análisis de `task_sensor_attribute.h`, `task_sensor.c` (Paso 15) |
| `tdse-tp2_00-system.md` | Análisis de System + interfaces (Paso ~18) |
| `tdse-tp2_00-actuator.md` | Análisis de Actuator + interfaces (Paso 20) |

## Pendiente
- Compilar y depurar el proyecto en STM32CubeIDE con la placa conectada.
- Confirmar mediante depuración real los valores de `task_dta_list[index]`
  (NOE, LET, BCET, WCET) mencionados en `tdse-tp2_00-app.md`.
