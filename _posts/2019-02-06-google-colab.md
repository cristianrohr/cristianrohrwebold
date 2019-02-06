---
title: "Google colab, o como usar una placa tesla sin pagar por ella"
excerpt: Google Colaboratory es una herramienta de Google, que permite la ejecución de código python mediante jupyter notebooks en un entorno virtual equipado con una GPU Tesla K80.
tags:
  - datascience
  - machine learning
  - programación
  - python
  - cloud
  - google
categories:
  - datascience
  - python
  - machine learning
  - cloud
toc: true
comments: true
header:
  overlay_image: /assets/images/cool-background-knn.png
caption: "Imagen: [coolbackgrounds.io](https://coolbackgrounds.io/)"
last_modified_at: 2019-02-06
---


## Introducción
[Google Colaboratoy](https://colab.research.google.com/) resulta una excelente forma de reducir la barrera de entrada al mundo del machine learning, ya que en muchas ocasiones sin el hardware apropiado, resulta imposible ( o sencillamente demasiado costoso en tiempo) implementar modelos de deep learning. En mi caso particular incluso de tareas más simples ya que mi vieja Zenbook debe jubilarse.

Cuando comenzamos a trabajar, vemos un notebook que recuerda a los notebooks de [Jupyter](https://jupyter.org/), en realidad es un notebook de Jupyter modificado, así que no es algo ajeno a cualquiera con un poco de experiencia. Los notebooks que generemos se pueden compartir como si se tratara de archivos alojados en Drive.

![gcf](/assets/images/googlecolabfirst.png)

En esta pantalla se puede sencillamente dar cancelar y el menú File crear un nuevo notebook.

Ya en el notebook se podrá ver la familiar interfaz de celdas al estilo Jupyter.

![gcc](/assets/images/googlecolabcell.png)

En el ejemplo de arriba muestro dos celdas, en la primera el comando para montar nuestra carpeta de google drive.

```python
from google.colab import drive
drive.mount('/content/gdrive')
```


De forma tal que su contenido se pueda usar en nuestro notebook, de esta forma podemos subir nuestros datasets a google drive y utilizarlos para trabajar en ellos.
Al momento de montar la carpeta nos pide que pinchemos un link para autenticarnos y luego que ingresemos un codigo en una celda, de esta forma ya tendremos acceso a nuestros archivos.
Existen otras formas de subir archivos, las cuales no comentare en esta ocasión.

Al igual que en los notebooks de jupyter podemos usar comandos al estilo unix.

```bash
!ls "/content/gdrive/My Drive"
```

anteponiendo el símbolo '!'

Para poder acceder a los archivos y carpetas solo es necesario utilizar el path de forma correcta, como en el siguiente ejemplo.

![gcc](/assets/images/googlecolabfiles.png)

Y luego solo es necesario darle vida a nuestro proyecto de machine learning cargando las líbrerias necesarias y poniendose a picar el código para nuestros modelos

![gcc](/assets/images/googlecolabcode.png)

De forma estándar solo soporta kernels de Python, pero es posible utilizar [R](https://www.r-project.org/) -> [Usar R](https://discourse.mc-stan.org/t/r-jupyter-notebook-rstan-on-google-colab/6101) y [Julia](https://julialang.org/) -> [Usar Julia](https://discourse.julialang.org/t/julia-on-google-colab-free-gpu-accelerated-shareable-notebooks/15319)


## Conclusión
SIn lugar a dudas es muy interesante la idea de Google colab por dos razones principales, la primera poner a disposición una placa Tesla por la que no debemos pagar dinero, y segundo la portabilidad de los notebooks.