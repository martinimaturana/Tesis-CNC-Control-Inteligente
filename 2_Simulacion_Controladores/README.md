# Fase 2: Diseño y Simulación de Controladores (Modelo Digital)

*Viene de: [Fase 1: Identificación de Sistemas](../1_Identificacion_Sistemas/)*

A partir de las funciones de transferencia obtenidas en la fase previa, esta sección aborda el diseño y validación de los algoritmos de control de posición. La síntesis y el análisis de estabilidad se realizan en un entorno de simulación (Modelo Digital en MATLAB) para evaluar el desempeño de tres estrategias: **PID Discreto, Control Lógico Difuso (FLC) y Sistema de Inferencia Neuro-Difuso Adaptativo (ANFIS)** antes de su implementación física.

---

## Arquitectura de las Leyes de Control

### 1. Controlador Lineal (PID Discreto)
El controlador Proporcional-Integral-Derivativo (PID) se implementó en su topología paralela discreta con un tiempo de muestreo de $T_s = 1$ ms ($1$ kHz). 

Para atenuar la amplificación del ruido de cuantización inherente a los encoders magnéticos, la acción derivativa emplea una aproximación de Euler hacia atrás acoplada a un filtro paso bajo IIR de primer orden ($\alpha$). La acción integral utiliza la aproximación trapezoidal de Tustin e incorpora un método *anti-windup* condicional para mitigar la saturación de la señal de control ante perturbaciones prolongadas.

<div align="center">
  <img src="../hardware/Imagenes/diagrama_pid.png" width="80%">
  <p><i>Arquitectura en lazo cerrado del controlador PID discreto con filtrado derivativo</i></p>
</div>

### 2. Control Lógico Difuso (FLC Sugeno PD+I)
Como alternativa a las limitaciones del control lineal frente a dinámicas variantes en el tiempo, se diseñó un sistema de inferencia difusa Takagi-Sugeno de orden cero. 

*   **Arquitectura PD+I:** El núcleo difuso evalúa el error de posición ($e$) y la derivada del error ($\Delta e$) actuando como un compensador PD. En paralelo, un integrador lineal compensa el error en régimen permanente.
*   **Base de Reglas (Heurística):** Se establecieron 5 funciones de pertenencia (NB, NS, Z, PS, PB) para cada entrada, consolidando una base paramétrica de 25 reglas lógicas con salidas constantes (*singletons*).

Para ilustrar el mecanismo de toma de decisiones de la lógica de inferencia, el controlador evalúa condiciones operativas (Reglas IF-THEN) como las siguientes:
1.  **Regla de Aceleración Máxima:** `IF (Error es NB) AND (Derivada es NB) THEN (Salida es NB)`. Escenario: La planta presenta un desfase negativo considerable y la velocidad de dicho desfase aumenta; el controlador exige el máximo esfuerzo negativo del actuador.
2.  **Regla de Régimen Estacionario:** `IF (Error es Z) AND (Derivada es Z) THEN (Salida es Z)`. Escenario: La planta ha alcanzado la referencia geométrica y su velocidad relativa es nula; el controlador anula la señal de mando para evitar oscilaciones y desgaste.
3.  **Regla de Freno Inercial:** `IF (Error es PB) AND (Derivada es NB) THEN (Salida es Z)`. Escenario: La planta presenta un desfase positivo, pero la inercia del sistema ya está reduciendo dicho error a alta velocidad; el controlador interrumpe el esfuerzo motriz para prevenir un sobreimpulso (*overshoot*).

<div align="center">
  <img src="../hardware/Imagenes/diagrama_flc.png" width="80%">
  <p><i>Estructura general del Controlador Lógico Difuso Sugeno (PD+I)</i></p>
</div>

A continuación, se presenta la morfología de los conjuntos de entrada y la distribución paramétrica de los *singletons* de salida:

<table align="center">
  <tr>
    <td align="center"><img src="../hardware/Imagenes/mfs_entradas.png" width="100%"></td>
    <td align="center"><img src="../hardware/Imagenes/mfs_salida.png" width="100%"></td>
  </tr>
  <tr>
    <td align="center"><b>(a) Funciones de pertenencia (Error y Derivada)</b></td>
    <td align="center"><b>(b) Salidas constantes (Singletons)</b></td>
  </tr>
