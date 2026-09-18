# TP2 - Actividad 03 - 6to Proyecto - System Statechart

Proyecto `tdse-tp2_03-model_integration`, copiado desde `TP2_Actividad_02`
(3 sensores), con foco en el modelo **System**.

**Placa utilizada:** NUCLEO-F103RB.

## Statechart implementado (`task_system.c` / `task_system_attribute.h`)

Corresponde al diagrama oficial `task_system.jpg` de la cátedra:
**Intelligent Parking Management System**, 5 estados, en modo `NORMAL`
(1 solo sistema, `g_task_system_mode = NORMAL`).

Estado inicial `ST_SYS_WAIT_FOR_CAR_ARRIEVE`, con acción de entrada:
`put_event_task_actuator(EV_LED_OFF, ID_LED_BARRIER_OPEN);`
`put_event_task_actuator(EV_LED_ON, ID_LED_BARRIER_CLOSE)` (barrera
cerrada por defecto).

| Estado | Evento [Guarda] | Acción | Próximo estado |
| :---: | :---: | :---: | :---: |
| WAIT_FOR_CAR_ARRIEVE | EV_SYS_CAMERA | - | WAIT_FOR_BUTTON_PRESSED |
| WAIT_FOR_BUTTON_PRESSED | EV_SYS_BUTTON | tick=DEL_SYS_MAX; put_event_task_actuator(EV_LED_BLINK, ID_LED_BARRIER_OPEN); put_event_task_actuator(EV_LED_OFF, ID_LED_BARRIER_CLOSE) | WAIT_FOR_BARRIER_OPENED |
| WAIT_FOR_BARRIER_OPENED | [tick > 0] | tick-- | WAIT_FOR_BARRIER_OPENED |
| WAIT_FOR_BARRIER_OPENED | [tick == 0] | put_event_task_actuator(EV_LED_ON, ID_LED_BARRIER_OPEN) | WAIT_FOR_CAR_LEAVES |
| WAIT_FOR_CAR_LEAVES | EV_SYS_SENSOR_COIL | tick=DEL_SYS_MAX; put_event_task_actuator(EV_LED_OFF, ID_LED_BARRIER_OPEN); put_event_task_actuator(EV_LED_BLINK, ID_LED_BARRIER_CLOSE) | WAIT_FOR_BARRIER_CLOSED |
| WAIT_FOR_BARRIER_CLOSED | [tick > 0] | tick-- | WAIT_FOR_BARRIER_CLOSED |
| WAIT_FOR_BARRIER_CLOSED | [tick == 0] | put_event_task_actuator(EV_LED_ON, ID_LED_BARRIER_CLOSE) | WAIT_FOR_CAR_ARRIEVE |

Los 3 sensores de la Actividad 02 se re-mapean a los eventos reales del
parking (`BTN_A` → `EV_SYS_CAMERA`, `BTN_B` → `EV_SYS_BUTTON`, `BTN_C` →
`EV_SYS_SENSOR_COIL`) mediante la cola `event_task_system_queue`
(`put_event_task_system()` / `get_event_task_system()` /
`any_event_task_system()`), y el System notifica a los 2 Actuator
(`ID_LED_BARRIER_OPEN` / `ID_LED_BARRIER_CLOSE`) mediante
`put_event_task_actuator()`.

## Pendiente
- Confirmar en la placa física que los 3 sensores (cámara/botón/bobina)
  disparan correctamente las transiciones del System.
- Compilar y depurar en STM32CubeIDE.
