# Consulta a Gemini — TP2 - Actividad 00 - Paso 07

**Prompt:** ¿Puedes ayudarme a realizar un Trabajo Práctico sobre Codificación en C de
Diagramas de Estado?

**Respuesta:**

¡Claro que sí! Un Trabajo Práctico de "Codificación en C de Diagramas de Estado"
consiste en traducir un modelo de máquina de estados (statechart), previamente
diseñado en una herramienta gráfica (como Itemis Create), a código C ejecutable
en un microcontrolador embebido. El objetivo es que el comportamiento dinámico
descripto en el diagrama (estados, eventos, transiciones, guardas y acciones)
se implemente fielmente en firmware.

El patrón de codificación más común para este tipo de ejercicios es el
siguiente:

1. **Enumerar estados y eventos**: se definen `typedef enum` para los estados
   posibles de la máquina y para los eventos/señales que la excitan.
2. **Separar configuración (cfg) de datos (dta)**: se usan dos arrays de
   estructuras — uno `const` con la configuración fija de cada instancia
   (por ejemplo, a qué pin GPIO está asociado un sensor), y otro variable con
   los datos de ejecución (estado actual, evento, contador de tiempo).
3. **Función `_init()`**: inicializa el array de datos con el estado inicial
   de la máquina.
4. **Función `_update()`**: se llama periódicamente (por ejemplo, cada 1 mS
   desde un Cyclic Executive disparado por el SysTick) y recorre todas las
   instancias configuradas, ejecutando la función de statechart de cada una.
5. **Función `_statechart()`**: contiene un `switch` sobre el estado actual,
   con un `case` por cada estado y, dentro de cada uno, tantos `if` como
   transiciones salientes tenga ese estado (con sus guardas si corresponde).
   Al cumplirse una condición, se ejecuta la acción asociada (por ejemplo,
   escribir un GPIO o encolar un evento hacia otra tarea) y se actualiza la
   variable de estado.
6. **Comunicación entre statecharts**: cuando un modelo (por ejemplo, un
   Sensor) necesita notificar a otro (por ejemplo, un System), se usa un
   mecanismo de cola de eventos (`put_event_task_x()` / `get_event_task_x()`)
   para desacoplar la ejecución de ambas tareas sin bloquear al emisor.

Este patrón es el que se aplica en el proyecto `model_integration` de este
TP2, con tres statecharts colaborando: **Sensor** (detecta la entrada física),
**System** (lógica central de decisión) y **Actuator** (controla la salida
física), conectados mediante colas de eventos y ejecutados secuencialmente en
cada ciclo de 1 mS desde `app_update()`.
