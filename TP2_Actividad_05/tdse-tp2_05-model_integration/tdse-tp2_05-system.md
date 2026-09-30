# Guía de Integración y Funcionamiento - Actividad 05

## 1. Descripción General
Esta actividad realiza la integración completa del sistema de control para la barrera de estacionamiento. Se gestionan dos actuadores independientes (LEDs) controlados desde la máquina de estados del sistema (`task_system.c`) en respuesta a los eventos de los botones (`task_sensor.c`).

### Asignación de Actuadores
* **LED A (Verde / LD2 en PA5):** Indicador de barrera abierta / subiendo.
* **LED B (Rojo / Externo en PA8):** Indicador de barrera cerrada / bajando.

---

## 2. Configuración de Hardware (`board.h`)

El LED externo (Rojo) se conecta en el pin **PA8** utilizando lógica de **Cátodo Común** (pata común conectada a GND).

```c
/* En app/inc/board.h */

/* LED A (LD2 On-board - Verde) */
#define LED_A_PIN       LD2_Pin
#define LED_A_PORT      LD2_GPIO_Port
#define LED_A_ON        GPIO_PIN_SET
#define LED_A_OFF       GPIO_PIN_RESET

/* LED B (PA8 Externo - Rojo - Cátodo Común) */
#define LED_B_PORT      GPIOA
#define LED_B_PIN       GPIO_PIN_8
#define LED_B_ON        GPIO_PIN_SET
#define LED_B_OFF       GPIO_PIN_RESET