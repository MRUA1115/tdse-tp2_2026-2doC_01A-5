## Depuración y Análisis de Estados - `task_sensor` (`task_sensor_dta_list[0]`)

Durante la depuración en STM32CubeIDE (`Paso 04`), se registró el comportamiento de la máquina de estados FSM y el filtrado antirrebote (*debounce*) inspeccionando la variable `task_sensor_dta_list[0]`.

### 1. Estado de Reposo Presionado
Al mantener presionado el botón antes de detener la ejecución con el *breakpoint* en `task_sensor_update()`:
* **`state`**: `ST_BTN_DOWN`
* **`event`**: `EV_BTN_DOWN`
* **`tick`**: `0` [ms]

### 2. Detección de Liberación (Transición)
Al soltar el botón y presionar **F8** (primer ciclo de ejecución de `app_update()`):
* **`event`**: Cambia a `EV_BTN_UP`
* **`state`**: Transiciona a `ST_BTN_RISING`
* **`tick`**: Se inicializa en `50` [ms] (`DEL_BTN_MAX`)

### 3. Conteo de Antirrebote (*Debounce*)
* Con cada ejecución subsecuente (**F8** / 1 ms por ciclo), el temporizador decrementa de a una unidad:
  $$\text{tick}_{n} = \text{tick}_{n-1} - 1 \text{ [ms]}$$
* Luego de presionar 50 veces **F8**, el contador alcanza el valor `0` [ms].

### 4. Confirmación del Estado Estable
* Con la siguiente ejecución de **F8** posterior a llegar a `tick = 0` [ms]:
  * **`state`**: Transiciona a `ST_BTN_UP` (estado final de reposo con el botón suelto).
  * Se valida la acción de liberación emitiendo la señal correspondiente hacia `task_system`.