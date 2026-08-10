# IA Digit Recognizer

Reconocedor de dígitos escritos a mano desarrollado en **C++**, con una interfaz gráfica para dibujar números del **0 al 9** y obtener predicciones de distintas redes neuronales entrenadas sobre MNIST.

El ejecutable permite dibujar directamente sobre una cuadrícula, ejecutar varias IAs y comparar sus predicciones desde una única interfaz.

![Interfaz del reconocedor de dígitos](assets/interface.png)

---

## ✨ Características

- Dibujo de dígitos directamente sobre una cuadrícula.
- Predicción de números del **0 al 9**.
- Ejecución y comparación de múltiples modelos neuronales.
- Modelos **Dense** y **CNN** con capas convolucionales, pooling y capas densas.
- Algunos modelos incluyen una clase adicional para reconocer cuando **no hay un número válido**.
- Soporte para modelos entrenados con **Data Augmentation**.
- Carga de imágenes y conjuntos/carpetas de imágenes para utilizar los mismos modelos fuera de la cuadrícula.
- Interfaz gráfica implementada con **Dear ImGui + GLFW**.
- Los modelos entrenados se almacenan en archivos `.json`.
- La aplicación está pensada para poder probar rápidamente distintas arquitecturas y observar cómo responden ante un mismo dibujo.

---

## 🖥️ Interfaz

La aplicación presenta una cuadrícula donde el usuario puede mantener presionado el clic y dibujar un número.

Después de dibujarlo se puede:

- **Predecir con IAs** → ejecuta los modelos disponibles y muestra el resultado de cada uno.
- **Limpiar Lienzo** → elimina el dibujo actual para realizar otra prueba.

Los resultados se muestran separados por modelo:

```text
Resultados de la Predicción de cada Red Neuronal:

NN Dense Model:       7
NN CNN (3x3 Middle Filter) Model:       7
NN CNN (2x2 Middle Filter) Model:       7
...
```

Esto permite comprobar visualmente cómo distintas arquitecturas interpretan exactamente el mismo dibujo.

---

## 🧠 Arquitecturas

Los modelos están organizados en la carpeta `models/` dentro de la carpeta del ejecutable.

Los archivos `.json` contienen la información necesaria de cada IA entrenada.

Los modelos están organizados **de menor a mayor potencia**, de arriba hacia abajo.

### 1. Dense IA

Primera arquitectura utilizada como referencia.

```text
784
 ↓
CPULayer
ReLU
He
Adam
128
 ↓
CPULayer
ReLU
He
Adam
64
 ↓
CPULayer
SoftMax
Xavier
Adam
10
```

### 2. CNN + Pooling + Dense

Primera arquitectura convolucional.

```text
784 (28x28)
 ↓
Conv2D
ReLU
8 filtros
3x3
padding 0
stride 1
 ↓
26x26x8
 ↓
Max Pooling
2x2
 ↓
13x13x8
 ↓
Conv2D
ReLU
16 filtros
3x3
padding 0
stride 1
 ↓
11x11x16
 ↓
Max Pooling
2x2
 ↓
6x6x16
 ↓
Dense
ReLU
He
Adam
64
 ↓
Dense
SoftMax
Xavier
Adam
10
```

Esta arquitectura combina extracción de características mediante convoluciones, reducción espacial mediante Max Pooling y clasificación mediante capas densas.

### 3. CNN + Pooling + Dense + Extra Training

Arquitectura basada en la anterior, pero con una configuración diferente para realizar pruebas adicionales.

Cambios principales:

- Middle Max Pooling de **2x2**.
- Salida SoftMax de **11 valores**.
- El valor adicional representa el caso en el que **no hay un número válido**.
- Entrenamiento de **50 epochs**.
- Incluye ejemplos donde no existe ningún número en la imagen.

Arquitectura:

```text
784 (28x28)
 ↓
Conv2D
ReLU
8 filtros
3x3
 ↓
26x26x8
 ↓
Max Pooling 2x2
 ↓
13x13x8
 ↓
Conv2D
ReLU
16 filtros
3x3
 ↓
11x11x16
 ↓
Max Pooling 2x2
 ↓
6x6x16
 ↓
Dense ReLU
64
 ↓
Dense SoftMax
11
```

El último valor de la SoftMax corresponde a **"no es un número"**.

---

## 🧪 CNN + Pooling + Dense + Data Augmentation

Esta versión utiliza la misma arquitectura anterior, incluyendo la salida de **11 clases**, pero incorpora imágenes modificadas durante el entrenamiento.

Las transformaciones utilizadas fueron:

- Rotación.
- Traslación.
- Escala.
- Ruido gaussiano.
- Brillo.

