---
title: "Introducción a KNN con R"
excerpt: El dataset Wisconsin Diagnostic Breast Cancer (WDBC) contiene 30 variables computadas a traves de la digitalización de imágenes conseguidas a través de punción aspirativa con aguja fina para el diagnóstico de tumores. El dataset contiene mas de 500 muestras, el reto es construir un clasificador capaz de reconocer si un lunar es maligno o benigno.
tags:
  - datascience
  - machine learning
  - programación
  - R
categories:
  - datascience
  - R
  - machine learning
toc: true
comments: true
header:
  overlay_image: /assets/images/cool-background-knn.png
caption: "Imagen: [coolbackgrounds.io](https://coolbackgrounds.io/)"
last_modified_at: 2018-11-30
---


# Introducción
En este tutorial vamos a utilizar machine learning para intentar predecir si un tipo de tumor es _benigno_ o _maligno_. Esto lo haremos utilizando el algoritmo [k-nearest neighbors](https://en.wikipedia.org/wiki/K-nearest_neighbors_algorithm) más conocido como KNN o K vecinos más cercanos en español. Este algoritmo se puede usar tanto en problemas de clasificación, como el que nos ocupa en este momento, o así tambien de regresión.

La idea en la que se basa es realmente muy simple, y consiste en mirar los K vecinos más cercanos y elegir la etiqueta de una nueva observación en base a las etiquetas de los vecinos. 

Generalmente se dice que es un algoritmo perezoso, ya que no construye un modelo, sino que almacena la información de todas las observaciones en memoria.

## Obtención del dataset

- Carga de librerías necesarias


```R
library(ggplot2)
library(caret)
library(class)
library(RCurl)
library(ggthemr)
ggthemr('fresh')

```

- Carga del dataset. Leemos el dataset directamente desde la web, y lo almacenamos en un [data.frame](https://www.rdocumentation.org/packages/base/versions/3.5.1/topics/data.frame) de R.


```R
UCI_data_URL <- getURL('https://archive.ics.uci.edu/ml/machine-learning-databases/breast-cancer-wisconsin/wdbc.data')
names <- c('id_number', 'diagnosis', 'radius_mean', 
         'texture_mean', 'perimeter_mean', 'area_mean', 
         'smoothness_mean', 'compactness_mean', 
         'concavity_mean','concave_points_mean', 
         'symmetry_mean', 'fractal_dimension_mean',
         'radius_se', 'texture_se', 'perimeter_se', 
         'area_se', 'smoothness_se', 'compactness_se', 
         'concavity_se', 'concave_points_se', 
         'symmetry_se', 'fractal_dimension_se', 
         'radius_worst', 'texture_worst', 
         'perimeter_worst', 'area_worst', 
         'smoothness_worst', 'compactness_worst', 
         'concavity_worst', 'concave_points_worst', 
         'symmetry_worst', 'fractal_dimension_worst')
breast_cancer <- read.table(textConnection(UCI_data_URL), sep = ',', col.names = names)
```

## Exploración del dataset
A continuación algunos comandos básicos que forman parte del **Análisis Exploratorio de Datos**, probablemente dedique un post a hablar de esto, pero mas adelante.


```R
str(breast_cancer)  # Información del tipo de dato al que pertenece cada variable
dim(breast_cancer)  # Cantidad de filas y columnas (muestras y variables)
```

    'data.frame':	569 obs. of  32 variables:
     $ id_number              : int  842302 842517 84300903 84348301 84358402 843786 844359 84458202 844981 84501001 ...
     $ diagnosis              : Factor w/ 2 levels "B","M": 2 2 2 2 2 2 2 2 2 2 ...
     $ radius_mean            : num  18,0 20,6 19,7 11,4 20,3 ...
     $ texture_mean           : num  10,4 17,8 21,2 20,4 14,3 ...
     $ perimeter_mean         : num  122,8 132,9 130,0 77,6 135,1 ...
     $ area_mean              : num  1001 1326 1203 386 1297 ...
     $ smoothness_mean        : num  0,1184 0,0847 0,1096 0,1425 0,1003 ...
     $ compactness_mean       : num  0,2776 0,0786 0,1599 0,2839 0,1328 ...
     $ concavity_mean         : num  0,3001 0,0869 0,1974 0,2414 0,1980 ...
     $ concave_points_mean    : num  0,1471 0,0702 0,1279 0,1052 0,1043 ...
     $ symmetry_mean          : num  0,242 0,181 0,207 0,260 0,181 ...
     $ fractal_dimension_mean : num  0,0787 0,0567 0,0600 0,0974 0,0588 ...
     $ radius_se              : num  1,095 0,543 0,746 0,496 0,757 ...
     $ texture_se             : num  0,905 0,734 0,787 1,156 0,781 ...
     $ perimeter_se           : num  8,59 3,40 4,58 3,44 5,44 ...
     $ area_se                : num  153,4 74,1 94,0 27,2 94,4 ...
     $ smoothness_se          : num  0,00640 0,00522 0,00615 0,00911 0,01149 ...
     $ compactness_se         : num  0,0490 0,0131 0,0401 0,0746 0,0246 ...
     $ concavity_se           : num  0,0537 0,0186 0,0383 0,0566 0,0569 ...
     $ concave_points_se      : num  0,0159 0,0134 0,0206 0,0187 0,0188 ...
     $ symmetry_se            : num  0,0300 0,0139 0,0225 0,0596 0,0176 ...
     $ fractal_dimension_se   : num  0,00619 0,00353 0,00457 0,00921 0,00511 ...
     $ radius_worst           : num  25,4 25,0 23,6 14,9 22,5 ...
     $ texture_worst          : num  17,3 23,4 25,5 26,5 16,7 ...
     $ perimeter_worst        : num  184,6 158,8 152,5 98,9 152,2 ...
     $ area_worst             : num  2019 1956 1709 568 1575 ...
     $ smoothness_worst       : num  0,162 0,124 0,144 0,210 0,137 ...
     $ compactness_worst      : num  0,666 0,187 0,424 0,866 0,205 ...
     $ concavity_worst        : num  0,712 0,242 0,450 0,687 0,400 ...
     $ concave_points_worst   : num  0,265 0,186 0,243 0,258 0,163 ...
     $ symmetry_worst         : num  0,460 0,275 0,361 0,664 0,236 ...
     $ fractal_dimension_worst: num  0,1189 0,0890 0,0876 0,1730 0,0768 ...



    569
    32


El comando `summary` es una forma muy sencilla de obtener estadísticas descriptivas de cada variable de nuestro dataset.


```R
summary(breast_cancer)
```


       id_number         diagnosis  radius_mean      texture_mean  
     Min.   :     8670   B:357     Min.   : 6,981   Min.   : 9,71  
     1st Qu.:   869218   M:212     1st Qu.:11,700   1st Qu.:16,17  
     Median :   906024             Median :13,370   Median :18,84  
     Mean   : 30371831             Mean   :14,127   Mean   :19,29  
     3rd Qu.:  8813129             3rd Qu.:15,780   3rd Qu.:21,80  
     Max.   :911320502             Max.   :28,110   Max.   :39,28  
     perimeter_mean     area_mean      smoothness_mean   compactness_mean 
     Min.   : 43,79   Min.   : 143,5   Min.   :0,05263   Min.   :0,01938  
     1st Qu.: 75,17   1st Qu.: 420,3   1st Qu.:0,08637   1st Qu.:0,06492  
     Median : 86,24   Median : 551,1   Median :0,09587   Median :0,09263  
     Mean   : 91,97   Mean   : 654,9   Mean   :0,09636   Mean   :0,10434  
     3rd Qu.:104,10   3rd Qu.: 782,7   3rd Qu.:0,10530   3rd Qu.:0,13040  
     Max.   :188,50   Max.   :2501,0   Max.   :0,16340   Max.   :0,34540  
     concavity_mean    concave_points_mean symmetry_mean    fractal_dimension_mean
     Min.   :0,00000   Min.   :0,00000     Min.   :0,1060   Min.   :0,04996       
     1st Qu.:0,02956   1st Qu.:0,02031     1st Qu.:0,1619   1st Qu.:0,05770       
     Median :0,06154   Median :0,03350     Median :0,1792   Median :0,06154       
     Mean   :0,08880   Mean   :0,04892     Mean   :0,1812   Mean   :0,06280       
     3rd Qu.:0,13070   3rd Qu.:0,07400     3rd Qu.:0,1957   3rd Qu.:0,06612       
     Max.   :0,42680   Max.   :0,20120     Max.   :0,3040   Max.   :0,09744       
       radius_se        texture_se      perimeter_se       area_se       
     Min.   :0,1115   Min.   :0,3602   Min.   : 0,757   Min.   :  6,802  
     1st Qu.:0,2324   1st Qu.:0,8339   1st Qu.: 1,606   1st Qu.: 17,850  
     Median :0,3242   Median :1,1080   Median : 2,287   Median : 24,530  
     Mean   :0,4052   Mean   :1,2169   Mean   : 2,866   Mean   : 40,337  
     3rd Qu.:0,4789   3rd Qu.:1,4740   3rd Qu.: 3,357   3rd Qu.: 45,190  
     Max.   :2,8730   Max.   :4,8850   Max.   :21,980   Max.   :542,200  
     smoothness_se      compactness_se      concavity_se     concave_points_se 
     Min.   :0,001713   Min.   :0,002252   Min.   :0,00000   Min.   :0,000000  
     1st Qu.:0,005169   1st Qu.:0,013080   1st Qu.:0,01509   1st Qu.:0,007638  
     Median :0,006380   Median :0,020450   Median :0,02589   Median :0,010930  
     Mean   :0,007041   Mean   :0,025478   Mean   :0,03189   Mean   :0,011796  
     3rd Qu.:0,008146   3rd Qu.:0,032450   3rd Qu.:0,04205   3rd Qu.:0,014710  
     Max.   :0,031130   Max.   :0,135400   Max.   :0,39600   Max.   :0,052790  
      symmetry_se       fractal_dimension_se  radius_worst   texture_worst  
     Min.   :0,007882   Min.   :0,0008948    Min.   : 7,93   Min.   :12,02  
     1st Qu.:0,015160   1st Qu.:0,0022480    1st Qu.:13,01   1st Qu.:21,08  
     Median :0,018730   Median :0,0031870    Median :14,97   Median :25,41  
     Mean   :0,020542   Mean   :0,0037949    Mean   :16,27   Mean   :25,68  
     3rd Qu.:0,023480   3rd Qu.:0,0045580    3rd Qu.:18,79   3rd Qu.:29,72  
     Max.   :0,078950   Max.   :0,0298400    Max.   :36,04   Max.   :49,54  
     perimeter_worst    area_worst     smoothness_worst  compactness_worst
     Min.   : 50,41   Min.   : 185,2   Min.   :0,07117   Min.   :0,02729  
     1st Qu.: 84,11   1st Qu.: 515,3   1st Qu.:0,11660   1st Qu.:0,14720  
     Median : 97,66   Median : 686,5   Median :0,13130   Median :0,21190  
     Mean   :107,26   Mean   : 880,6   Mean   :0,13237   Mean   :0,25427  
     3rd Qu.:125,40   3rd Qu.:1084,0   3rd Qu.:0,14600   3rd Qu.:0,33910  
     Max.   :251,20   Max.   :4254,0   Max.   :0,22260   Max.   :1,05800  
     concavity_worst  concave_points_worst symmetry_worst   fractal_dimension_worst
     Min.   :0,0000   Min.   :0,00000      Min.   :0,1565   Min.   :0,05504        
     1st Qu.:0,1145   1st Qu.:0,06493      1st Qu.:0,2504   1st Qu.:0,07146        
     Median :0,2267   Median :0,09993      Median :0,2822   Median :0,08004        
     Mean   :0,2722   Mean   :0,11461      Mean   :0,2901   Mean   :0,08395        
     3rd Qu.:0,3829   3rd Qu.:0,16140      3rd Qu.:0,3179   3rd Qu.:0,09208        
     Max.   :1,2520   Max.   :0,29100      Max.   :0,6638   Max.   :0,20750        


## Preprocesamiento de datos
Generalmente en el paso anterior de análisis exploratorio de datos o EDA por sus siglas en inglés, generaríamos visualizaciones de los datos, como ser histogramas y qq-plots para chequear la distribución, boxplot's para detectar la presencia de outliers, calcula estadísticas de la muestra, analizar correlación entre variables, repito, pienso dedicar otro post a este tipo de tareas previas al modelado. 

Luego del EDA, debemos preprocesar, en caso de ser necesario, los datos para entrar a la etapa de modelado. En este caso vamos a realizar 3 tareas:

- Remover la columna con el ID de la muestra (id_number)
- Renombrar los dos posibles valores de la columna diagnosis, de _M_ y _B_ a _Maligno_ y _Benigno_
- Normalizar los datos

La normalización de datos es un preprocesamiento muy importante y necesario para ciertos tipos de algoritmos de machine learning. En el caso particular de los algoritmos que se basan en métricas de distancia, supongamos que nuestras features son por ejemplo altura de la persona, largo del brazo y largo del dedo. Si los datos no están normalizados, la diferencia de alturas de diferentes personas sera mucho mayor, que por ejemplo, la diferencia en la longitud del dedo, por esto es necesario realizar una normalización de los datos, y de esta forma evitar que el algoritmo le de más importancia a ciertas características.



```R
# Remover la columna con el ID de la muestra (id_number)
breast_cancer <- breast_cancer[,!(colnames(breast_cancer) %in% "id_number")]

# Renombrar los dos posibles valores de la columna diagnosis, de M y B a Maligno y Benigno
breast_cancer$diagnosis <- factor(breast_cancer$diagnosis, levels = c("M", "B"), labels = c("Malignant", "Benign"))
table(breast_cancer$diagnosis)

# Normalizar los datos
breast_cancer_n <- as.data.frame(lapply(breast_cancer[,2:31], scale, center = TRUE, scale = TRUE))
breast_cancer_n$diagnosis <-  breast_cancer$diagnosis
breast_cancer_n_labels <- breast_cancer$diagnosis
```


    
    Malignant    Benign 
          212       357 


Chequeamos que la normalización funcionó.

- Antes:


```R
summary(breast_cancer[,c("radius_mean", "area_mean", "smoothness_mean")])
```


      radius_mean       area_mean      smoothness_mean  
     Min.   : 6,981   Min.   : 143,5   Min.   :0,05263  
     1st Qu.:11,700   1st Qu.: 420,3   1st Qu.:0,08637  
     Median :13,370   Median : 551,1   Median :0,09587  
     Mean   :14,127   Mean   : 654,9   Mean   :0,09636  
     3rd Qu.:15,780   3rd Qu.: 782,7   3rd Qu.:0,10530  
     Max.   :28,110   Max.   :2501,0   Max.   :0,16340  


- Después


```R
summary(breast_cancer_n[,c("radius_mean", "area_mean", "smoothness_mean")])
```


      radius_mean        area_mean       smoothness_mean   
     Min.   :-2,0279   Min.   :-1,4532   Min.   :-3,10935  
     1st Qu.:-0,6888   1st Qu.:-0,6666   1st Qu.:-0,71034  
     Median :-0,2149   Median :-0,2949   Median :-0,03486  
     Mean   : 0,0000   Mean   : 0,0000   Mean   : 0,00000  
     3rd Qu.: 0,4690   3rd Qu.: 0,3632   3rd Qu.: 0,63564  
     Max.   : 3,9678   Max.   : 5,2459   Max.   : 4,76672  


## Modelado
Como dije en la introducción, este algoritmo es un lazzy learner y no genera un modelo, sino que utiliza datos para calcular la distancia de los K vecinos más cercanos con respecto a una muestra incógnita y le asigna la etiqueta de la clase mayoritaria.

Hay parámetros que se pueden modificar como ser la métrica de distancia que se emplea, o los K vecinos que se desean tener en cuenta.

En este caso en particular vamós a chequear como se comporta el algoritmo cuando tenemos en cuenta valores de K desde 1 a 15. Generalmente los valores de K son impares, para evitar posibles empates. 

Utilizaremos la función knn del paquete [class](https://cran.r-project.org/web/packages/class).

Comenzamos separando nuestro dataset en datos de entrenamiento y test. Un 80% de datos de entrenamiento y 20% de testeo


```R
set.seed(20) # Semilla para replicar resultados
# trainIndex contendra la información de las muestras de entrenamiento
trainIndex <- createDataPartition(breast_cancer_n[, "diagnosis"], p = 0.8, 
                                  list = FALSE, 
                                  times = 1)

x_train = breast_cancer_n[trainIndex, ] # Datos de entrenamiento
x_test = breast_cancer_n[-trainIndex, ] # Datos de testeo
y_train = x_train$diagnosis             # Etiqueta datos de entrenamiento
y_test = x_test$diagnosis               # Etiqueta datos de testeo
# Remuevo la columna diagnosis de x_train y x_test
x_train <- x_train[, !(colnames(x_train) %in% "diagnosis")]
x_test <- x_test[, !(colnames(x_test) %in% "diagnosis")]

nrow(x_train)
nrow(x_test)
```


456



113


Creo una función llamada **runKNN** que recibe como parámetros el valor de K que deseo, y los datos de entrenamiento, testeo y la etiqueta de los datos


```R
runKNN <- function(Kval, trainData, testData, trainLabels, testLabels) {
  # Utilizo la funcion knn del paquete class para predecir con knn
  # Retorna la accuracy
  pred <- knn(k = Kval, train = trainData, test = testData, cl = trainLabels)
  cm <- as.matrix(table(Predicted = pred, Actual = testLabels))
  accuracy <- sum(diag(cm))/sum(cm)
  accuracy
}
```

Luego defino los valores de K que voy a probar, dijimos de 1 a 15


```R
k_values <- seq(1, 15, by = 2)
```


Utilizaremos la función lapply para ejecutar la funcion _runKNN_ con los diferentes valores de K


```R
resultados <- lapply(k_values, runKNN, x_train, x_test, y_train, y_test)
resultados
```


<ol>
	<li>0,929203539823009</li>
	<li>0,955752212389381</li>
	<li>0,955752212389381</li>
	<li>0,946902654867257</li>
	<li>0,955752212389381</li>
	<li>0,955752212389381</li>
	<li>0,938053097345133</li>
	<li>0,929203539823009</li>
</ol>



Obtuvimos los valores de accuracy a través de diferentes valores de K. Podemos graficar los resultados para cada valor de K utilizando ggplot2.


```R
df <- data.frame(K = k_values, Accuracy = unlist(resultados))
ggplot(data = df, aes(x = K, y = Accuracy)) + geom_line()+ theme_light() + ggtitle("KNN, Accuracy K=1:15")
```



![KNN_plot](/assets/images/KNN_plot.png)


### Resumen
Hasta el momento fuimos capaces de predecir si el tipo de tumor es maligno o benigno con más de un 95% de certeza.

![KNNsalt](/assets/images/KNN.jpg)

Nada mal eh???

Bueno, la realidad es que este dataset es simple de resolver por eso es que aún un método como KNN entrega buenos resultados.

Además nuestro modelado presenta ciertas deficiencias. La más notoria es que no usamos una técnica como validación cruzada (cross-validation). 

La validación cruzada es una técnica utilizada para evaluar los resultados de un análisis estadístico y garantizar que son independientes de la partición entre datos de entrenamiento y prueba, es decir que nuestros resultados no se vean favorecidos o perjudicados por el dataset de entrenamiento y testeo que utilizamos. Ok Ok más despacio cerebrito. Suena muy bien y muy útil, pero como sería?, simple, miremos la siguiente imagen que es un ejemplo de 5-Fold Cross Validation.

![5fcv](/assets/images/validacion_cruzada.png)

Como se puede observar, la idea es particionar el dataset en un número de folds, en el ejemplo 5, y en cada iteración se utilizan 4 folds para entrenar y el restante para testear, finalmente se promedian los errores y se calcula la desviación estandar.

## Cross Validation

Vamos a realizar 5-Fold cross validation sobre el dataset y vamos a chequear los nuevos resultados.

Primero creamos las 5 particiones del dataset


```R
# Creo 5 particiones
set.seed(87)
data_folds <- createFolds(breast_cancer_n[, "diagnosis"], k = 5, 
                              list = TRUE, returnTrain = FALSE)
```


```R
str(data_folds)
```

    List of 5
     $ Fold1: int [1:114] 10 15 20 21 22 25 29 31 40 57 ...
     $ Fold2: int [1:115] 6 11 16 17 28 30 32 35 37 51 ...
     $ Fold3: int [1:113] 1 5 8 13 14 19 23 36 49 53 ...
     $ Fold4: int [1:113] 2 4 9 24 41 42 43 44 45 46 ...
     $ Fold5: int [1:114] 3 7 12 18 26 27 33 34 38 39 ...


Ahora vamos a realizar la cross-validation. El codigo a continuación esta dentro de un ciclo FOR. En R no se suelen usar este tipo de ciclos, pero para simplificar sobre todo para posibles lectores que vengan de otros lenguajes, lo implementaremos de esa forma.

Utilizaremos un valor de K = 5.


```R
# Variable para almacenar los resultados
accuracy_list <- list()
# Iteramos los 5 folds
for(i in 1:5) {
      fold_index <- data_folds[[i]]
      X_train <- breast_cancer_n[-fold_index, !(colnames(breast_cancer_n) %in% "diagnosis")]
      X_test <- breast_cancer_n[fold_index, !(colnames(breast_cancer_n) %in% "diagnosis")]
      y_train <- breast_cancer_n[-fold_index, "diagnosis"]
      y_test <- breast_cancer_n[fold_index, "diagnosis"]
      pred <- knn(k = 5, train = X_train, test = X_test, cl = y_train)    
      cm <- as.matrix(table(Predicted = pred, Actual = y_test))
      # Calculo estadísticas del fold
      accuracy <- sum(diag(cm))/sum(cm)
      accuracy_list[[i]] <- accuracy     
}
```

Ahora vamos a ver el accuracy de cada uno de los 5 folds


```R
accuracy_list
```


<ol>
	<li>0,982456140350877</li>
	<li>0,965217391304348</li>
	<li>0,964601769911504</li>
	<li>0,964601769911504</li>
	<li>0,964912280701754</li>
</ol>



Podemos observer que la performance del algoritmo depende del fold, asi la accuracy va desde 96 a 98%.

A continuación la accuracy media y la desviación estandar para un valor de K = 5


```R
mean(unlist(accuracy_list))
sd(unlist(accuracy_list))
```


0,968357870435998



0,007885311


Bueno, esto ha sido todo por hoy. En los artículos futuros vamos a ir mejorando esto, y analizando otros datasets, hablaremos más de EDA, otras métricas para evaluar la performance del algoritmo, utilizaremos [caret](https://scikit-learn.org/stable/), un paquete de R muy bueno para machine learning, y también [scikit-learn](https://scikit-learn.org/stable/) en python.

