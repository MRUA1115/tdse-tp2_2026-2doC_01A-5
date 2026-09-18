# FIUBA - Electrónica - Taller de Sistemas Embebidos
## Trabajo Práctico N°: 2 - Diagramas de Estado - Codificación en C
### Año-Cuatrimestre - Curso-Grupo
### Responsable de la entrega:
| Padrón | Apellidos, Nombres | Fecha | Deadline |
| :----- | :--------------------- | :------: | :-------: |
| 112119 | 2026, ZZZ | | |

## Estructura del repositorio

| Carpeta | Actividad | Descripción |
| :--- | :--- | :--- |
| `TP2_Actividad_00` | Problem approach | Proyecto base `model_integration` (1 Sensor + 1 System + 1 Actuator), tal como se descarga de la cátedra |
| `TP2_Actividad_01` | 4to Proyecto - 1 Sensor Statechart | Sensor real (4 estados con debounce por tick), según `task_sensor.jpg` |
| `TP2_Actividad_02` | 5to Proyecto - 3 Sensor Statechart | Extensión del Sensor real a 3 instancias (botones/dip switch) |
| `TP2_Actividad_03` | 6to Proyecto - System Statechart | System real (5 estados, Intelligent Parking Management), según `task_system.jpg` |
| `TP2_Actividad_04` | 7mo Proyecto - 1 Actuator Statechart | Actuator real (3 estados OFF/ON/BLINK), según `task_actuator.jpg` |
| `TP2_Actividad_05` | 8vo Proyecto - 2 Actuator Statechart | Extensión del Actuator real a 2 instancias (LEDs externos) |

**Placa utilizada:** NUCLEO-F103RB.

> Nota: las Actividades 02 y 05 (extensión a 3 sensores y 2 actuadores) requieren
> hardware adicional (dip switch/teclado membrana y módulo LED RGB/mini semáforo)
> para su verificación física final. El código fue preparado de forma anticipada,
> pendiente de validar en la placa real una vez disponible el hardware.
