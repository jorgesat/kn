+++
title = "Crear un blog académico gratuito con hugo y GitHub"
date = 2019-01-05T11:55:52+01:00
draft = false

# Authors. Comma separated list, e.g. `["Bob Smith", "David Jones"]`.
authors = ["Jorge Saturno"]

# Tags and categories
# For example, use `tags = []` for no tags, or the form `tags = ["A Tag", "Another Tag"]` for one or more tags.
tags = ["blog", "hugo"]
categories = []

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["deep-learning"]` references
#   `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
# projects = ["internal-project"]

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
[image]
  # Caption (optional)
  caption = "Fatos Bytyqi, CC BY"

  # Focal point (optional)
  # Options: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight
  focal_point = ""
+++

Esta entrada es un tutorial para crear un sitio web personal con CV y blog usando el tema [**Academic**](https://sourcethemes.com/academic/) para *hugo*. Existen muchas alternativas para crear un blog. Quizás las más conocidas sean *wordpress* y *blogspot*. Recientemente, mucha gente está optando por *medium* para bloguear. Aunque *medium* podría ofrecer la posibilidad de llegar a una audiencia más amplia, tiene grandes desventajas, ya que es una plataforma cerrada e inflexible. :thumbsdown:

Usualmente los blogs son sitios dinámicos (un servidor consulta una base de datos y produce el contenido dependiendo de lo que el usuario quiera leer). Con [*hugo*](https://picodotdev.github.io/blog-bitix/2018/05/generador-de-paginas-web-estaticas-y-bitacoras-con-hugo/) podemos crear un sitio web completamente estático (sin bases de datos) y usarlo como blog. ¿Las ventajas? Es mucho más ligero y no necesitas pagar un costoso hospedaje o tener un sitio web gratuito lleno de publicidad. De hecho, con *hugo* puedes crear un sitio web profesional y atractivo y alojarlo gratuitamente en *GitHub* o *GitLab*. Sólo tendrías que pagar si quisieras tener un dominio personalizado pero no es necesario. Otra de las ventajas de *hugo* es que no requiere que aprendas a programar. *Hugo* ofrece la posibilidad de crear tu sitio web usando el lenguaje go, a partir de archivos de texto (con formato markdown) que son muy fáciles de crear y editar.

{{% alert note %}}
Advertencia: No soy programador de carrera y puede que algunos de los términos usados en este artículo no sean los técnicamente correctos.
{{% /alert %}}

¿Qué necesitas para crear el sitio académico con *hugo*?

- Una cuenta en *GitHub* o *GitLab* donde hospedarás tu sitio.
- Instalar *hugo* en tu computadora.
- Programa para editar archivos de texto.
- Lista de publicaciones académicas en un archivo *bibtex*.

Ya que vamos a crear un blog académico como el que estás leyendo, usaremos el tema **Academic** de *hugo* -También puedes usar otros temas que puedes encontrar en la [galería de temas de *hugo*](https://themes.gohugo.io/)-

## Instalar hugo

La instalación de *hugo* depende del sistema operativo que uses. Puedes encontrar detalles de distintas instalaciones en inglés en este [sitio](https://gohugo.io/getting-started/installing/#quick-install) o [acá](https://blog.adrianistan.eu/2016/05/26/tutorial-hugo-espanol-generador-sitios-estaticos/), en español.

En Arch Linux o Manjaro puedes ejecutar

```Shell
sudo pacman -Syu hugo
```

En Ubuntu o Debian,

```Shell
sudo apt-get install hugo
```

## Preparar la plantilla **Academic**

Lo primero que haremos es clonar el tema **Academic** y guardarlo localmente en nuestra computadora. Para ello, ejecuta los siguientes comandos:

```Shell
git clone https://GitHub.com/sourcethemes/academic-kickstart.git Mi_sitio
cd Mi_sitio
git submodule update --init --recursive
```

Luego se debe crear un nuevo repositorio en *GitHub* o *GitLab*. En el caso de usar *GitHub*, el repositorio debería llamarse `<usuario>.github.io`. Este es el repositorio donde el sitio final será alojado. Agrega este repositorio a un submódulo de la siguiente manera:

```Shell
git submodule add -f -b master https://github.com/<usuario>/<usuario>.github.io.git public
```

Luego, sincronizamos el repositorio local con el remoto en *GitHub*:

```Shell
git add .
git commit -m "Primer commit"
git push -u origin master
```

El último paso sería crear el sitio web como tal, en HTML. Esta tarea se conoce como "renderizar" el sitio y lo hará *hugo* para nosotros. Además, el sitio web creado será guardado en la carpeta `/public` y esta carpeta es la que vamos a sincronizar con el repositorio `<usuario>.github.io`, de la siguiente forma:

```Shell
hugo
cd public
git add .
git commit -m "Crear sitio"
git push origin master
cd ..
```

{{% alert note %}}
Ten en cuenta que estos dos últimos pasos son los que necesitarás ejecutar cada vez que quieras actualizar tu sitio, por ejemplo, para agregar una nueva entrada.
{{% /alert %}}

## Primeros pasos para editar el sitio web

Lo primero que debemos hacer es abrir el archivo `config.toml` y cambiar las variables necesarias. Por ejemplo, podemos cambiar el nombre del sitio, nuestro nombre y biografía, etc.

```TOML
baseurl = "https://<usuario>.github.io" # Usa este URL si piensas usar GitHub

# Title of your site
title = "Blog académico de Perico de los Palotes" # Acá cambias el título de tu sitio
```

Si deseas que el sitio esté en español modifica la siguiente variables:

```TOML
defaultContentLanguage = "es"
```

Hay muchas otras variables que puedes cambiar en `config.toml`, por ejemplo, tus cuentas de redes sociales, o tu foto de perfil. Recomiendo que inspecciones todo el archivo y experimentes con él.

Una vez listos estos cambios podemos crear el sitio usando el comando `hugo`. Luego de creado el sitio, puedes visualizarlo ejecutando el comando `hugo server`. Esta es una vista preliminar. Puedes cambiar otras variables o seguir los siguientes pasos para agregar tus publicaciones y tu primera entrada al blog. Es recomendable visualizarlo localmente antes de sincronizar con *GitHub*. Para ello, abre la siguiente dirección en el navegador: `localhost:1313`.

## Crear lista de publicaciones académicas

Necesitas tener una lista de tus publicaciones como archivo *bibtex*. Esto puedes hacerlo desde tu gestor de referencias (zotero, [mendeley](http://tuxtor.shekalug.org/generar-referencias-bibtex-con-mendeley/), etc.) Luego, con el archivo *bibtex*, podrás producir los archivos .md requeridos para mostrar cada publicación, los cuales se localizarán en el directorio `content/publication`. No te preocupes que este proceso se puede hacer fácilmente usando un código de python que prepararon los desarrolladores.

Si tienes instalado Python 3, ve al directorio de tu sitio academic y ejecuta el script de la siguiente forma:

```python
pip3 install -U academic
academic import --bibtex <directorio/publicaciones.bib>
```

Si no tienes Python instalado, [esta guía](https://tutorial.djangogirls.org/es/python_installation/) te puede ayudar.

## Crear primera entrada en el blog

Lo que tienes que hacer primero es crear un archivo "primera_entrada.md" (por sugerir un nombre) en el directorio `content/post` y editarlo.

```Shell
hugo new --kind post post/primera_entrada
```

Abre el archivo .md en tu editor de texto y escribe algún texto usando el [formato markdown](https://markdown.es/sintaxis-markdown/). Una vez escrita la entrada, podemos ejecutar `hugo server` y visualizar el sitio abriendo `localhost:1313` en el navegador.

El último paso es cargar los archivos en el repositorio *GitHub* o *GitLab*, tal como se mostró al principio. Luego de cargados los archivos al repositorio, tu sitio web será visible en la URL "[usuario].github.io".
