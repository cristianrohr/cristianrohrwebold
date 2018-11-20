---
title:  "Curriculum vitae en Rmarkdown y latex"
date:   2018-11-18
layout: single
author_profile: true
comments: true
tags:
  - datascience
  - tecnologia
  - programación
  - R
  - markdown
categories:
  - datascience
  - R
---

# Crear y mantener un CV

Mantener el CV es una de esas tareas tanto necesarias, como en muchos casos tediosas, y que generalmente consumen mucho mas del tiempo del que queremos y deberíamos.
En mi caso particular use muchísimos modelos diferentes, y los hice en diferentes plataformas, desde el famoso editor de texto de la marca de las ventanas, markdown, latex, editores online y no se si no me olvido alguna otra version. Mis ultimas versiones fueron en Latex con el template [Awesome CV](https://github.com/posquit0/Awesome-CV).

Con motivo de la creación del blog decidí una vez más realizar la actualización del templado de mi CV, pero decidido a que esta vez al menos me dure un par de años (ojala que al menos 5). Por este motivo decidí migrarme nuevamente a [markdown](https://daringfireball.net/projects/markdown/syntax), por dos motivos: a) la facilidad de edición, b) porque utilizando [Rmarkdown](https://rmarkdown.rstudio.com/) puedo mezclar facilmente markdown y latex para añadir una capa extra de customización al CV.

Finalmente despues de un rato de búsqueda decidí usar como base el template de [isteves](https://github.com/isteves/resume) 

![iecv](/assets/img/cv_ie.jpg)

Lo estuve modificando bastante, agregando estilos nuevos de latex, y realizando la escritura mezclando latex y markdown. El resultado final se puede ver en [este repo](https://github.com/cristianrohr/CV_Rmarkdown).

![crcv](/assets/img/new_CR.png)

## Modificar el template

El archivo `.Rmd` contiene el codigo para generar el CV. La cabecera del archivo en formato `yaml` contiene la información basica que debe ser modificada. Esta información se pasa al template de latex `cv-latex.tex`, donde debería modificarse la forma o el orden en el que se muestra esta información.

El resto del documento esta escrito mezclando markdown con latex.

```kramdown
---
output: 
  pdf_document:
    latex_engine: xelatex
    template: cv-latex.tex
geometry: margin=.7in

#bibliography: citations.bib
#nocite: '@*'

title: "Curriculum Vitae"
author: Cristian Rohr
name: Cristian
lastname: Rohr
jobtitle: Bioinformático 
address: Barrichuelo de Cartuja N°15 Granada, España

fontawesome: yes
email: cristianrohr768@gmail.com - cristian.rohr@heritas.com.ar
github: your_github
linkedin: cristianrohrbio
twitter: your_twitter
phone: "+34 667283297"
web: cristianrohr.github.io
updated: no

keywords: Data Science, Bioinformatics

fontsize: 10pt # Valores validos entre 10 y 12pt
urlcolor: NavyBlue
---
```
