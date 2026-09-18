# Hardware y Ensamble Mecatrónico

En esta sección se detalla el diseño mecánico, la selección de actuadores, la manufactura de piezas 3D y la integración electrónica de la plataforma CNC. La máquina está diseñada bajo un enfoque de arquitectura abierta y bajo costo, transfiriendo la complejidad de la precisión mecánica hacia los algoritmos de control inteligente.

![Texto alternativo (para accesibilidad)](Imagenes/MAQUINA_CNC.jpeg)

## Arquitectura General

El diseño se basa en una estructura cartesiana de 3 ejes accionada por motores paso a paso y controlada por una arquitectura distribuida:
*   **Gestión (Soft Real-Time):** Raspberry Pi 4 Model B (Interfaz HMI y servidor).
*   **Control (Hard Real-Time):** ESP32 DevKit v1 operando el lazo cerrado a 1 kHz.

### Dinámica de los Ejes

*   **Eje Y (Pórtico / Gantry):** 2x Motores NEMA 23 + Transmisión por doble husillo T8 (8 mm/rev).
    <div align="center">
      <img src="Imagenes/riel lineal.jpeg" width="70%">
      <p><i>Detalle de la transmisión del Eje Y</i></p>
    </div>

*   **Eje X (Carro Transversal):** 1x Motor NEMA 17 + Transmisión elástica por correa GT2 (40 mm/rev).

*   **Eje Z (Cabezal Vertical):** 1x Motor NEMA 11 + Transmisión por husillo trapezoidal de paso fino (2 mm/rev).
    <div align="center">
      <img src="Imagenes/riel nema11.avif" width="70%">
      <p><i>Detalle de la transmisión del Eje Z</i></p>
    </div>

---

## Componentes Impresos en 3D (STL)

Todas las piezas personalizadas fueron manufacturadas mediante deposición fundida (FDM). Puedes encontrar los archivos `.stl` listos para imprimir en la carpeta `STLs hardware mecanico/` de este directorio.

** Parámetros de impresión críticos (Recomendados):**
*   **Material:** PLA+ (Requerido por su estabilidad dimensional y resistencia a la tracción de 50-60 MPa).
*   **Relleno (Infill):** 70% con patrón **Giroide** (TPMS) para optimizar la rigidez específica ante flexo-torsión.
*   **Perímetros (Paredes):** 4 perímetros.

### Listado de Piezas:

#### 1. Soporte Base del Puente X (`soporte base puente.stl`)
Base trapezoidal para el acople del pórtico X sobre los carros Y.
<table align="center">
  <tr>
    <td align="center"><img src="Imagenes/soporte_puente_cad.png" width="100%"></td>
    <td align="center"><img src="Imagenes/soporte_puente_real.jpeg" width="100%"></td>
  </tr>
  <tr>
    <td align="center"><b>(a) Modelo CAD</b></td>
    <td align="center"><b>(b) Pieza impresa e instalada</b></td>
  </tr>
</table>

#### 2. Acople Motor Eje X (`acople perfil 4040 nema17.stl`)
Soporte con nervadura de refuerzo para el motor NEMA 17.
<table align="center">
  <tr>
    <td align="center"><img src="Imagenes/acople_4040.png" width="100%"></td>
    <td align="center"><img src="Imagenes/acople_4040_real.jpeg" width="100%"></td>
  </tr>
  <tr>
    <td align="center"><b>(a) Modelo CAD</b></td>
    <td align="center"><b>(b) Pieza impresa e instalada</b></td>
  </tr>
</table>

#### 3. Soporte Polea de Reenvío (`acople polea suelta_v2.stl`)
Soporte para la polea libre de reenvío GT2 del eje X.
<table align="center">
  <tr>
    <td align="center"><img src="Imagenes/acople_polea_suelta.png" width="100%"></td>
    <td align="center"><img src="Imagenes/acople_polea_suelta_real.jpeg" width="100%"></td>
  </tr>
  <tr>
    <td align="center"><b>(a) Modelo CAD</b></td>
    <td align="center"><b>(b) Pieza impresa e instalada</b></td>
  </tr>
</table>