Se alternaron datos normales con datos modificados mediante Data Augmentation.

### Primera versión

```text
Arquitectura:
CNN + Pooling + Dense
11 clases

Data:
MNIST + Data Augmentation

Epochs:
100

Mini-batch:
64
```

Resultados obtenidos:

| Métrica | Resultado |
|---|---:|
| Loss de entrenamiento | 0.0199 |
| Accuracy entrenamiento | 99.50% |
| Loss de test | 0.0377 |
| Accuracy test | 98.42% |

### Segunda versión

Se volvió a entrenar utilizando:

- Mini-batch de 128.
- 100 epochs.
- `intelligentTraining_Baseline`.
- Un modelo previamente entrenado como punto de partida.

Resultados:

| Métrica | Resultado |
|---|---:|
| Loss de entrenamiento | 0.0170 |
| Accuracy entrenamiento | 99.82% |
| Loss de test | 0.0359 |
| Accuracy test | 98.73% |

---

## 🚀 CNN avanzada + Data Augmentation

La arquitectura más potente del proyecto utiliza más capas convolucionales y una representación más profunda antes de llegar a la clasificación.

```text
784 (28x28)
 ↓
Conv2D
ReLU
16 filtros
3x3
 ↓
26x26x16
 ↓
Conv2D
ReLU
16 filtros
3x3
 ↓
24x24x16
 ↓
Max Pooling
2x2
 ↓
12x12x16
 ↓
Conv2D
ReLU
16 filtros
3x3
 ↓
10x10x16
 ↓
Conv2D
ReLU
16 filtros
3x3
 ↓
8x8x16
 ↓
Max Pooling
2x2
 ↓
4x4x16
 ↓
Dense
ReLU
He
Adam
128
 ↓
Dense
SoftMax
Xavier
Adam
11
```

Esta red utiliza:

- **4 capas convolucionales**.
- **2 capas Max Pooling**.
- Una capa densa de **128 neuronas**.
- Una salida SoftMax de **11 clases**.
- La clase número 11 representa imágenes donde no hay un número.

Entrenamiento utilizado:

```text
Epochs: 200
Dataset: MNIST + datos aumentados
Salida: 11 clases
```

Resultados de la versión evaluada:

| Métrica | Resultado |
|---|---:|
| Loss de entrenamiento | 0.0052 |
| Accuracy entrenamiento | 99.92% |
| Loss de test | 0.0172 |
| Accuracy test | 99.33% |

Esta arquitectura presentó el mejor desempeño de las redes incluidas en el proyecto.

---

## 📊 Resumen de modelos

| Modelo | Arquitectura | Clases | Entrenamiento |
|---|---|---:|---|
| Dense IA | Dense 128 → 64 → 10 | 10 | MNIST |
| CNN 3x3 | Conv 8 → Pool → Conv 16 → Pool → Dense 64 → 10 | 10 | MNIST |
| CNN 2x2 Extra | CNN anterior + salida adicional | 11 | MNIST + imágenes sin número |
| CNN + Augmentation | CNN 2x2 + Data Augmentation | 11 | 100 epochs |
| CNN Augmentation avanzada | 4 Conv → 2 Pool → Dense 128 → 11 | 11 | Entrenamiento extendido + Augmentation |

---

## 🖼️ Dibujar los números

El reconocimiento funciona mejor cuando el número se dibuja de forma similar a los datos utilizados durante el entrenamiento.

Por ejemplo, se pueden incluir imágenes de referencia en el README para mostrar cómo conviene dibujar cada dígito:

```text
assets/
├── interface.png
├── digits/
│   ├── 0.png
│   ├── 1.png
│   ├── 2.png
│   ├── 3.png
│   ├── 4.png
│   ├── 5.png
│   ├── 6.png
│   ├── 7.png
│   ├── 8.png
│   └── 9.png
```

Por ejemplo:

```markdown
![Ejemplos de dígitos](assets/digits/example.png)
```

GitHub **sí permite mostrar imágenes directamente dentro de un `README.md`** utilizando rutas relativas al repositorio.

También es posible crear una sección individual para cada número:

```markdown
### 0
![Cómo dibujar el 0](assets/digits/0.png)

### 1
![Cómo dibujar el 1](assets/digits/1.png)
```

Esto es especialmente útil para explicar qué tipo de escritura reconoce mejor el modelo.

---

## 📁 Estructura del proyecto

Una estructura recomendada para la versión publicada sería:

