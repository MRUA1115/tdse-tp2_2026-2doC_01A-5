# Consulta a Gemini — TP2 - Actividad 00 - Paso 12

**Prompt:** Analizar y explicar (en español), el funcionamiento del código fuente
contenido en los archivos adjuntos: `app.c`, `app_it.c`, `logger.c`, `logger.h`,
`systick.c` y `dwt.h`. Indicar el impacto de usar `LOGGER_INFO()` en la evolución
de las variables `g_app_tick_cnt`, `g_app_runtime_us` y `task_dta_list[index]`
(WCET, indicar unidad de medida) al ejecutar dicho código fuente desde su inicio
(`app_init()`) y en sucesivas ejecuciones del loop principal de la aplicación
(`app_update()`).

**Archivos adjuntos:** `app/src/app.c`, `app/src/app_it.c`, `app/src/logger.c`,
`app/inc/logger.h`, `app/src/systick.c`, `app/inc/dwt.h`.

**Respuesta:**

## `app.c` — el corazón del Cyclic Executive

Este archivo implementa la arquitectura de **Cyclic Executive** (Update by
Time Code, período = 1 mS) que orquesta las 3 tareas de la aplicación
(Sensor, System, Actuator).

- **`task_cfg_list[]`**: array `const` con punteros a las funciones
  `_init`/`_update` de cada tarea, en el orden en que se ejecutan
  (Sensor → System → Actuator). `TASK_QTY` se calcula con `sizeof()`, por lo
  que agregar una tarea nueva no requiere tocar el resto del código.
- **`task_dta_t`**: estructura con 4 métricas de tiempo de ejecución por
  tarea: `NOE` (Number Of Executions, contador de veces ejecutada), `LET`
  (Last Execution Time, tiempo de la última ejecución en **microsegundos**),
  `BCET` (Best-Case Execution Time, el menor tiempo registrado, en
  **microsegundos**) y `WCET` (Worst-Case Execution Time, el mayor tiempo
  registrado, en **microsegundos**).
- **`app_init()`**: inicializa el contador global `g_app_cnt` en 0, llama a
  `cycle_counter_init()` (activa el contador de ciclos DWT del núcleo
  Cortex-M3), recorre `task_cfg_list[]` llamando a `task_x_init()` de cada
  tarea, e inicializa `task_dta_list[index]` con `BCET = 1000` (valor
  inicial alto para que la primera medición real siempre sea menor) y
  `WCET = 0` (para que la primera medición real siempre sea mayor).
- **`app_update()`**: primero verifica de forma atómica (`CPSID i`/`CPSIE i`,
  deshabilitando/habilitando interrupciones) si `g_app_tick_cnt` (incrementado
  cada 1 mS por el SysTick, ver `app_it.c`) es mayor a 0. Si es así, lo
  decrementa y ejecuta un ciclo completo de todas las tareas: por cada tarea,
  resetea el contador de ciclos (`cycle_counter_reset()`), ejecuta
  `task_x_update()`, mide el tiempo transcurrido en microsegundos
  (`cycle_counter_get_time_us()`) y actualiza `NOE`, `LET`, `BCET`, `WCET` y
  el acumulado `g_app_runtime_us` (suma de los `LET` de todas las tareas en
  ese ciclo, es decir, el tiempo total que tomó ejecutar un ciclo completo
  de la aplicación, en microsegundos). Este proceso se repite en un `while`
  mientras sigan quedando "ticks" pendientes (por si `app_update()` se
  ejecutó más lento que el período de 1 mS y se acumularon varios ticks).

## `app_it.c` — la interfaz entre interrupciones y aplicación

- **`g_app_tick_cnt`** (`volatile uint32_t`): variable compartida entre la
  interrupción de SysTick y el loop principal — de ahí que su acceso en
  `app.c` esté protegido con `CPSID i`/`CPSIE i` (sección crítica), evitando
  condiciones de carrera.
- **`app_it_init()`**: inicializa `g_app_tick_cnt` en 0 de forma atómica.
- **`HAL_SYSTICK_Callback()`**: es invocado automáticamente por la HAL cada
  vez que ocurre la interrupción de SysTick (cada 1 mS), e incrementa
  `g_app_tick_cnt` — este es el mecanismo que le indica a `app_update()`
  cuándo "toca" ejecutar un nuevo ciclo de tareas.
- **`HAL_GPIO_EXTI_Callback()`**: callback de interrupción externa (EXTI),
  se dispara cuando cambia el estado del botón B1. En este proyecto base
  está vacío (comentario "Work to be done"), ya que la detección del botón
  se hace por *polling* dentro de `task_sensor_statechart()` y no por
  interrupción — el pin queda configurado como `GPIO_MODE_IT_RISING` en
  `main.c`, pero de momento no se usa activamente esa vía.

## `logger.c` / `logger.h` — sistema de trazas por semihosting

- **`LOGGER_CONFIG_ENABLE`** (1): habilita las macros de logging.
- **`LOGGER_CONFIG_MAXLEN`** (64): tamaño máximo del buffer de mensaje
  (`logger_msg_buffer_[64]`).
- **`LOGGER_CONFIG_USE_SEMIHOSTING`** (1): indica que los mensajes se envían
  por `printf()` vía semihosting (requiere que el debugger esté conectado y
  configurado en ese modo; si se compila con este flag en 0, `logger_log_print_()`
  no hace nada).