#### 4. Acople Módulo Eje Z (`acople eje z a carro eje y.stl`)
Interfaz estructural para vincular el módulo lineal del eje Z al carro del eje X.
<table align="center">
  <tr>
    <td align="center"><img src="Imagenes/acople_carro_eje_z.png" width="100%"></td>
    <td align="center"><img src="Imagenes/acople_carro_eje_z_real.jpeg" width="100%"></td>
  </tr>
  <tr>
    <td align="center"><b>(a) Modelo CAD</b></td>
    <td align="center"><b>(b) Pieza impresa e instalada</b></td>
  </tr>
</table>

#### 5. Efector Final: Porta-Marcador (`acople lapiz marcador.stl`)
Efector final (porta-herramientas) con retención cilíndrica para pruebas de validación espacial.
<table align="center">
  <tr>
    <td align="center"><img src="Imagenes/acople_marcador.png" width="100%"></td>
    <td align="center"><img src="Imagenes/acople_marcador_real.jpeg" width="100%"></td>
  </tr>
  <tr>
    <td align="center"><b>(a) Modelo CAD</b></td>
    <td align="center"><b>(b) Pieza impresa e instalada</b></td>
  </tr>
</table>

#### 6. Sujetador de Correa (`acople correa gt2 6mm - carro eje y.stl`)
Mordaza o sujetador dentado para fijar la correa GT2 al carro móvil.
<table align="center">
  <tr>
    <td align="center"><img src="Imagenes/acople_correa_al_carro.png" width="100%"></td>
    <td align="center"><img src="Imagenes/acople_correa_dentada_real.jpeg" width="100%"></td>
  </tr>
  <tr>
    <td align="center"><b>(a) Modelo CAD</b></td>
    <td align="center"><b>(b) Pieza impresa e instalada</b></td>
  </tr>
</table>

#### 7. Carcasas y Soportes para Sensores AS5600

Conjunto de carcasas protectoras para la placa del sensor y adaptadores de centrado para los imanes diametrales de neodimio. Cada pieza fue diseñada específicamente para la morfología de su respectivo actuador, garantizando la concentricidad y el entrehierro exacto.

**7.1. Sensores Eje Y (Motores NEMA 23)**
*   **Archivos:** `acople as5600 nema23.stl` y `soporte iman nema23.stl`
<table align="center">
  <tr>
    <td align="center"><img src="Imagenes/acople_as5600_nema23.png" width="100%"></td>
    <td align="center"><img src="Imagenes/acople_as5600_nema23_real.jpeg" width="100%"></td>
  </tr>
  <tr>
    <td align="center"><b>(a) Modelo CAD (NEMA 23)</b></td>
    <td align="center"><b>(b) Instalación física en Eje Y</b></td>
  </tr>
</table>

**7.2. Sensor Eje X (Motor NEMA 17)**
*   **Archivos:** `acople as5600 nema17.stl` y `soporte iman nema17.stl`
<table align="center">
  <tr>
    <td align="center"><img src="Imagenes/acople_as5600_nema17.png" width="100%"></td>
    <td align="center"><img src="Imagenes/acople_as5600_nema17_real.jpeg" width="100%"></td>
  </tr>
  <tr>
    <td align="center"><b>(a) Modelo CAD (NEMA 17)</b></td>
    <td align="center"><b>(b) Instalación física en Eje X</b></td>
  </tr>
</table>

**7.3. Sensor Eje Z (Motor NEMA 11)**
*   **Archivos:** `acople as5600 nema11.stl` y `soporte iman nema11.stl`
*   *Nota:* El soporte del imán incluye un vástago prolongado diseñado específicamente para insertarse a presión en la cavidad posterior del eje de este motor.
<table align="center">
  <tr>
    <td align="center"><img src="Imagenes/acople_as5600_nema11.png" width="100%"></td>
    <td align="center"><img src="Imagenes/acople_as5600_nema11_real.jpeg" width="100%"></td>
  </tr>
  <tr>
    <td align="center"><b>(a) Modelo CAD (NEMA 11)</b></td>
    <td align="center"><b>(b) Instalación física en Eje Z</b></td>
  </tr>
</table>

---

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

