# Tarea-1-FUNDAMENTOS-PROCESAMIENTO-DE-IM-GENES
#Pregunta 1: Saturación Selectiva de Color
Se implementa la función `colorsaturation(imagen, m_l, h_l, modo)` que recibe los siguientes parámetros:

imagen: imagen RGB normalizada en `[0, 1]`.
h_l: lista de tonos (en grados) donde se definen puntos de control.
m_l: lista de ganancias asociadas a cada `h_l`.
modo: `"HSV"` o `"cielch"`.

Para cada pixel se calcula una ganancia m(h) interpolando linealmente entre los puntos de control definidos.

Además para evitar problemas con los angulos debido a la periodicidad, se extienden los puntos de control:

h_extendido = [h_l[-1] - 360.0] + h_l + [h_l[0] + 360.0]
m_extendido = [m_l[-1]]         + m_l + [m_l[0]]

Para cargar la imagen se utiliza:
import cv2
import numpy as np

imagen = cv2.imread(r"ruta\a\imagen.tif")
imagen_normalizada = imagen.astype(np.float32) / 255.0
img_rgb = cv2.cvtColor(imagen_normalizada, cv2.COLOR_BGR2RGB)

Dentro del código van algunas de las experimentaciones, para algunas simplemente se cambiaba las variables, por lo que no están explicitas.

Para las gráficas de m(h), se incluye la función graficarm(), además se explicitan las variables y los valores usados para el experimento A,B, C y D.

#Pregunta 2: Ecualización de Histograma y Clahe

Implementación propia de ecualización de histograma local, con control de contraste y comparación con CLAHE

En el código se implementan dos funciones principales
**`imhist(X, umbral)`** La cual calcula la función de distribución acumulada (CDF) normalizada de una imagen en escala de grises, con la opción de aplicar un umbral de recorte.

 La ecualización local sin límite sobreexpone la imagen, para evitarlo, se recorta el histograma según el histograma.
 
 **`ecual_local(imagen, umbral, dimension_malla, dimension_paso)`** Esta aplica la ecualización por regiones (bloques), puede haber solapamiento, no obstante, combina los resultados mediante un promedio ponderado por la cantidad de regiones que cubren cada pixel.

 Más abajo se incluye la función **`analisisbins(imagen, bins)`** la cual cuantiza la imagen en un menor número de niveles de grises, para ver el efecto del número de bins.

 Se realiza la comparación del método de ecualización con umbral con CLAHE mediante **`cv2.createCLAHE`**

#Uso de ecualización local sin límite
imagen_final = ecual_local(imagen, umbral=None, 
                            dimension_malla=(200, 200), 
                            dimension_paso=(100, 100))
Umbral=None; No hay control de contraste
dimension_malla=(200, 200); cada bloque mide 200x200px
dimension_paso=(100, 100); los bloques se desplazan 100px entre si.

#Para comparar CLAHE

alto, ancho = imagen.shape
tile_y = alto  // 200
tile_x = ancho // 200
clahe = cv2.createCLAHE(clipLimit=3.0, tileGridSize=(tile_y, tile_x))
imagen_clahe = clahe.apply(imagen)

tileGridSize = (alto // 200, ancho // 200): indica cuántos bloques hay.

En el código van incluidos algunos de los experimentos, alguna de las variables eran modificadas para alternar entre un experimento a otro.

#Ecualización global 
imagen_global = ecual_local(imagen, None, (alto, ancho), (alto, ancho))

#Para el efecto de los bins se probó: bins=32,64,128,256 con mallas de 300x300px y 80x80px.

#Control de contraste
Sin límite (umbral=None)
Con límite (umbral=470)
CLAHE (clipLimit=3.0)

#Pregunta 3: Reescalado
Este código carga una imagen, la pasa de BGR a RGB y le aplica la función reescalado.

Para cada píxel de la imagen de salida en la posición (i, j):

$$x = \frac{i}{s}, \quad y = \frac{j}{s}$$

donde s es el factor de escala. Si s < 1$ se está reduciendo (varios píxeles de salida corresponden a uno de entrada). Si s > 1 se está ampliando (un píxel de entrada se reparte entre varios de salida).

#Vecino más cercano
Es decir, se elige el píxel más cercano a la posición solicitada.

#Interpolación bilineal

En el código se implementa en dos etapas (interpolación horizontal seguida de vertical):
python
fy1 = f11 + (f21 - f11) / (x2 - x1) * (x - x1)
fy2 = f12 + (f22 - f12) / (x2 - x1) * (x - x1)
fxy = fy1 + (fy2 - fy1) / (y2 - y1) * (y - y1)

La imagen se carga con:
imagen = cv2.imread(r"ruta\a\imagen.tif")
imagen = cv2.cvtColor(imagen, cv2.COLOR_BGR2RGB)

El reescalado se aplica:
resultado = reescalado(imagen, escala=1.4, tipo_interpolacion="bilineal")
tipo_interpolacion: "bilineal" o "vecino_mas_cercano"
escala: factor que amplia o atenúa

Para el archivo #pregunta 3 bilineal vs vecino más cercano.txt (el que compara los dos métodos de interpolación) se carga una imagen, que tambien se convierte a RGB, luego se le aplica reescalado con s=1.4 y se usan los dos métodos disponibles para comparar

img_bilineal = reescalado(imagen, escala=1.4, tipo_interpolacion="bilineal")
img_vecino   = reescalado(imagen, escala=1.4, tipo_interpolacion="vecino_mas_cercano")
