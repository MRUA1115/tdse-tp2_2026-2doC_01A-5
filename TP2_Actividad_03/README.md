# TP2 - Actividad 03 - 6to Proyecto - System Statechart

Proyecto `tdse-tp2_03-model_integration`, copiado desde `TP2_Actividad_02`
(3 sensores), con foco en el modelo **System**.

**Placa utilizada:** NUCLEO-F103RB.

## Statechart implementado (`task_system.c` / `task_system_attribute.h`)

Corresponde al modelo `System Statechart` diseñado en el TP1, en modo
`NORMAL` (1 solo sistema, `g_task_system_mode = NORMAL`).

| Estado | Evento | Acción | Próximo estado |
| :---: | :---: | :---: | :---: |
| ST_SYS_IDLE | EV_SYS_ACTIVE | put_event_task_actuator(EV_LED_ACTIVE, ID_LED_A) | ST_SYS_ACTIVE |
| ST_SYS_ACTIVE | EV_SYS_IDLE | put_event_task_actuator(EV_LED_IDLE, ID_LED_A) | ST_SYS_IDLE |

El System recibe eventos desde cualquiera de los 3 sensores (Actividad 02)
mediante la cola `event_task_system_queue` (`put_event_task_system()` /
`get_event_task_system()` / `any_event_task_system()`), y notifica al
Actuator mediante `put_event_task_actuator()`.

## Pendiente
- Confirmar en la placa física que los 3 sensores disparan correctamente
  las transiciones del System (requiere que la Actividad 02 esté validada
  con hardware conectado).
- Compilar y depurar en STM32CubeIDE.
