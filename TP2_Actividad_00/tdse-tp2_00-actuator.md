# Consulta a Gemini — TP2 - Actividad 00 - Paso 20

**Prompt:** Analizar y explicar (en español), el funcionamiento del código fuente
contenido en los archivos adjuntos: `task_actuator_attribute.h`,
`task_actuator.c` y `task_actuator_interface.c`.

**Archivos adjuntos:** `app/inc/task_actuator_attribute.h`,
`app/src/task_actuator.c`, `app/src/task_actuator_interface.c`.

**Respuesta:**

## `task_actuator_attribute.h` — tipos del statechart de Actuator

- **`task_actuator_ev_t`**: eventos — `EV_LED_IDLE` y `EV_LED_ACTIVE`,
  enviados por el Task System.
- **`task_actuator_st_t`**: estados — `ST_LED_IDLE` (apagado) y
  `ST_LED_ACTIVE` (encendido). Es una versión simplificada de 2 estados del
  `Actuator Statechart` diseñado en el TP1 (que contemplaba 4 estados con
  fases de parpadeo de arranque/parada); en este proyecto de codificación
  base, el actuador conmuta directamente entre apagado y encendido sin
  transición de parpadeo intermedia.
- **`task_actuator_id_t`**: identificador de cada instancia de actuador
  (en el proyecto base, solo `ID_LED_A`; se extiende a `ID_LED_A`/`ID_LED_B`
  en la Actividad 05).
- **`task_actuator_cfg_t`**: configuración fija con el identificador, puerto
  y pin GPIO, los niveles lógicos de encendido/apagado (`led_on`/`led_off`)
  y un `tick_max` reservado para futura temporización.
- **`task_actuator_dta_t`**: datos en tiempo de ejecución — tick, estado,
  evento y flag de "evento pendiente".

## `task_actuator.c` — implementación del statechart de Actuator

- **`task_actuator_cfg_list[]`**: array de configuración, una entrada por
  cada LED (en el proyecto base, solo `ID_LED_A` asociado a `LD2_Pin`).
  `ACTUATOR_CFG_QTY`/`ACTUATOR_DTA_QTY` se calculan dinámicamente con
  `sizeof()`, permitiendo agregar más actuadores sin tocar el resto del
  código (como se hace en la Actividad 05).
- **`task_actuator_init()`**: inicializa el estado de cada instancia en
  `ST_LED_IDLE`, el evento en `EV_LED_IDLE`, el flag en `false`, y **apaga
  físicamente el LED** con `HAL_GPIO_WritePin(..., led_off)` — garantiza un
  estado inicial conocido y consistente entre el software y el hardware.
- **`task_actuator_update()`**: recorre todas las instancias y ejecuta
  `task_actuator_statechart(index)` para cada una.
- **`task_actuator_statechart(index)`**: según el estado actual (`switch`):
  - En `ST_LED_IDLE`: si `flag` es verdadero y el evento es `EV_LED_ACTIVE`,
    se enciende el LED físicamente (`HAL_GPIO_WritePin(..., led_on)`), se
    limpia el flag y se transiciona a `ST_LED_ACTIVE`.
  - En `ST_LED_ACTIVE`: si `flag` es verdadero y el evento es `EV_LED_IDLE`,
    se apaga el LED (`HAL_GPIO_WritePin(..., led_off)`), se limpia el flag y
    se transiciona a `ST_LED_IDLE`.
  - `default`: estado de recuperación, resetea a `ST_LED_IDLE`.

## `task_actuator_interface.c` — interfaz de comunicación

Implementa `put_event_task_actuator(event, identifier)`: a diferencia del
Task System (que usa una cola FIFO compartida entre múltiples emisores),
el Actuator recibe comandos de forma **directa**, escribiendo el evento y
levantando el flag en la posición exacta del array
`task_actuator_dta_list[identifier]` correspondiente al LED destino. Esto es
posible porque el único emisor de eventos hacia el Actuator es el Task
System (un solo "productor" por actuador en cada instante), a diferencia de
los Sensores, que pueden generar eventos en cualquier momento y de forma
concurrente entre sí, justificando el uso de una cola en ese caso.

## Resumen de la cadena completa Sensor → System → Actuator

1. El **Sensor** detecta por *polling* el cambio de estado físico de un
   botón y encola un evento (`EV_SYS_ACTIVE`/`EV_SYS_IDLE`) hacia el System
   mediante `put_event_task_system()`.
2. El **System**, en su siguiente ciclo de ejecución, desencola el evento
   con `get_event_task_system()`, evalúa la transición correspondiente en su
   propio statechart y, si corresponde, notifica directamente al Actuator
   con `put_event_task_actuator()`.
3. El **Actuator**, en su siguiente ciclo, procesa el evento recibido y
   actúa sobre el hardware (`HAL_GPIO_WritePin()`) para encender o apagar el
   LED físico.

Todo este flujo ocurre dentro de un mismo ciclo de 1 mS de `app_update()`
(Sensor, System y Actuator se ejecutan en ese orden en cada tick), por lo
que en la práctica la reacción del sistema ante un botón presionado se
percibe como instantánea para un observador humano.
