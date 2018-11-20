---
title:  "Introducción básica a numpy"
date:   2018-11-20
layout: single
author_profile: true
tags:
  - datascience
  - programación
  - Python
categories:
  - datascience
  - Python
toc: true
comments: true
---

[Numpy](http://www.numpy.org/) es un paquete con módulos que permiten una computación numérica eficiente en Python. Incluye tipos no presentes en el lenguaje Python básico como vectores multidimensionales.

Permite el manejo de datos de cualquier tipo simple. Las funciones operan elemento a elemento.


```python
import numpy as np
from math import sin
```

En Python básico


```python
x = [4, 5, 6]
y = [sin(i) for i in x]
y
```




    [-0.7568024953079282, -0.9589242746631385, -0.27941549819892586]



con numpy


```python
x = np.array(x)
y = np.sin(x)
y
```




    array([-0.7568025 , -0.95892427, -0.2794155 ])



algunos ejemplos


```python
x = np.array([[1,2,3],[4,5,6]], int)
print(x)
print("Shape")
print(x.shape)
print("itemsize")
print(x.itemsize)
```

    [[1 2 3]
     [4 5 6]]
    Shape
    (2, 3)
    itemsize
    8


## Listas vs numpy arrays

- Las listas de python pueden contener elementos de diferente tipo.
- Los arrays de numpy deben contener elementos del mismo tipo, y los array multidimensionales la misma cantidad de columnas


```python
# Lista de python
x = [1, 3, "hola"]
print(x)
print(type(x))
print(type(x[0]))
print(type(x[2]))
# Permite tipos de datos diferentes
```

    [1, 3, 'hola']
    <class 'list'>
    <class 'int'>
    <class 'str'>



```python
# Array de numpy
x = np.array([[1,2,'hola'],[3,4,5]])
print(x)
print(type(x))
print(type(x[0]))
print(type(x[0][0]))
print(type(x[0][2]))
# En este caso convirtio los números enteros a tipo string
```

    [['1' '2' 'hola']
     ['3' '4' '5']]
    <class 'numpy.ndarray'>
    <class 'numpy.ndarray'>
    <class 'numpy.str_'>
    <class 'numpy.str_'>


## Creación de vectores


```python
# Vector de ceros
x = np.zeros(3)
print(x)
```

    [0. 0. 0.]



```python
# Vector de unos
x = np.ones(5)
print(x)
print(type(x))
print(type(x[0]))
```

    [1. 1. 1. 1. 1.]
    <class 'numpy.ndarray'>
    <class 'numpy.float64'>



```python
# linspace(a, b, c) genera 'c' elementos entre 'a' y 'b', incluyendo a  'b'
x = np.arange(5, 20, 2)
print(x)
```

    [ 5  7  9 11 13 15 17 19]



```python
# arange(a, b, c) genera elementos entre 'a' y 'b', sin incluir a 'b', espaciados por 'c'
x = np.arange(5, 20, 2)
print(x)

print()

x = np.arange(5, 20, 1)
print(x)
```

    [ 5  7  9 11 13 15 17 19]
    
    [ 5  6  7  8  9 10 11 12 13 14 15 16 17 18 19]


## Asignación de elementos


```python
x = np.array([3,6,7,8])
print(x)
x[3] = 10
print(x)
```

    [3 6 7 8]
    [ 3  6  7 10]



```python
## Vector n dimensional
x = np.arange(10).reshape(2,5)
print(x)
print()
x[0][2] = 20
print(x)
print()
print("Dimensiones")
print(x.ndim)
```

    [[0 1 2 3 4]
     [5 6 7 8 9]]
    
    [[ 0  1 20  3  4]
     [ 5  6  7  8  9]]
    
    Dimensiones
    2



```python
# Acceso a la primer fila
print(x[0])
# Acceso a la primer columnas
print(x[:,0])
```

    [ 0  1 20  3  4]
    [0 5]


## Slicing


```python
print(x)
print()
print(x[0:2,0:2])
print()
print(x[0:2,-1])
print()
print(x[0:,2])
```

    [[ 0  1 20  3  4]
     [ 5  6  7  8  9]]
    
    [[0 1]
     [5 6]]
    
    [4 9]
    
    [20  7]


## Indexación indirecta


```python
x = np.arange(10)
print(x)
print()
y = x[[0,2,5]]
print(y)
print()
y = x[[0, -1]]
print(y)
```

    [0 1 2 3 4 5 6 7 8 9]
    
    [0 2 5]
    
    [0 9]


## Selección


```python
x = np.arange(20)
print(x)
print()
y = x[x < 10]
print(y)
print()
print(x[x == x.max()])
print()
x[x == x.max()] = 100
print(x)
print()
```

    [ 0  1  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17 18 19]
    
    [0 1 2 3 4 5 6 7 8 9]
    
    [19]
    
    [  0   1   2   3   4   5   6   7   8   9  10  11  12  13  14  15  16  17
      18 100]
    


## Trabajando con arrays


```python
x = np.arange(36)
print(x)
```

    [ 0  1  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17 18 19 20 21 22 23
     24 25 26 27 28 29 30 31 32 33 34 35]



```python
x = x.reshape(6, 6)
print(x)
```

    [[ 0  1  2  3  4  5]
     [ 6  7  8  9 10 11]
     [12 13 14 15 16 17]
     [18 19 20 21 22 23]
     [24 25 26 27 28 29]
     [30 31 32 33 34 35]]


## Operaciones sobre vectores


```python
x = np.arange(10)
print(x)
print()
y = 3*x + 5
print(y)
print()
```

    [0 1 2 3 4 5 6 7 8 9]
    
    [ 5  8 11 14 17 20 23 26 29 32]
    



```python
# Operaciónes a través de un eje
x = np.arange(10)
x = x.reshape(2,5)

# suma
print(x)
print()
print(np.sum(x, axis = 0)) # Suma a traves de las FILAS, produce los totales de COLUMNAS
print()
print(np.sum(x, axis = 1)) # Suma a traves de las COLUMNAS, produce los totales de FILAS
print()
```

    [[0 1 2 3 4]
     [5 6 7 8 9]]
    
    [ 5  7  9 11 13]
    
    [10 35]
    



```python
# minimo
print(x.min(axis = 0))
print()
print(x.min(axis = 1))
```

    [0 1 2 3 4]
    
    [0 5]



```python
# media
print(x.mean(axis = 0))
print()
print(x.mean(axis = 1))
```

    [2.5 3.5 4.5 5.5 6.5]
    
    [2. 7.]


## Eficiencia

La implementación de las operaciones matemáticas en los arrays de numpy son mas eficientes que su contraparte en Python clásico.


```python
# Python clásico
x = range(10000)
%timeit [i**2 for i in x]
```

    3.5 ms ± 69 µs per loop (mean ± std. dev. of 7 runs, 100 loops each)



```python
x = np.arange(10000)
%timeit x**2
```
