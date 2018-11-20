---
title:  "Curriculum vitae en Rmarkdown y latex"
date:   2018-11-18
layout: single
author_profile: true
tags:
  - datascience
  - tecnologia
  - programación
  - R
  - markdown
categories:
  - datascience
  - R
comments: true
---

# Crear y mantener un CV

Mantener el CV es una de esas tareas tanto necesarias, como en muchos casos tediosas, y que generalmente consumen mucho mas del tiempo del que queremos y deberíamos.
En mi caso particular use muchísimos modelos diferentes, y los hice en diferentes plataformas, desde el famoso editor de texto de la marca de las ventanas, markdown, latex, editores online y no se si no me olvido alguna otra version. Mis ultimas versiones fueron en Latex con el template [Awesome CV](https://github.com/posquit0/Awesome-CV).

Con motivo de la creación del blog decidí una vez más realizar la actualización del templado de mi CV, pero decidido a que esta vez al menos me dure un par de años (ojala que al menos 5). Por este motivo decidí migrarme nuevamente a [markdown](https://www.markdownguide.org/basic-syntax/), por dos motivos: a) la facilidad de edición, b) porque utilizando [Rmarkdown](https://rmarkdown.rstudio.com/) puedo mezclar facilmente markdown y latex para añadir una capa extra de customización al CV.

Finalmente despues de un rato de búsqueda decidí usar como base el template de [isteves](https://github.com/isteves/resume) 

![iecv](/assets/img/cv_ie.jpg)

Lo estuve modificando bastante, agregando estilos nuevos de latex, y realizando la escritura mezclando latex y markdown. El resultado final se puede ver en [este repo](https://github.com/cristianrohr/CV_Rmarkdown).

![crcv](/assets/img/new_CR.png)

## Modificar el template

El archivo `.Rmd` contiene el codigo para generar el CV. La cabecera del archivo en formato `yaml` contiene la información basica que debe ser modificada. Esta información se pasa al template de latex `cv-latex.tex`, donde debería modificarse la forma o el orden en el que se muestra esta información.

```md
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
address: Mi domicilio

fontawesome: yes
email: mail@mail.com
github: your_github
linkedin: cristianrohrbio
twitter: your_twitter
phone: "+55 5555555"
web: cristianrohr.github.io
updated: no

keywords: Data Science, Bioinformatics

fontsize: 10pt # Valores validos entre 10 y 12pt
urlcolor: NavyBlue
---
```

El resto del documento esta escrito mezclando markdown con latex y puede ser editado en estos dos lenguajes.

```md
# Experiencia Profesional 

\textbf{\textcolor{coolblack} {Líder Bioinformática}} | \textbf{Héritas, Rosario, Argentina} \hfill \textcolor{mygray} {Oct. 2018 - Actualidad}

\textbf{\textcolor{coolblack} {Analista Bioinformático}} | \textbf{Héritas, Rosario, Argentina} \hfill \textcolor{mygray} {May 2016 - Sep. 2018}

- En Héritas mi tarea principal como analista bioinformático fue el desarrollo e implementación de los algoritmos y modelos predictivos del test prenatal no invasivo [Héritas VISION](http://heritas.com.ar/genomica-de-la-reproduccion/vision/), y del test de microbioma intestinal [Héritas MicroXplora](http://heritas.com.ar/microxplora/). Además de estar fuertemente involucrado en el desarrollo de herramientas en Shiny/R para automatizar tareas y asistir a los análistas genéticos, en diversos productos como el test de biopsías líquidas [Héritas OncoSens](http://heritas.com.ar/genomica-clinica/oncosens/), [Héritas Focus](http://heritas.com.ar/genomica-clinica/exoma-clinico/), [Héritas CLEAR](http://heritas.com.ar/genomica-clinica/cancer-hereditario/)
- Diseño, desarrollo, implementación y mantenimiento de los pipelines de análisis en un entorno de computo de alto rendimiento para la entrega de resultados en tiempo y forma.
```

## Bonus:
Podemos importar bibliografía en formato bibtex, la cual pueden ser nuestras publicaciones.

Para esto, si tenemos [google scholar](https://scholar.google.es/citations?user=hyLMQvAAAAAJ&hl=en) podemos exportar nuestras publicaciones en formato bibtex,
guardamos el archivo en la carpeta del template del CV, como citations.bib, y el yaml del template descomentamos las siguientes dos lineas


```md
#bibliography: citations.bib
#nocite: '@*'
```

de esta forma nuestras publicaciones se ubicaran al final de CV, lo cual facilita el mantenimiento del mismo.
