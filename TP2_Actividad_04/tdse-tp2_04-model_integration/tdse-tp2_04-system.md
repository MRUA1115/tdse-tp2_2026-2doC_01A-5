# Archivo de Depuración: tdse-tp2_04-actuator.md

**Confirmación de funcionamiento de la máquina de estados del Actuador tras ejecuciones de `app_update()`**

A continuación se detallan los valores almacenados y observados en la estructura de datos del actuador del LED (`task_actuator_dta_list[0]`) a lo largo del flujo completo de la barrera de acceso. La unidad de medida de la variable temporizadora `tick` es en **milisegundos [ms]**.

### 1. Estado Inicial (Barrera Cerrada en Reposo)
* **`state`:** `ST_LED_OFF`
* **`event`:** `EV_LED_OFF`
* **`flag`:** `false`
* **`tick`:** 0 [ms]
* **Comportamiento observado:** El LED permanece apagado en espera de comandos provenientes de la tarea del sistema.

### 2. Inicio de Subida / Apertura de Barrera (Presión del Botón B)
* **Evento de recepción:** `EV_LED_BLINK` (enviado por la tarea del sistema al transicionar a `ST_SYS_WAIT_FOR_BARRIER_OPENED`)
* **`state`:** `ST_LED_BLINK`
* **`flag`:** `false` (se consume el evento de la cola y se limpia la bandera)
* **`tick`:** 500 [ms] (se carga `DEL_LED_MAX` al ingresar y se recarga en cada conmutación)
* **Comportamiento observado:** El LED alternará su estado lógico mediante `HAL_GPIO_TogglePin()` cada vez que el contador decremente a 0 [ms], indicando visualmente la maniobra de subida.

### 3. Barrera Completamente Abierta (Fin del temporizador de subida)
* **Evento de recepción:** `EV_LED_ON`
* **`state`:** `ST_LED_ON`
* **`flag`:** `false`
* **`tick`:** 0 [ms]
* **Comportamiento observado:** El actuador interrumpe el parpadeo y fija el pin en estado activo (`LED_ON`), indicando al conductor que puede avanzar.

### 4. Inicio de Bajada / Cierre de Barrera (Presión del Botón C)
* **Evento de recepción:** `EV_LED_BLINK` (enviado por el sistema al transicionar a `ST_SYS_WAIT_FOR_BARRIER_CLOSED`)
* **`state`:** `ST_LED_BLINK`
* **`flag`:** `false`
* **`tick`:** 500 [ms]
* **Comportamiento observado:** El LED retoma el parpadeo periódico durante el descenso de la barrera.

### 5. Barrera Completamente Cerrada (Fin del temporizador de bajada)
* **Evento de recepción:** `EV_LED_OFF`
* **`state`:** `ST_LED_OFF`
* **`flag`:** `false`
* **`tick`:** 0 [ms]
* **Comportamiento observado:** El actuador apaga definitivamente el LED (`LED_OFF`), retornando la máquina de estados al reposo e iniciando nuevamente el ciclo.