- **`LOGGER_INFO(...)`**: macro que arma un mensaje con `snprintf()`
  (formateado como `[info] <mensaje>\n`) y lo imprime con `printf()`. Todo el
  cuerpo de la macro está envuelto en `CPSID i`/`CPSIE i` para que el armado
  y envío del mensaje sea atómico (no se interrumpa a mitad de camino por
  otra interrupción que también use el logger).
- **`GET_NAME(var)`**: macro de "stringification" (`#var`) que convierte el
  nombre de una variable en su representación como cadena de texto, usada
  para imprimir en los logs tanto el nombre como el valor de una variable
  (por ejemplo, `LOGGER_INFO("%s = %lu", GET_NAME(index), index)` imprime
  `index = 3`).

### Impacto de usar `LOGGER_INFO()` en `g_app_tick_cnt`, `g_app_runtime_us` y `task_dta_list[index].WCET`

Cada llamada a `LOGGER_INFO()` internamente ejecuta `printf()` vía
semihosting, que es una operación **bloqueante y relativamente lenta**
(el núcleo debe comunicarse con el depurador/host a través del protocolo de
semihosting, deteniendo la ejecución del programa hasta que el host procesa
la solicitud). Esto tiene 3 efectos concretos:

1. **`g_app_tick_cnt`**: como todo el cuerpo de `LOGGER_INFO()` deshabilita
   las interrupciones (`CPSID i`), mientras se imprime un mensaje **no se
   incrementa `g_app_tick_cnt`** aunque haya pasado tiempo real (el SysTick
   sigue "sonando" pero su ISR queda pendiente hasta que se reactivan las
   interrupciones). Si se hace logging extensivo dentro de una tarea que se
   ejecuta muy seguido, esto puede provocar que se **acumulen varios ticks**
   pendientes de una sola vez, notándose luego como una ráfaga de ejecuciones
   de `app_update()` en el siguiente ciclo del `while`.
2. **`g_app_runtime_us`**: al ser la suma de los `LET` (tiempo de ejecución)
   de las 3 tareas en un ciclo, si alguna de esas tareas llama a
   `LOGGER_INFO()` durante su `_update()`, el tiempo que demora el semihosting
   **queda incluido dentro de la medición del DWT** (`cycle_counter_get_time_us()`),
   inflando artificialmente el `LET` de esa tarea en ese ciclo particular.
3. **`task_dta_list[index].WCET`** (medido en **microsegundos**): dado que
   `WCET` guarda el **mayor** `LET` observado hasta el momento, si en algún
   ciclo la tarea ejecuta `LOGGER_INFO()` (por ejemplo, durante `task_x_init()`,
   donde se llama repetidas veces para imprimir el estado inicial de cada
   índice), el tiempo de esa llamada de semihosting puede dominar por completo
   la medición de WCET, mostrando un valor mucho mayor al que tendría la
   tarea en producción real (sin logging activo). Por eso, en las mediciones
   de tiempos de ejecución "reales" del sistema (fuera de depuración), se
   recomienda deshabilitar `LOGGER_CONFIG_ENABLE` o reducir la cantidad de
   llamadas a `LOGGER_INFO()` dentro del camino crítico de las tareas.

## `systick.c` — retardo bloqueante basado en SysTick

Implementa `systick_delay_us()`, una función de espera **bloqueante** en
microsegundos que no depende de interrupciones: lee el registro `SysTick->VAL`
(contador regresivo de 24 bits del temporizador SysTick) al inicio, calcula
cuántas cuentas corresponden al retardo pedido (usando `SystemCoreClock`, la
frecuencia del reloj del núcleo) y espera en un `while` activo hasta que
transcurra ese número de cuentas, contemplando el caso de que el contador
haga *wrap-around* (vuelva a su valor de recarga `SysTick->LOAD`). No se usa
actualmente en el flujo principal de `app.c`/`app_it.c` (no aparece invocada
en estos archivos), pero está disponible como utilidad para retardos cortos
y precisos en otras partes del proyecto.

## `dwt.h` — medición de tiempo de ejecución por hardware

Provee funciones inline para usar el **DWT (Data Watchpoint and Trace)**, un
periférico de depuración presente en los núcleos Cortex-M3/M4 que incluye un
contador de ciclos de reloj (`DWT->CYCCNT`) de 32 bits:

- `cycle_counter_init()`: habilita el bloque de trace/debug (`DEMCR`) y
  arranca el contador de ciclos en 0.
- `cycle_counter_reset()`: reinicia el contador a 0 (usado por `app.c` antes
  de cada `task_x_update()`).
- `cycle_counter_get()`: devuelve el valor crudo del contador (en ciclos de
  reloj).
- `cycle_counter_get_time_us()`: convierte los ciclos acumulados a
  **microsegundos**, dividiendo por `SystemCoreClock / 1000000` (los ciclos
  por microsegundo según la frecuencia de reloj configurada). Esta es la
  función que usa `app.c` para calcular `LET` de cada tarea.

Este mecanismo es lo que permite medir con precisión de hardware (a nivel de
ciclos de reloj) cuánto tarda en ejecutarse cada tarea, sin el overhead de
usar un temporizador por software.


