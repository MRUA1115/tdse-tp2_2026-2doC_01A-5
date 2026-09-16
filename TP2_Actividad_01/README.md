# TP2 - Actividad 01 - 4to Proyecto - 1 Sensor Statechart

Proyecto `tdse-tp2_01-model_integration`, copiado desde
`TP2_Actividad_00` y renombrado según lo pedido por la guía.

Codifica en C el statechart de **Sensor** para 1 botón (B1 de la placa
NUCLEO-F103RB), correspondiente al modelo `Sensor Statechart` diseñado en el
TP1.

**Placa utilizada:** NUCLEO-F103RB.

## Statechart implementado (`task_sensor.c` / `task_sensor_attribute.h`)

| Estado | Evento | Acción | Próximo estado |
| :---: | :---: | :---: | :---: |
| ST_BTN_IDLE | EV_BTN_DOWN | put_event_task_system(EV_SYS_ACTIVE) | ST_BTN_ACTIVE |
| ST_BTN_ACTIVE | EV_BTN_UP | put_event_task_system(EV_SYS_IDLE) | ST_BTN_IDLE |

## Pendiente
- Compilar y depurar el proyecto en STM32CubeIDE con la placa conectada.
- Verificar el funcionamiento presionando el botón B1 físico.
