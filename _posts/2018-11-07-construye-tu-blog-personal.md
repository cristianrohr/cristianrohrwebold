---
title:  "Como crear un sitio web personal + blog con github pages y jekyll"
date:   2018-11-16
layout: single
author_profile: true
comments: true
tags:
  - blog
  - web
  - tecnologia
  - programación
categories:
  - blog
  - programación
---

Para mi primer post voy a comentar un poco las razones que me llevaron a hacerlo. Lo más importante es que hace mucho tiempo tenía ganas de tener mi blog sobre bioinformática, por un lado porque me apasiona y por otro lado me parece un recurso util que le puede servir a muchas personas de la misma forma que a mi me han servido muchos blogs y páginas.

Obviamente al decidirse a escribir un blog la primer pregunta que surge es: que tecnología utilizo??. Tenías vistas las paginas web personales y de proyectos  hosteadas en [**Github**](https://github.com). Leyendo encontre el muy utilizado combo de [**Jekyll**](https://jekyllrb.com/) más Github Pages y el aparentemente archi famoso tema de Jekyll [**minimal-mistakes**](https://mmistakes.github.io/minimal-mistakes/).

A continuación voy a comentar los pasos necesarios para la creación de un sitio con estas tecnologías en un entorno Linux. En mi caso particular utilizo la distro [**Linux Mint**](https://linuxmint.com/).

### Recurso de interés:
+ Aprendiendo sobre Github Pages: [Github hosting](http://jmcglone.com/guides/github-pages/)

## Paso 1: Instalar _Jekyll_ - _minimal-mistakes_ y generar el template del sitio

### Instalar _Jekyll_
+ Tipear en el terminal `gem install jekyll` para la instalación. Asumiendo que tenemos todas las dependencias debería funcionar. Si tienen problemas, chequear -> [jekyll install](https://jekyllrb.com/docs/installation/).

### Instalar el tema _minimal-mistakes_
+ Fork el repo [Minimal Mistakes](https://github.com/mmistakes/minimal-mistakes/fork)
+ Clonar el repo y renombrarlo. Si van a utilizar github pages para hostear usen USERNAME.github.io, donde USERNAME es su usuario de github.
+ Ingresen al directorio y ejecuten `gem install bundler`
+ Ejecuten `bundle install`, esto instala las dependencias necesarias.
+ Ejecuten `bundle exec jekyll serve`, y el template del sitio debería estar disponible accediendo a la siguiente dirección del navegador [localhost:4000/](localhost:4000/).
+ Luego sigan los pasos en la guía para configurar diferentes opciones [configuration](https://mmistakes.github.io/minimal-mistakes/docs/configuration/). Basicamente consiste en modificar el archivo `_config.yml`

## Paso 2: agregar páginas

Las páginas se especifican en el archivo `_data/navigation.yml`:

```yml
main:
  - title: "Quick-Start Guide"
    url: /docs/quick-start-guide/
  - title: "Posts"
    url: /year-archive/
  - title: "Categories"
    url: /categories/
  - title: "Tags"
    url: /tags/
  - title: "Pages"
    url: /page-archive/
  - title: "Collections"
    url: /collection-archive/
  - title: "External Link"
    url: https://google.com
```

Cada `-` corresponde a un tab de la página:

- `title` es el título.
- `url` es el link al archivo que contiene el contenido de la página en `markdown`.

Por ejemplo, para agregar la página `Blogs`, solo hay que agregar dos líneas al archivo `navigation.yml`:

```yml
	- title: "Blogs"
	  url: /Blogs/
```

Luego creamos un directorio llamado `/_pages/`, y adentro creamos un archivo llamado `blogs.md` con el siguiente contenido:

```md
---
title:  "Blogs"
layout: archive
permalink: /Blogs/
author_profile: true
comments: true
---

Esta es mi página de blog.
```

Asegurarse que el `permalink:` coincida con la `url` en el archivo `navigation.yml`.

## Paso 3: agregar posts

### Posts
Los posts se deben colocar en el directorio `_posts` y llamarse de acuerdo al siguiente esquema `AÑO-MES-DIA-nombrearchivo.md` de esta forma _minimal-mistakes_ los puede identificar de forma automática.

Un ejemplo de post en markdown:

```md
---
layout: single
title:  "Mi primer post"
date:   2016-11-11
---

Mi primer post que copado!!!
```

donde `layout:single` indica que es un post simple; `title` aparece como título de pagina; `date` indica la última actualización. Además hay otros parámetros para configurar: [más parámetros](https://mmistakes.github.io/minimal-mistakes/docs/posts/).



## Paso 4: hostear en Github
[Github pages](https://pages.github.com/) es una gran herramienta GRATUITA para hostear. Si se crea un repositorio llamado `username.github.io`, automáticamente seva a convertir a pagina web y estará disponible en la URL `http://username.github.io`. 

+ Creamos el repositorio USERNAME.github.io, recuerden cambiar USERNAME por su usuario.
+ Ingresamos a nuestra carpeta local donde estuvimos trabajando

```bash
# iniciamos el repositorio de git
git init

# agregamos el repositorio remoto
git remote add origin "ADDRESS.OF.YOUR.GITHUB.REPOSITORY"

# agregamos los cambios y realizamos el commit
git add .
git commit -a -m "first commit"

# pusheamos los cambios
git push origin master
```


Si todo esta bien, deberíamos poder ingresar a "http://username.github.io"!
