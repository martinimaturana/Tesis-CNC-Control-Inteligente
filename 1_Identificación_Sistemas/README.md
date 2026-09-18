# Fase 1: Identificación de Sistemas y Modelo Digital

Esta carpeta contiene el código fuente necesario para extraer el modelo matemático de la planta física (operando en lazo abierto) y el entorno de simulación computacional donde se sintetizan y validan los algoritmos de control antes de su despliegue en hardware. 

La metodología se divide en dos etapas secuenciales alojadas en sus respectivas subcarpetas:

## 1. Adquisición de Datos (`esp32_data_logging/`)
Contiene el firmware en C++ ejecutado en el microcontrolador ESP32 para excitar dinámicamente los actuadores y recolectar telemetría de alta fidelidad. 

**Características principales:**
*   **Inyección Pseudoaleatoria:** Generación de un tren de pulsos (STEP/DIR) con cambios de dirección variables y ráfagas para capturar la inercia y fricción de cada eje.
*   **Adquisición de Sensores:** Lectura del encoder magnético absoluto AS5600 (I2C) y registro de corriente mediante el sensor INA219.
*   **Almacenamiento y HMI:** El ESP32 levanta un punto de acceso WiFi y un servidor Web que permite iniciar/detener la captura y descargar el archivo `datos.csv` generado internamente en la memoria Flash (LittleFS).

## 2. Estimación y Síntesis de Controladores (`matlab_sys_id/`)
Script central de MATLAB (`nema23_v3.m`, escalable a los demás motores) que procesa la telemetría, estima la planta matemática y exporta las lógicas de control.

**Flujo de procesamiento:**
1.  **Preprocesamiento:** Aplicación del algoritmo de desenrollado de fase (*unwrap*) sobre los datos crudos del encoder y conversión cinemática al dominio espacial (milímetros).
2.  **Identificación de Sistemas:** Estimación de modelos paramétricos (ARX, ARMAX, OE) y redes neuronales (MLP). El modelo **Output-Error (OE)** es seleccionado por su superioridad al modelar el actuador como un integrador puro afectado por ruido aditivo en el sensor.
3.  **Remuestreo (Modelo Digital):** La función de transferencia se remuestrea a 1 kHz ($T_s = 1$ ms) para asegurar compatibilidad estricta con el entorno de ejecución en tiempo real del ESP32.
4.  **Diseño Inteligente:** 
    *   Sintonización de un controlador Lógico Difuso (FLC) tipo Sugeno PD+I.
    *   Entrenamiento de una red neuronal ANFIS utilizando 500 épocas (algoritmo de aprendizaje híbrido) para clonar y suavizar la superficie difusa experta.
5.  **Exportación Embebida:** Conversión de las superficies continuas de control en **Tablas de Búsqueda Bidimensionales (LUT 21x21)** formateadas directamente como matrices estáticas `const float` de C++ para pegarlas en el firmware final.

---
**Nota de reproducibilidad:** Las perturbaciones no lineales (fricción de Coulomb, juego mecánico/backlash de 0.5 mm y ruido gaussiano) son inyectadas analíticamente en este entorno de MATLAB para evaluar el rechazo a perturbaciones y garantizar la estabilidad absoluta (Criterio de Lur'e/Círculo) previo al mecanizado real.
