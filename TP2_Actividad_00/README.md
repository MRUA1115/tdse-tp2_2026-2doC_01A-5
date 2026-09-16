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

## Pendiente
- Compilar y depurar el proyecto en STM32CubeIDE con la placa conectada.
- Consultar a Gemini el análisis del código fuente de `app.c`, `task_sensor.c`,
  `task_system.c` y `task_actuator.c`, y almacenar la respuesta en un archivo
  `gemini_00.txt` dentro de esta carpeta (paso pedido por la guía, requiere
  interacción manual con la herramienta).
