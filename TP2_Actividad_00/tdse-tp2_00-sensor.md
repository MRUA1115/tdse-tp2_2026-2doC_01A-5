# Consulta a Gemini — TP2 - Actividad 00 - Paso 15

**Prompt:** Analizar y explicar (en español), el funcionamiento del código fuente
contenido en los archivos adjuntos: `task_sensor_attribute.h` y `task_sensor.c`.

**Archivos adjuntos:** `app/inc/task_sensor_attribute.h`, `app/src/task_sensor.c`.

**Respuesta:**

## `task_sensor_attribute.h` — tipos del statechart de Sensor

Define los tipos usados por la tarea Sensor, correspondientes al modelo
`Sensor Statechart` del TP1, aunque simplificado a 2 estados (a diferencia
del modelo de 4 estados con debounce del TP1, en este proyecto base el
Sensor tiene una versión más directa de 2 estados, ya que el filtrado de
rebote se delega al hardware/HAL):

- **`task_sensor_ev_t`**: eventos posibles — `EV_BTN_UP` (botón liberado) y
  `EV_BTN_DOWN` (botón presionado).
- **`task_sensor_st_t`**: estados posibles — `ST_BTN_IDLE` (inactivo) y
  `ST_BTN_ACTIVE` (activo).
- **`task_sensor_id_t`**: identificador de cada instancia de sensor
  configurada (en el proyecto base, solo `ID_BTN_A`).
- **`task_sensor_cfg_t`**: estructura de **configuración** (constante), con
  el identificador, el puerto y pin GPIO asociado, el nivel lógico que
  indica "presionado" (`pressed`), un valor máximo de tick (`tick_max`,
  reservado para uso futuro de temporización) y los dos eventos que se
  envían al Task System al pasar a cada estado (`signal_up`/`signal_down`).
- **`task_sensor_dta_t`**: estructura de **datos** (variable en tiempo de
  ejecución), con el tick actual, el estado actual y el último evento
  detectado.

## `task_sensor.c` — implementación del statechart

- **`task_sensor_cfg_list[]`**: array de configuración con una sola entrada
  para `ID_BTN_A`, asociada al pin del botón B1 (`BTN_A_PORT`/`BTN_A_PIN`
  definidos en `board.h`), indicando que el nivel `BTN_A_PRESSED` corresponde
  a "presionado", y que al presionar/soltar se deben enviar los eventos
  `EV_SYS_IDLE`/`EV_SYS_ACTIVE` hacia el Task System.
- **`SENSOR_CFG_QTY`/`SENSOR_DTA_QTY`**: se calculan dinámicamente con
  `sizeof(task_sensor_cfg_list) / sizeof(task_sensor_cfg_t)`, por lo que
  agregar más sensores al array (como se hace en la Actividad 02, con 3
  sensores) no requiere modificar el resto del código.
- **`task_sensor_init()`**: recorre todas las instancias configuradas e
  inicializa su estado en `ST_BTN_IDLE` y su evento en `EV_BTN_UP`,
  imprimiendo por log el estado inicial de cada índice.
- **`task_sensor_update()`**: recorre todas las instancias y llama a
  `task_sensor_statechart(index)` para cada una.
- **`task_sensor_statechart(index)`**: es la función principal del
  statechart. Primero **lee el estado físico del pin** con
  `HAL_GPIO_ReadPin()` y lo compara contra el nivel `pressed` configurado,
  determinando el evento actual (`EV_BTN_DOWN` o `EV_BTN_UP`). Luego, según
  el estado actual (`switch`):
  - En `ST_BTN_IDLE`: si el evento es `EV_BTN_DOWN`, se llama a
    `put_event_task_system(signal_down)` (encola `EV_SYS_ACTIVE` para el
    System) y se transiciona a `ST_BTN_ACTIVE`.
  - En `ST_BTN_ACTIVE`: si el evento es `EV_BTN_UP`, se llama a
    `put_event_task_system(signal_up)` (encola `EV_SYS_IDLE`) y se
    transiciona a `ST_BTN_IDLE`.
  - `default`: estado de recuperación, resetea a `ST_BTN_IDLE`.

Este statechart corresponde a una versión simplificada del `Sensor
Statechart` diseñado en el TP1: en el TP1 se contemplaban 4 estados
(`UP`/`FALLING`/`DOWN`/`RISING`) con un temporizador de debounce explícito
(`DEL_BTN_NAME`/`T_DEBOUNCE`); en este proyecto de codificación, el
antirrebote se resuelve mediante *polling* directo del pin (leyendo su
estado en cada ciclo de 1 mS), lo que en la práctica logra un efecto similar
sin necesidad de modelar los estados transitorios `FALLING`/`RISING`
explícitamente en el código C.
