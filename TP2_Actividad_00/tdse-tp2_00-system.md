# Consulta a Gemini — TP2 - Actividad 00 - Paso ~18

**Prompt:** Analizar y explicar (en español), el funcionamiento del código fuente
contenido en los archivos adjuntos: `task_system_attribute.h`,
`task_actuator_attribute.h`, `task_system.c`, `task_system_interface.c` y
`task_actuator_interface.c`.

**Archivos adjuntos:** `app/inc/task_system_attribute.h`,
`app/inc/task_actuator_attribute.h`, `app/src/task_system.c`,
`app/src/task_system_interface.c`, `app/src/task_actuator_interface.c`.

**Respuesta:**

## `task_system_attribute.h` — tipos del statechart de System

- **`task_system_ev_t`**: eventos — `EV_SYS_IDLE` y `EV_SYS_ACTIVE`, enviados
  por el/los Sensor(es).
- **`task_system_st_t`**: estados — `ST_SYS_IDLE` y `ST_SYS_ACTIVE`,
  correspondientes al modelo `System Statechart` del TP1.
- **`task_system_dta_t`**: estructura de datos con `tick`, `state`, `event` y
  un flag `flag` (indica si hay un evento nuevo pendiente de procesar).

## `task_system.c` — implementación del statechart de System

- **`task_system_mode_t`**: enum con un único modo `NORMAL` (más `MODE_QTY`
  como centinela para calcular `SYSTEM_DTA_QTY`), pensado para poder
  extender la tarea a múltiples "sistemas" independientes en el futuro
  (aunque en este TP solo se usa un modo).
- **`g_task_system_mode`**: variable global que indica el modo activo.
- **`task_system_init()`**: llama a `init_event_task_system()` (inicializa la
  cola de eventos), fija el modo en `NORMAL`, e inicializa el estado en
  `ST_SYS_IDLE`, el evento en `EV_SYS_IDLE` y el flag en `false`.
- **`task_system_update()`**: según `g_task_system_mode`, llama a
  `task_system_normal_statechart()` (único caso implementado; `default`
  fuerza el modo a `NORMAL` como recuperación).
- **`task_system_normal_statechart()`**: primero verifica si hay algún
  evento pendiente en la cola con `any_event_task_system()`; si lo hay,
  marca `flag = true` y obtiene el evento con `get_event_task_system()`.
  Luego, según el estado actual:
  - En `ST_SYS_IDLE`: si `flag` es verdadero y el evento es `EV_SYS_ACTIVE`,
    se notifica al Actuator con `put_event_task_actuator(EV_LED_ACTIVE, ID_LED_A)`
    y se transiciona a `ST_SYS_ACTIVE`.
  - En `ST_SYS_ACTIVE`: si `flag` es verdadero y el evento es `EV_SYS_IDLE`,
    se notifica al Actuator con `put_event_task_actuator(EV_LED_IDLE, ID_LED_A)`
    y se transiciona a `ST_SYS_IDLE`.
  - En ambos casos, `flag` se pone en `false` apenas se consume el evento,
    evitando procesarlo dos veces.

## `task_system_interface.c` — cola de eventos de System

Implementa una **cola circular (FIFO)** de hasta 16 eventos
(`QUEUE_LENGTH = 16`), usada para comunicar de forma asíncrona los Sensores
con el System:

- **`init_event_task_system()`**: inicializa `head`, `tail` y `count` en 0, y
  llena el array `queue[]` con el valor centinela `EMPTY` (255).
- **`put_event_task_system(event)`**: encola un evento en la posición `head`
  e incrementa `head` (con wrap-around al llegar a `QUEUE_LENGTH`). También
  incrementa `count`.
- **`get_event_task_system()`**: desencola el evento en la posición `tail`,
  lo marca como `EMPTY`, incrementa `tail` (con wrap-around) y decrementa
  `count`.
- **`any_event_task_system()`**: indica si hay eventos pendientes comparando
  `head != tail`. *(Nota de diseño: al no usar directamente `count > 0`, esta
  función podría dar un falso negativo únicamente si la cola llegase a
  llenarse por completo con exactamente 16 eventos sin consumir, algo que no
  ocurre en el uso normal de esta aplicación dado el bajo volumen de eventos
  generado.)*

Esta cola es la que permite que **cualquier cantidad de sensores** (1 en la
Actividad 01, 3 en la Actividad 02) notifiquen al mismo System sin que las
tareas tengan que ejecutarse de forma sincronizada.

## `task_actuator_interface.c` — enrutamiento de eventos al Actuator

A diferencia del System (que usa una cola compartida), el Actuator recibe
eventos de forma **directa e indexada**:

- **`put_event_task_actuator(event, identifier)`**: escribe directamente el
  `event` y pone `flag = true` en la posición `task_actuator_dta_list[identifier]`
  correspondiente. Esto permite que el System notifique a un LED específico
  (por su `identifier`) sin necesidad de una cola FIFO, ya que en general
  cada actuador recibe comandos puntuales y no hay riesgo de perder eventos
  en tránsito (a diferencia de los sensores, que pueden generar eventos en
  cualquier momento de forma asíncrona respecto al System).

Este diseño de interfaz "directa por índice" es el que permite que en la
Actividad 05 (2 actuadores) el System simplemente llame dos veces a
`put_event_task_actuator()` (una con `ID_LED_A` y otra con `ID_LED_B`) para
comandar ambos LEDs en simultáneo.