```text
IA-Digit-Recognizer/
│
├── README.md
│
├── IAs/
│   └── IA_MNIST/
│
├── simpleimages/
│   ├── RedesConvolucionales_pizarra.png
│   ├── RedesConvolucionales.png
│   ├── ConversionImagenes_pizarra.png
│   └── ConversionImagenes.png
│
├── assets/
│   ├── interface.png
│   └── digits/
│       ├── 0.png
│       ├── 1.png
│       ├── 2.png
│       ├── 3.png
│       ├── 4.png
│       ├── 5.png
│       ├── 6.png
│       ├── 7.png
│       ├── 8.png
│       └── 9.png
│
└── Release/
    ├── IA_Digit_Recognizer.exe
    └── models/
        ├── [modelo_dense].json
        ├── [modelo_cnn_3x3].json
        ├── [modelo_cnn_2x2].json
        ├── [modelo_augmentation].json
        └── [modelo_cnn_advanced].json
```

La carpeta `models/` debe encontrarse junto al ejecutable para que la aplicación pueda cargar las distintas redes.

---

## 🧩 Tecnologías y librerías

El proyecto utiliza principalmente:

- **C++**
- **Dear ImGui** — interfaz gráfica.
- **GLFW** — ventana y gestión de entrada.
- **nlohmann/json (`json.hpp`)** — almacenamiento y carga de los modelos.
- **stb_image (`stb_image.h`)** — carga de imágenes.
- **MaggIA** — librería de redes neuronales desarrollada desde cero.

La implementación de las redes neuronales, convoluciones, pooling, entrenamiento y procesamiento necesario para los modelos pertenece a la librería desarrollada desde cero.

---

## 🖼️ Imágenes y conversiones

Además del reconocimiento de MNIST, el proyecto incluye procesamiento de imágenes.

Se pueden trabajar imágenes y conjuntos completos de imágenes utilizando una cantidad específica de canales, de forma similar al procesamiento utilizado con MNIST.

Se incluyen conversiones de imagen mediante:

- Nearest Neighbor.
- Bilinear.
- Bicubic.
- Promedio.

También se incluyen explicaciones visuales de estos sistemas en:

```text
simpleimages/
├── ConversionImagenes_pizarra
└── ConversionImagenes
```

Las redes convolucionales y el funcionamiento del pooling se resumen visualmente en:

```text
simpleimages/
├── RedesConvolucionales_pizarra
└── RedesConvolucionales
```

---

## 🎯 Objetivo del ejecutable

El objetivo principal del programa es permitir experimentar con diferentes redes neuronales para reconocimiento de dígitos de una forma visual e interactiva.

El usuario puede:

1. Dibujar un número.
2. Ejecutar las redes disponibles.
3. Ver qué predice cada modelo.
4. Comparar las respuestas entre arquitecturas.
5. Probar modelos entrenados con diferentes configuraciones.
6. Probar imágenes externas y conjuntos de imágenes.

La aplicación funciona como una demostración visual de las redes neuronales implementadas en **MaggIA**.

---

## 📌 Modelos

Los modelos `.json` incluidos en `models/` representan diferentes versiones de las redes entrenadas.

Están organizados desde la arquitectura menos potente hasta la más potente para facilitar su comparación.

Algunos modelos tienen:

- Arquitecturas diferentes.
- Una clase adicional para detectar ausencia de número.
- Entrenamiento con ejemplos sin número.
- Data Augmentation.
- Mayor profundidad y cantidad de filtros.

La aplicación detecta/carga estos modelos y muestra sus predicciones en la interfaz.

---

## 📅 Cronología del proyecto

- **05/07/2026** — Inicio del proyecto MNIST, convoluciones, pooling e imágenes.
- **13/07/2026** — Finalización de la primera etapa del proyecto MNIST.
- **04/08/2026** — Inicio de las pruebas con Data Augmentation y arquitecturas más avanzadas.
- **07/08/2026** — Finalización del proyecto principal y de las pruebas de Data Augmentation.
- **10/08/2026** — Fecha indicada como finalización total del proyecto, incluyendo organización y simplificación final de los contenidos.

---

## 🧠 MaggIA

Este proyecto forma parte del desarrollo de **MaggIA**, una librería de redes neuronales implementada desde cero en C++.

El reconocedor de dígitos funciona como una aplicación de demostración para las diferentes capacidades de la librería, especialmente:

```text
Dense Networks
      ↓
Convolutional Networks
      ↓
Pooling
      ↓
Image Processing
      ↓
Data Augmentation
      ↓
Multiple trained models
      ↓
Interactive inference
```

---

## 📜 Licencia

Añade aquí la licencia que quieras utilizar para el proyecto, por ejemplo MIT:

```text
MIT License
```

Si el proyecto depende de otras librerías, conserva también sus respectivas licencias y avisos.
