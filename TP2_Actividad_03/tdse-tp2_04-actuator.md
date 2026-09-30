# Archivo de Depuración: tdse-tp2_04-xxxxx.md

**Confirmación de funcionamiento de la máquina de estados del Sistema tras ejecuciones de `app_update()`**

A continuación se detallan los valores almacenados y observados en la estructura de datos del sistema (`task_dta_list[0]`) durante el flujo completo de la barrera de acceso. La unidad de medida de la variable temporizadora `tick` es en **milisegundos [ms]**[cite: 42].

### 1. Estado Inicial (Esperando llegada del vehículo)
*   **`state`:** `ST_SYS_WAIT_FOR_CAR_ARRIEVE`
*   **`event`:** `EV_SYS_IDLE`
*   **`tick`:** 0 [ms]
*   **Comportamiento observado:** El sistema ignora cualquier entrada que no sea el Botón A. Mientras espera, el evento se mantiene en `EV_SYS_IDLE`.

### 2. Detección del Vehículo (Presión del Botón A)
*   **Evento de transición:** `EV_SYS_CAMERA`
*   **`state`:** `ST_SYS_WAIT_FOR_BUTTON_PRESSED`
*   **`event`:** `EV_SYS_IDLE` (Vuelve al estado de reposo mientras espera la siguiente acción).
*   **`tick`:** 0 [ms]

### 3. Emisión de Ticket (Presión del Botón B)
*   **Evento de transición:** `EV_SYS_BUTTON` (El evento se procesa en el sistema luego de que transcurren los 50 ticks de *debounce* en la máquina de estados del sensor correspondiente).
*   **`state`:** `ST_WAIT_FOR_BARRIER_OPEN`
*   **`event`:** `EV_SYS_IDLE`
*   **`tick`:** 500 [ms] (Se carga el temporizador para simular la demora mecánica de apertura).

### 4. Barrera Abierta (Fin de temporizador de apertura)
*   **Evento de transición:** Automática (cuando `tick` llega a 0).
*   **`state`:** `ST_WAIT_FOR_CAR_LEAVES`
*   **`event`:** `EV_SYS_IDLE`
*   **`tick`:** 0 [ms]

### 5. Paso del Vehículo (Presión del Botón C)
*   **Evento de transición:** `EV_SYS_SENSOR_COIL`
*   **`state`:** `ST_WAIT_FOR_BARRIER_CLOSE`
*   **`event`:** `EV_SYS_IDLE`
*   **`tick`:** 500 [ms] (Se carga el temporizador para simular la demora mecánica de cierre).

### 6. Fin del Ciclo (Fin de temporizador de cierre)
*   **Evento de transición:** Automática (cuando `tick` llega a 0).
*   **`state`:** `ST_SYS_WAIT_FOR_CAR_ARRIEVE`
*   **`event`:** `EV_SYS_IDLE`
*   **`tick`:** 0 [ms]
*   **Comportamiento observado:** El sistema reinicia el ciclo correctamente y queda a la espera de un nuevo vehículo.