</table>

### 3. Sistema de Inferencia Neuro-Difuso Adaptativo (ANFIS)
Para optimizar la superficie de control obtenida empíricamente, se implementó una red neuronal ANFIS de 5 capas orientada a clonar la heurística del FLC mediante aprendizaje supervisado.

*   El algoritmo ajusta los antecedentes mediante retropropagación del gradiente y los consecuentes mediante estimación por mínimos cuadrados (LSE).
*   Se sustituyeron los conjuntos poligonales del FLC por funciones de campana generalizada (`gbellmf`). Al ser de clase $C^\infty$, estas funciones actúan como un filtro espacial continuo sobre la superficie de control, atenuando transiciones abruptas en la señal de mando.

<div align="center">
  <img src="../hardware/Imagenes/diagrama_anfis.png" width="70%">
  <p><i>Arquitectura topológica de la red neuro-difusa (ANFIS) de 5 capas</i></p>
</div>

---

## Hiperparámetros y Sintonización (Guía de Adaptación)

Para replicar la metodología en arquitecturas CNC con dinámicas disímiles, se deben reajustar los siguientes hiperparámetros en los scripts de síntesis:

### Control Lineal (PID)
*   $K_p$ **(Ganancia Proporcional):** Determina la rigidez del seguimiento. Su incremento reduce el tiempo de subida, pero amplifica el sobreimpulso y el requerimiento de corriente del motor.
*   $K_i$ **(Ganancia Integral):** Suprime el error en régimen permanente. Valores excesivos inducen inestabilidad temporal por el fenómeno de *windup* frente a la fricción estática del sistema mecánico.
*   $K_d$ **(Ganancia Derivativa):** Proporciona amortiguamiento dinámico, frenando el actuador anticipadamente frente a variaciones inerciales.

### Control Lógico Difuso (FLC)
Dado que el universo de discurso de los controladores difusos está normalizado internamente (ej. $[-50, 50]$ para la entrada de error y $[-150, 150]$ para la derivada), la adaptación a unidades físicas de medición se realiza mediante factores de escala:
*   $G_e$ **(Ganancia de Error):** Escala el error de posición métrico antes del proceso de fuzificación. Un incremento en $G_e$ aumenta la sensibilidad del controlador ante desviaciones espaciales submili-métricas.
*   $G_{de}$ **(Ganancia de Derivada):** Escala la tasa de cambio del error. Define el nivel de amortiguación virtual; un valor elevado suaviza la trayectoria pero incrementa el tiempo de asentamiento.
*   $K_{flc}$ **(Ganancia Integral Paralela):** Define la magnitud del acumulador integral discreto acoplado a la salida del PD difuso para asegurar convergencia estricta a estado estacionario.

### Red Neuro-Difusa (ANFIS)
La clonación y optimización de la superficie requiere la parametrización específica de la red neuronal mediante la función `anfisOptions`:
*   **Funciones de Pertenencia (MFs):** Se utilizan 5 funciones por entrada matemática (`[5 5]`) para mantener la equivalencia topológica con la matriz de 25 reglas del FLC original.
*   **Tipo de MF:** Se configuran funciones de campana generalizada (`gbellmf`) para los antecedentes, garantizando derivabilidad continua y un mapeo espacial exento de aristas lógicas.
*   **Épocas de Entrenamiento (`EpochNumber`):** El hiperparámetro se fijó en 500 iteraciones. Este volumen asegura la convergencia del Error Cuadrático Medio (RMSE) de la red sin incurrir en sobreajuste (*overfitting*).
*   **Método de Optimización:** Algoritmo Híbrido (Gradient Descent + LSE).

---

## Parámetros de Simulación y No Linealidades (MATLAB)

El entorno de simulación evalúa la robustez de los algoritmos mediante la inyección analítica de perturbaciones y no linealidades mecánicas. Para replicar los ensayos, se deben configurar los siguientes parámetros en los scripts `nema17_v4.m`, `nema23_v3_2.m` y `nema11_v3.m`:

