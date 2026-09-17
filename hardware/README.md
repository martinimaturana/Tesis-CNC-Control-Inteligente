# Hardware y Ensamble Mecatrónico

En esta sección se detalla el diseño mecánico, la selección de actuadores, la manufactura de piezas 3D y la integración electrónica de la plataforma CNC. La máquina está diseñada bajo un enfoque de arquitectura abierta y bajo costo, transfiriendo la complejidad de la precisión mecánica hacia los algoritmos de control inteligente.

![Texto alternativo (para accesibilidad)](Imagenes/MAQUINA_CNC.jpeg)

## Arquitectura General

El diseño se basa en una estructura cartesiana de 3 ejes accionada por motores paso a paso y controlada por una arquitectura distribuida:
*   **Gestión (Soft Real-Time):** Raspberry Pi 4 Model B (Interfaz HMI y servidor).
*   **Control (Hard Real-Time):** ESP32 DevKit v1 operando el lazo cerrado a 1 kHz.

### Dinámica de los Ejes
*   **Eje Y (Pórtico / Gantry):** 2x Motores NEMA 23 + Transmisión por doble husillo T8 (8 mm/rev).
*   **Eje X (Carro Transversal):** 1x Motor NEMA 17 + Transmisión elástica por correa GT2 (40 mm/rev).
*   **Eje Z (Cabezal Vertical):** 1x Motor NEMA 11 + Transmisión por husillo trapezoidal de paso fino (2 mm/rev).

---

## Componentes Impresos en 3D (STL)

Todas las piezas personalizadas fueron manufacturadas mediante deposición fundida (FDM). Puedes encontrar los archivos `.stl` listos para imprimir en la carpeta `3d_stl_parts/` de este directorio.

** Parámetros de impresión críticos (Recomendados):**
*   **Material:** PLA+ (Requerido por su estabilidad dimensional y resistencia a la tracción de 50-60 MPa).
*   **Relleno (Infill):** 70% con patrón **Giroide** (TPMS) para optimizar la rigidez específica ante flexo-torsión.
*   **Perímetros (Paredes):** 4 perímetros.

### Listado de Piezas:
1.  `soporte_base_puente.stl`: Base trapezoidal para el acople del pórtico X sobre los carros Y.
2.  `acople_motor_x.stl`: Soporte con nervadura de refuerzo para el motor NEMA 17.
3.  `acople_polea_suelta.stl`: Soporte para la polea de reenvío GT2 del eje X.
4.  `acople_carro_eje_z.stl`: Interfaz estructural para vincular el módulo lineal del eje Z al carro del eje X.
5.  `acople_marcador.stl`: Efector final (porta-herramientas) con retención cilíndrica.
6.  `acople_correa_dentada.stl`: Mordaza o sujetador dentado para fijar la correa GT2 al carro móvil.
7.  **Carcasas y Soportes para Sensores AS5600:**
    *   `carcasa_as5600_nema23.stl` y `soporte_iman_nema23.stl`
    *   `carcasa_as5600_nema17.stl` y `soporte_iman_nema17.stl`
    *   `carcasa_as5600_nema11.stl` y `soporte_iman_nema11.stl`

---

## Esquema Electrónico y Asignación de Pines

El tablero de control utiliza una fuente de 24 VDC / 14.6A para la etapa de potencia (Drivers TB6600 ajustados a 1/8 de microstepping = 1600 pul/rev) y un regulador Step-Down XL4016 (5 VDC) para la electrónica lógica.

### Pinout del ESP32 (Microcontrolador de Tiempo Real)

| Subsistema / Actuador | Pin ESP32 | Función Específica |
| :--- | :---: | :--- |
| **Eje X** (NEMA 17) | `GPIO 27` (PUL) / `GPIO 33` (DIR) | Generación de pasos y dirección. |
| **Eje Y1 Principal** (NEMA 23) | `GPIO 2` (PUL) / `GPIO 15` (DIR) | Control de motor primario pórtico. |
| **Eje Y2 Secundario** (NEMA 23) | `GPIO 32` (PUL) / `GPIO 23` (DIR) | Control síncrono del pórtico (Gantry). |
| **Eje Z** (NEMA 11) | `GPIO 25` (PUL) / `GPIO 26` (DIR) | Generación de pasos y dirección. |
| **Sensor Eje Y1** (AS5600) | `GPIO 21` (SDA) / `GPIO 22` (SCL) | I2C Hardware 0 (Bus principal). |
| **Sensor Eje X** (AS5600) | `GPIO 18` (SDA) / `GPIO 19` (SCL) | I2C Hardware 1. |
| **Sensor Eje Z** (AS5600) | `GPIO 4` (SDA) / `GPIO 5` (SCL) | I2C vía Software (Bit-Banging). |

---

## Lista de Materiales (BOM)

El costo directo de hardware (BOM) estimado de esta plataforma es de ~$769 USD, demostrando que es posible obtener buena precisión mediante el uso de inteligencia computacional.

*   **Estructura:** Perfiles de aluminio V-Slot (40x20 y 40x40), 1 kg PLA+.
*   **Transmisión:** 2x Kits C-Beam XL 1000, 1x riel lineal T-type (100mm), correa y polea GT2.
*   **Potencia:** Fuente Mean Well 24VDC/14.6A, Conversor DC-DC XL4016, 4x Drivers TB6600.
*   **Control y Sensores:** 1x ESP32 DevKit v1, 1x Raspberry Pi 4 Model B (4GB), 3x Encoders magnéticos AS5600.

