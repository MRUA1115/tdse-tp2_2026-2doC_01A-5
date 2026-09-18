# TP2 - Actividad 01 - 4to Proyecto - 1 Sensor Statechart

Proyecto `tdse-tp2_01-model_integration`, copiado desde
`TP2_Actividad_00` y renombrado según lo pedido por la guía.

Codifica en C el statechart de **Sensor** para 1 botón (B1 de la placa
NUCLEO-F103RB), correspondiente al diagrama oficial `task_sensor.jpg` de la
cátedra (4 estados, con debounce por tick), coincidente con el modelo
`Sensor Statechart` ya validado en el TP1.

**Placa utilizada:** NUCLEO-F103RB.

## Statechart implementado (`task_sensor.c` / `task_sensor_attribute.h`)

| Estado | Evento [Guarda] | Acción | Próximo estado |
| :---: | :---: | :---: | :---: |
| ST_BTN_UP | EV_BTN_DOWN | tick = tick_max | ST_BTN_FALLING |
| ST_BTN_FALLING | EV_BTN_UP | - | ST_BTN_UP |
| ST_BTN_FALLING | EV_BTN_DOWN [tick == 0] | put_event_task_system(signal_down) | ST_BTN_DOWN |
| ST_BTN_FALLING | EV_BTN_DOWN [tick > 0] | tick-- | ST_BTN_FALLING |
| ST_BTN_DOWN | EV_BTN_UP | tick = tick_max | ST_BTN_RISING |
| ST_BTN_RISING | EV_BTN_DOWN | - | ST_BTN_DOWN |
| ST_BTN_RISING | EV_BTN_UP [tick == 0] | put_event_task_system(signal_up) | ST_BTN_UP |
| ST_BTN_RISING | EV_BTN_UP [tick > 0] | tick-- | ST_BTN_RISING |

## Pendiente
- Compilar y depurar el proyecto en STM32CubeIDE con la placa conectada.
- Verificar el funcionamiento presionando el botón B1 físico.