1.  **Filtro Derivativo Global (`alpha = 0.002`):** Establece una frecuencia de corte de $\approx 0.8$ Hz. Este factor de atenuación es crítico a 1 kHz; valores superiores introducen ruido estocástico en la derivada, induciendo oscilaciones de alta frecuencia (*chattering*) que pueden saturar la etapa de potencia.
2.  **No Linealidades (`flags` activas):** 
    *   Fricción de Coulomb y fricción viscosa.
    *   Juego mecánico (Backlash): 0.5 mm en husillos y 0.2 mm en poleas/correas.
    *   Perturbación externa de 20.5 N y ruido blanco gaussiano de medición.
3.  **Restricción de Hardware (`flags.VelMaxFisica`):** Límite dinámico de velocidad impuesto por la cinemática de cada eje. Configurado a `70.0 mm/s` para X/Y y `10.0 mm/s` para el eje Z.

---

## Desempeño Dinámico Multivariable

La exposición de las estrategias a las condiciones de carga descritas generó métricas diferenciadas según la naturaleza cinemática de cada eje:

*   **Controlador PID:** Presentó los mayores tiempos de asentamiento y desviación transitoria, ocasionados por la saturación del término integral frente al roce estático y la incapacidad de la derivada altamente filtrada para predecir discontinuidades.
*   **Controlador FLC Sugeno:** Disminuyó el error geométrico, pero la topología de sus conjuntos de pertenencia amplificó la Variación Total (TV) del esfuerzo de control al interactuar con el ruido estocástico del sensor.
*   **Controlador ANFIS:** Registró el menor Error Cuadrático Medio (RMSE) y estabilizó la respuesta temporal (ej. 0.66 s de tiempo de asentamiento en el eje X). El proceso de entrenamiento optimizó la superficie, lo que se tradujo en una señal de mando con menor dispersión vibratoria en los actuadores mecánicos.

<table align="center">
  <tr>
    <td align="center"><img src="../hardware/Imagenes/Sugeno vs ANFIS vs PID -CRITICO-NEMA23_V2.png" width="100%"></td>
    <td align="center"><img src="../hardware/Imagenes/Sugeno vs ANFIS vs PID-caso3-nema17_v3.png" width="100%"></td>
  </tr>
  <tr>
    <td align="center"><b>(a) Eje Y (NEMA 23): Respuesta bajo perturbación externa</b></td>
    <td align="center"><b>(b) Eje X (NEMA 17): Estabilización de la transmisión elástica</b></td>
  </tr>
</table>

---

## Discretización y Extracción de Superficies (LUT 2D)

La evaluación iterativa de 25 reglas de inferencia difusa exige una carga computacional que excede el período de interrupción de 1 ms disponible en el microcontrolador ESP32.

Para asegurar determinismo temporal, el script de MATLAB mapea el espacio de estados continuo de las estrategias FLC y ANFIS y lo discretiza en matrices constantes bidimensionales (**Tablas de Búsqueda de 21x21 puntos**). Al finalizar la simulación, el código genera la sintaxis en C++ (ej. `const float matriz_anfis_X[21][21]`). 

La integración de estas matrices en el firmware reduce la complejidad algorítmica de $\mathcal{O}(N)$ a $\mathcal{O}(1)$ mediante una interpolación bilineal que requiere un tiempo de ejecución de procesamiento inferior a $12 \, \mu\text{s}$.

<div align="center">
  <img src="../hardware/Imagenes/superficie_3d_anfis.png" width="60%">
  <p><i>Superficie de control no lineal (LUT 21x21) optimizada por ANFIS</i></p>
</div>

---

## Siguiente Paso: Implementación y Validación en Hardware

Con las tablas de interpolación generadas y las constantes paramétricas definidas, la etapa final contempla la inyección de la lógica en el ESP32 para ejecutar interpolaciones multieje (polígonos y espirales) comprobando empíricamente el error de contorno espacial sobre la plataforma CNC.

**[Ir a la Fase 3: Ejecución en Lazo Cerrado (Hardware)](../3_Ejecucion_Lazo_Cerrado/)**
