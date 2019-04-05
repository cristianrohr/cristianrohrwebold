---
title: "Extracción de características en imágenes, aplicado a la detección de peatones"
excerpt: El objetivo de esta post es implementar descriptores para la extracción de características de imágenes. De forma concreta la aplicación sera al problema de distinguir peatones del fondo.
tags:
  - datascience
  - machine learning
  - programación
  - python
  - imágenes
  - LBP
  - HoG
  - OpenCV
categories:
  - datascience
  - python
  - machine learning
  - imágenes
toc: true
comments: true
header:
  overlay_image: /assets/images/cool-background-knn.png
caption: "Imagen: [coolbackgrounds.io](https://coolbackgrounds.io/)"
last_modified_at: 2019-04-05
---


## Introducción

Vamos a ir directo a la faena. Vamos a utilizar la librería de procesamiento de imágenes [OpenCV](https://opencv.org/) la cual se encuentra disponible para diferentes lenguajes de programación, en este caso voy a usar Python.

Esta es una tarea de extracción de características en imágenes para reconocer objetos en la imagen, en este caso lo voy a aplicar a detectar peatones. Para reconocer los peatones vamos a extraer características de la imagen que permitan representarlo y describirlo matemáticamente, estas características son llamadas **descriptores** y los hay de distintos tipos, se pueden centrar en diferentes características de las imágenes como color, tamaño, textura, etc. A lo largo de este post voy a hablar del [Histograma de Gradientes Orientados](https://en.wikipedia.org/wiki/Histogram_of_oriented_gradients) (HoG).

El codigo implementado esta en [este repositorio de GitHub](https://github.com/cristianrohr/HoG_Peatones).


## Implementación

### Lectura de datos

Primero vamos a leer las imágenes, dentro de la carpeta data/train tenemos:

- "data/train/pedestrians/" 1916 imágenes de peatones
- "data/train/background/" 2390 imágenes de fondo

Para esto creamos la funcion de carga de datos que leera cada una de las imágenes de tamaño 64x128, las imágenes de fondo tendran como etiqueta el valor 0, por su parte las imágenes con peatones tendran como etiqueta el valor 1.  

Al momento de leer las imágenes la funcion de carga calcula el descriptor HoG, para esto primero definimos el objeto y luego hacemos el calculo con las funciones

```python
hog = cv2.HOGDescriptor() 
h = hog.compute(img)
```

a continuación el codigo completo

```python
# Cargo las librerías necesarias
from sklearn.metrics import confusion_matrix
import numpy as np
import cv2 as cv2
from matplotlib import pyplot as plt
from os import scandir, getcwd
from os.path import abspath
from sklearn.svm import SVC
from sklearn.metrics import confusion_matrix
import itertools
from sklearn.metrics import classification_report
from sklearn.model_selection import cross_val_score
from sklearn.model_selection import KFold
from sklearn.model_selection import cross_val_score
from sklearn.model_selection import GridSearchCV
import pickle
from tqdm import tqdm
from joblib import dump, load
import pandas as pd


def loadTrainingData():
    """
    Funcion para leer los datos de entrenamiento
    
    Retorna:
    trainingData -- matriz con las instancias
    classes --      vector con las clases de cada instancia
    """

    # Matriz de descriptores que deben ser completadas
    trainingData = np.array([])
    trainingDataNeg = np.array([])

    ################################################
    # Casos positivos
    # Obtengo la lista de casos positivos
    listFiles = [abspath(arch.path) for arch in scandir(PATH_POSITIVE_TRAIN) if arch.is_file()]
    # Itero los archivos
    for i in tqdm(range(len(listFiles))):
        file = listFiles[i]
        # Leo la imagen
        img = cv2.imread(file, cv2.IMREAD_COLOR)
        # Calculo el HOG
        hog = cv2.HOGDescriptor()
        h = hog.compute(img)
        # Lo paso a 1 dimension
        h2 = h.ravel()
        # Agrego a trainingData
        trainingData = np.hstack((trainingData, h2))
    print("Leidas " + str(len(listFiles)) + " imágenes de entrenamiento -> positivas")
    # Hago un reshape
    trainingData = trainingData.reshape((len(listFiles), len(h2)))
    # Genero el vector de clases
    classes = np.ones(len(listFiles))

    ################################################
    # Casos negativos
    # Obtengo la lista de casos positivos
    listFilesNeg = [abspath(arch.path) for arch in scandir(PATH_NEGATIVE_TRAIN) if arch.is_file()]
    # Itero los archivos
    for i in tqdm(range(len(listFilesNeg))):
        file = listFilesNeg[i]
        # Leo la imagen
        img = cv2.imread(file, cv2.IMREAD_COLOR)
        # Calculo el HOG
        hog = cv2.HOGDescriptor()
        h = hog.compute(img)
        # Lo paso a 1 dimension
        h2 = h.ravel()
        # Agrego a trainingData
        trainingDataNeg = np.hstack((trainingDataNeg, h2))
    print("Leidas " + str(len(listFilesNeg)) + " imágenes de entrenamiento -> negativas")
    # Hago un reshape
    trainingDataNeg = trainingDataNeg.reshape((len(listFilesNeg), len(h2)))
    # Genero el vector de clases
    classesNeg = np.zeros(len(listFilesNeg))

    # Merge de los datos
    # Matriz de features
    trainingData = np.concatenate((trainingData, trainingDataNeg), axis=0)
    # Vector de clases
    classes = np.concatenate((classes, classesNeg), axis=0)
    return trainingData, classes

# Seteo de paths y archivos de prueba
PATH_POSITIVE_TRAIN = "data/train/pedestrians/"
PATH_NEGATIVE_TRAIN = "data/train/background/"
PATH_POSITIVE_TEST = "data/test/pedestrians/"
PATH_NEGATIVE_TEST = "data/test/background/"
EXAMPLE_POSITIVE = PATH_POSITIVE_TEST + "AnnotationsPos_0.000000_crop_000011b_0.png"
EXAMPLE_NEGATIVE = PATH_NEGATIVE_TEST + "AnnotationsNeg_0.000000_00000002a_0.png"
```

### Creación del modelo

A continuación definimos una funcion para clasificar una imagen y se entrena un modelo de machine learning utilizando el algoritmo SVM implementado en el paquete de Python scikit-learn (https://scikit-learn.org/stable/). Se utilizan parámetros por defecto y un kernel **lineal** para el entrenamiento.

El modelo lo probaremos con dos imágenes una de fondo y una de un peaton

![negativa](/assets/images/AnnotationsNeg_0.000000_00000002a_0.png)

![positiva](/assets/images/AnnotationsPos_0.000000_crop_000011b_0.png)


```python
def test(imagen, clasificador):
    """
    Funcion para predecir el tipo de una muestra
    
    Parámetros:
    imagen --       ruta de la imagen a leer
    clasificador -- clasificador de sklearn ya entrenado
    
    Retorna:
    valor -- retorna la predicción realizada
    """
    img = cv2.imread(imagen, cv2.IMREAD_COLOR)
    hog = cv2.HOGDescriptor()
    h = hog.compute(img)
    h2 = h.reshape((1, -1))
    return(clasificador.predict(h2))

# -----------------------------
# Ejecución de la prueba
# -----------------------------

# Obtengo los datos de trainning
X_Train, y_train = loadTrainingData()

# Creo una SVM con kernel linear
clf = SVC(kernel="linear")

# Entreno la SVM
clf.fit(X_Train, y_train)

# -------------------------------------------
# Pruebo con imágenes
# -------------------------------------------

# Ejemplo negativo
res = test(EXAMPLE_NEGATIVE, clf)
print(EXAMPLE_NEGATIVE, " fue clasificado como: ", res)

# Ejemplo positivo
res = test(EXAMPLE_POSITIVE, clf)
print(EXAMPLE_POSITIVE, " fue clasificado como: ", res)
```

Esto devuelve como resultado

```python
data/test/background/AnnotationsNeg_0.000000_00000002a_0.png  fue clasificado como:  [0.]
data/test/pedestrians/AnnotationsPos_0.000000_crop_000011b_0.png  fue clasificado como:  [1.]
```

La imagen negativa (fondo) fue clasificada como 0 y la imagen con el peaton fue clasificada como 1. Una vez que chequeamos que funciona correctamente evaluaremos la performance sobre un conjunto de datos de test

- 500 imágenes de peatones
- 600 imágenes de fondo

En el repo de GitHub se encuentran las funciones para cargar los datos de test y graficar la matriz de confusion

El codigo para leer los datos de test y clasificar las imágenes es el siguiente

```python
# Obtengo los datos de test
X_Test, y_test = loadTestingData()
target_names = ['background', 'pedestrians']

# Realizo predicciones sobre el dataset de test
predicciones = clf.predict(X_Test)

# Imprimo un reporte de la clasificación
print(classification_report(y_test, predicciones, target_names=target_names))
``` 

Para evaluar los resultados primero utilice la función **classification_report** del paquete scikit-learn que calcula métricas sobre la ejecución del algoritmo comparando las etiquetas conocidas para los datos de test con las predicciones realizadas por la SVM.

A continuación graficamos la matriz de confusión

```python
# Calculo la matriz de confusion
cnf_matrix = confusion_matrix(y_test, predicciones)

# Grafico la matriz de confusion
np.set_printoptions(precision=2)
plt.figure()
plot_confusion_matrix(cnf_matrix, classes=target_names,
                      title='Matriz de confusion')
```

![hogconfusion](/assets/images/HOG_Confusion.png)

Podemos observar que 24 peatones fueron clasificados de forma errónea como fondo, y 12 imágenes de fondo fueron clasificadas de forma errónea como peatones. Esto indica que estamos cometiendo más errores para clasificar correctamente nuestra clase positiva (peatones). A pesar de esto nuestro algoritmo con un puntaje F1 de 0.97 indica que el modelo distingue muy bien los peatones del fondo.

### Validación cruzada

Hasta ahora se entreno el modelo sobre un conjunto de datos de entrenamiento y luego el modelo fue probado sobre un conjunto de datos de test. Sin embargo al realizar la evaluación de nuestro modelo de esta forma podemos tener el problema de que las particiones utilizadas para entrenar y probar el algoritmo no son representativas o tienen algun tipo de bias. Para solucionar esto podemos utilizar validación cruzada. La validación cruzada es una técnica utilizada para evaluar los resultados de un análisis estadístico y garantizar que son independientes de la partición entre datos de entrenamiento y prueba. 

Para esto mezclamos los datos de train y test en una única matriz de datos y realizamos 10 Fold CV.

```python
# Hago un merge de los datos de train y test
allData = np.concatenate((X_Train, X_Test), axis=0)
allClasses = np.concatenate((y_train, y_test))

# Ahora implemento 10 Fold Cross Validation
scores = cross_val_score(SVC(kernel="linear"), allData, allClasses, cv=10, n_jobs = 4)

# Con esto puede calcular la exactitud promedio y la varianza
print("Exactitud Promedio (Varianza): %0.4f (+/- %0.4f)" % (scores.mean(), scores.std() * 2))

# Muestro la exactitud obtenida en cada fold de 10 Fold CV
np.set_printoptions(precision=4)
print("Exactitud de cada fold= {}".format(scores))
```

Donde obtenemos la siguiente exactitud en la clasificación

```python
Exactitud Promedio (Varianza): 0.9628 (+/- 0.0164)
Exactitud de cada fold= [0.9704 0.9556 0.9482 0.9519 0.9667 0.9704 0.9593 0.9741 0.9648 0.9667]
```

### Ajuste de hiperparámetros

Hasta ahora solo se había utilizado la SVM con el kernel lineal y el resto de parámetros por defecto, sin embargo el algoritmo SVM tiene diversos hiperparámetros que pueden ser ajustados con el objetivo de mejorar u optimizar el modelo.

Para este ajuste utilize la función **grid_search** del paquete scikit-learn al cual dada una serie de parámetros prueba las diferentes combinaciones de los mismos para evaluar cual es la mejor. Además como parte del proceso le indique que para cada posible combinación de parámetros realize 5-Fold Cross Validation.

En la tabla a continuación se indican cuales fueron los parámetros que se probaron.

- C: [0.01, 1, 10]
- gamma: [0.01, 1, 10]
- kernel: ['linear', 'rbf', 'poly']

Lo más importante es que probaremos dos kernels aparte del kernel lineal, los kernels _rbf_ y _poly_ además de diferentes valores de _C_ y _gamma_. [Aquí](https://scikit-learn.org/stable/modules/svm.html) pueden leer acerca de los diferentes parámetros y kernels.


```python
def grid_search(classifier, X_train, y_train, gparams,
                score='accuracy', cv=10, jobs=-1):
    """
    Realizo una búsqueda de hiperparametros utilizando una grilla
    con la función GridSearchCV de sklearn.model_selection

    Parámetros:
    classifier -- sklearn ML algorithm
    X_train --    Training features
    y_train --    Training labels
    gparams --    Dictionary of parameters to be screened
    score --      Scoring metric
    cv --         K value for Cross Validation
    jobs --       Cores (-1 for all)

    Retorna:
        Mejor modelo
    """
    
    # Inicializo la clase de GridSearch
    gd_sr = GridSearchCV(estimator=classifier,
                         param_grid=gparams,
                         scoring=score,
                         cv=cv,
                         n_jobs=jobs)

    # Hago un fit de los datos de train
    gd_sr.fit(X_train, y_train)

    # Resultados
    best_result = gd_sr.best_score_
    print("Mejor Resultado = {}".format(best_result))

    # Get the best parameters
    best_parameters = gd_sr.best_params_
    print("Mejores parámetros = {}".format(best_parameters))

    return gd_sr


# Defino los parámetros de búsqueda
params = {'C': [0.01, 1, 10],
          'gamma': [0.01, 1, 10],
          'kernel': ['linear', 'rbf', 'poly']
         }

results = grid_search(SVC(), allData, allClasses, params, score='accuracy', cv=5, jobs=6)
```

Los mejores parámetros para el modelo son los siguientes:

```python
Mejor Resultado = 0.9757676655567887
Mejores parámetros = {'C': 10, 'gamma': 0.01, 'kernel': 'rbf'}
```

## Detección de peatones a diferentes escalas

Hasta ahora se estuvo trabajando con imágenes de tamaño 128x64. Los datos de entrenamiento y testing solo tenían una imagen de peaton o fondo. En este caso se extiende el framework para detectar personas en imágenes de cualquier tamaño y a diferentes escalas. La idea es detectar las personas y marcarlas con un rectángulo.

Se volvio a entrenar el clasificar y para esto utilize todas las imágenes de ambas carpetas (train y test) para entrenar el modelo. El kernel utilizado fue rbf con parámetros C:10 y gamma:0.01, elegí estos parámetros porque fueron los que mejores resultados dieron en el proceso de optimización de hiperparámetros. 

**Algo muy importante es que agregue la opción probability=True**, esto es porque con esta opción puedo realizar predicciones en donde obtengo la probabilidad para cada clase, y esto luego lo usare para filtrar las detecciones de peatones para disminuir el número de falsos positivos.

```python
# Creo una SVM con kernel rbf
clf = SVC(kernel="rbf", C = 10, gamma = 0.01, probability = True)

# Entreno la SVM
clf.fit(allData, allClasses)

# Guardo el modelo entreando
dump(clf, 'HOGbest_clf.joblib')
```

Dada una imagen de tamaño mxn lo que hago es tomar una porción de dicha imagen de tamaño 128x64, y pruebo en dicha porción si hay una persona, si se encuentra una persona y la probabilidad de esa clase es mayor a 0.9 guardo las posiciones en una lista. Para moverme por la imagen original utilizo desplazamientos de 32 pixeles tanto en el eje X como el eje Y.

Para solucionar los problemas de escala de la detección de las personas, creo [piramides de la imágen](http://acodigo.blogspot.com/2017/02/piramides-de-imagenes-con-opencv.html) (pirámide multiescala) muestreando la imagen original a tamaños menores. De forma recursiva cada imagen se reduce a un 83% de la anterior. 

![deteccionpeatonesexp](/assets/images/436062.fig.007.jpg)

Esto se realiza con la siguiente función.

```python

def get_pyramid(img):

    pyramid = [img] # Asigno la imagen sin escala como primer elemento
    new_level = img # Imagen sobre la que ire realizando las disminuciones de tamaño

    while np.shape(new_level)[0] >= 128 and np.shape(new_level)[1] >= 64:
        new_level = cv2.GaussianBlur(src=new_level, ksize=(7, 7), sigmaX=1)
        # 0.8333333 is 1 / 1.2
        new_level = cv2.resize(new_level, dsize=(0, 0), fx=0.8333333, fy=0.8333333)
        pyramid.append(new_level)
    return pyramid


```

La siguiente función es la que implementa la ventana deslizante para la detección de los peatones

```python
def piramide_HOG(piramide_img, pred_thr = 0.9):
    """
    Función para la detección de peatones utilizando el descriptor HoG y una SVM
    
    Parámetros:
     - piramide_img: piramide imágenes
     - pred_thr: thresold de la probabilidad para guardar una deteccion como peaton
     
    Retorna:
     - peatones: lista de listas con las detecciones
    """
    
    # Creo el descriptor para la imagen
    hog = cv2.HOGDescriptor()

    # Imagen sin escalar
    img_sin_escala = piramide_img[0]

    # Variables auxiliares
    cantidad = 0
    peatones = [] # Aquí guardo todas las detecciones

    # Itero las imagenes de la piramide
    for index in range(len(piramide_img)):

        # Obtengo la escala y la imagen
        imgp = piramide_img[index]
        # Calculo la escala de la imagen, a partir del tamaño de la original
        escala = imgp.shape[0] / img_sin_escala.shape[0]

        # Voy a ir desplazandome por la imagen
        alto_maximo = imgp.shape[0]
        ancho_maximo = imgp.shape[1]

        # Itero el alto
        for wy in np.arange(0, alto_maximo - 128, 32):
            # Itero el ancho
            for wx in np.arange(0, ancho_maximo - 64, 32):

                # Obtengo la porcion de imagen 128x64
                cropped = imgp[wy:wy+128, wx:wx+64]
                
                # Calculo el hog
                h = hog.compute(cropped)
                # Lo paso a 1 dimension
                h2 = h.reshape((1, -1))
                
                # Pruebo si hay un peaton
                pred = hog_clf.predict(h2)
                # Calculo las probabilidades de las dos clases
                predprob = hog_clf.predict_proba(h2)
                
                # Si la prediccion de peaton es positiva y supera el threshold
                if pred == 1 and predprob[0][1] > pred_thr:
                    # Guardo la posicion de la deteccion para graficar
                    detectado = [] # Genero lista vacia
                    if escala < 1:
                        detectado.append(wx/escala)
                        detectado.append(wy/escala)
                        detectado.append((wx/escala)+(64/escala))
                        detectado.append((wy/escala)+(128/escala))
                    else:
                        detectado.append(wx)
                        detectado.append(wy)
                        detectado.append(wx+64)
                        detectado.append(wy+128)
                    peatones.append(detectado) # Lo agrego a la lista general
                    cantidad += 1              # Cuento un peaton más
    return peatones
```


A continuación probamos sobre una imagen marcando las detecciones

```python
# Busco los peatones y los marcpo

peatones_encontrados = piramide_HOG(piramide)
copia_hog = imagen.copy()
for peatonf in peatones_encontrados:
    v1 = int(peatonf[0])
    v2 = int(peatonf[1])
    v3 = int(peatonf[2])
    v4 = int(peatonf[3])
    cv2.rectangle(copia_hog, (v1, v2), (v3, v4), (255,0,0), 2)
    
# Grafico las detecciones del HoG comun
img_peatonesHoG = copia_hog[:,:,::-1]
fig = plt.figure(figsize = (15, 15))
ax = fig.add_subplot(111)
ax.imshow(img_peatonesHoG, interpolation='none')
ax.set_title('Peatones detectados HoG')
plt.show()
plt.savefig("HoG_peaton.png")
```

![deteccionpeatones](/assets/images/HOG_peaton.png)

El problema que nos encontramos al desarrollar esta metodología es la detección múltiple de la misma persona. Para solucionar esto utilize la técnica Non-Maximum Suppression (NMS), más concretamente la implementación en el paquete _imutils_ de python.

```python
copia_hog_NMS = imagen.copy()

# Obtengo los cuadrados para calcular el NMS
rects = np.array([[x, y, w, h] for (x, y, w, h) in peatones_encontrados])
pick = non_max_suppression(rects, probs=None, overlapThresh=0.65)
print(pick)

# Grafico los cuadrados finales
for (xA, yA, xB, yB) in pick:
    cv2.rectangle(copia_hog_NMS, (xA, yA), (xB, yB), (0, 255, 0), 2)

# Grafico las detecciones
img_peatonesHoGNMS = copia_hog_NMS[:,:,::-1]
fig = plt.figure(figsize = (15, 15))
ax = fig.add_subplot(111)
ax.imshow(img_peatonesHoGNMS, interpolation='none')
ax.set_title('Peatones detectados HoG filtrado NMS')
plt.show()
plt.savefig("HOG_NMS_peaton.png")

```
![deteccionpeatonesNMS](/assets/images/HOG_NMS_peaton.png)
