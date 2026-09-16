# Consulta a Gemini — TP2 - Actividad 00 - Paso 09

**Prompt:** Analizar y explicar (en español), el funcionamiento del código fuente
contenido en los archivos adjuntos: `main.c`, `stm32f1xx_it.c` y
`startup_stm32f103rbtx.s`.

**Archivos adjuntos:** `Core/Src/main.c`, `Core/Src/stm32f1xx_it.c`,
`Core/Startup/startup_stm32f103rbtx.s`.

**Respuesta:**

## `startup_stm32f103rbtx.s`

Es el archivo de arranque en ensamblador generado por STM32CubeMX. Define la
**tabla de vectores de interrupción** (posiciones de memoria con las
direcciones de cada rutina de servicio de interrupción, ISR) y la rutina
`Reset_Handler`, que se ejecuta apenas el microcontrolador sale de reset:
inicializa la pila (`SP`), copia las variables inicializadas desde flash a
RAM (sección `.data`), pone en cero las variables no inicializadas (sección
`.bss`), y finalmente llama a `SystemInit()` y luego a `main()`. Todas las
ISR (como `SysTick_Handler`, `EXTI15_10_IRQHandler`, etc.) apuntan por
defecto a handlers "débiles" (`weak`) que hacen un loop infinito, salvo que
sean redefinidas en `stm32f1xx_it.c` (como ocurre en este proyecto).

## `main.c`

Es el punto de entrada de la aplicación (`int main(void)`), generado por
STM32CubeMX/CubeIDE, con secciones "USER CODE" donde se agrega código propio
sin que se pierda al regenerar el proyecto desde el `.ioc`. Su secuencia es:

1. `initialise_monitor_handles()` (solo si `LOGGER_CONFIG_USE_SEMIHOSTING`
   está habilitado): inicializa la comunicación de semihosting con el
   depurador, necesaria para que `printf()` (usado por `logger.c`) pueda
   imprimir mensajes en la consola de STM32CubeIDE.
2. `HAL_Init()`: inicializa la capa HAL (Hardware Abstraction Layer),
   configura el `SysTick` con un período de 1 mS (usado como base de tiempo
   de la librería HAL) y ajusta la prioridad de interrupciones.
3. `SystemClock_Config()`: configura el reloj del sistema (PLL a partir del
   oscilador interno HSI, con divisores para lograr la frecuencia de
   trabajo del microcontrolador STM32F103RB).
4. `MX_GPIO_Init()` / `MX_USART2_UART_Init()`: inicializan los periféricos
   generados por CubeMX — en este caso, el botón B1 como entrada con
   interrupción por flanco ascendente (`GPIO_MODE_IT_RISING`), el LED LD2
   como salida push-pull, y el USART2 a 115200 baudios (aunque en este
   proyecto la salida de logs real se hace por semihosting, no por UART).
5. `app_init()`: inicializa la capa de aplicación (las 3 tareas: Sensor,
   System, Actuator).
6. `while(1) { app_update(); }`: loop infinito (superloop) que llama
   continuamente a `app_update()`, la función que ejecuta el Cyclic
   Executive de la aplicación.

## `stm32f1xx_it.c`

Contiene las implementaciones reales de las ISR declaradas como "weak" en el
archivo de arranque, generadas y mantenidas por CubeMX:

- **`SysTick_Handler()`**: se ejecuta cada 1 mS (según la configuración del
  SysTick hecha por `HAL_Init()`). Llama a `HAL_IncTick()` (incrementa el
  contador interno de tick de HAL, usado por funciones como `HAL_GetTick()`
  y `HAL_Delay()`) y a `HAL_SYSTICK_IRQHandler()`, que a su vez invoca al
  callback `HAL_SYSTICK_Callback()` (implementado en `app_it.c`, ver
  `tdse-tp2_00-app.md`), donde la aplicación incrementa su propio contador
  de tick (`g_app_tick_cnt`).
- **`EXTI15_10_IRQHandler()`**: rutina de interrupción externa para el rango
  de pines EXTI 10 a 15 (donde cae el pin del botón B1 en la NUCLEO-F103RB).
  Llama a `HAL_GPIO_EXTI_IRQHandler(B1_Pin)`, que internamente invoca al
  callback `HAL_GPIO_EXTI_Callback()` (también implementado en `app_it.c`)
  cuando se detecta el flanco configurado.
- El resto de los handlers (`NMI_Handler`, `HardFault_Handler`,
  `MemManage_Handler`, `BusFault_Handler`, `UsageFault_Handler`) quedan con
  su comportamiento por defecto de CubeMX: un loop infinito, útil solo para
  detectar (con el debugger) que ocurrió una condición de error grave y
  detener la ejecución ahí mismo para inspeccionar la causa.

## Relación entre los tres archivos

`startup_stm32f103rbtx.s` prepara el entorno de ejecución y salta a
`main()`; `main.c` configura los periféricos y arranca el loop de la
aplicación; `stm32f1xx_it.c` conecta las interrupciones de hardware
(SysTick, EXTI) con la lógica de la aplicación definida en `app_it.c`,
cerrando el ciclo entre el hardware del microcontrolador y el firmware de
la aplicación (tareas Sensor/System/Actuator).
