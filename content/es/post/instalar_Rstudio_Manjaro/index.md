+++
title = "Instalar RStudio en Manjaro Linux y mejorar el rendimiento de R con OpenBLAS"
subtitle = ""

# Add a summary to display on homepage (optional).
summary = ""

date = 2019-04-05T12:22:39+02:00
draft = false

# Authors. Comma separated list, e.g. `["Bob Smith", "David Jones"]`.
authors = ["Jorge Saturno"]

# Is this a featured post? (true/false)
featured = false

# Tags and categories
# For example, use `tags = []` for no tags, or the form `tags = ["A Tag", "Another Tag"]` for one or more tags.
tags = ["rstudio", "manjaro"]
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
  caption = "mararie, Flickr, CC BY SA"

  # Focal point (optional)
  # Options: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight
  focal_point = ""
+++

[RStudio](https://www.rstudio.com/) es el entorno de desarrollo para R más popular hoy en día. Su instalación en Manjaro (y otras distribuciones basadas en Arch Linux) es muy sencilla pero requiere usar el repositorio mantenidos por los usuarios, conocido como Arch User Repository (AUR). Usualmente, se usaba *yaourt* para instalar paquetes del AUR pero dado que este fue descontinuado, tendremos que instalar *yay*, uno de sus sustitutos.

## Instalar *yay*
```
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si
```

## Instalar RStudio usando *yay*
```
yay -S rstudio-desktop-bin
```

## Instalación de OpenBLAS
Las librerías de BLAS instaladas por defecto en Manjaro no son las de mejor rendimiento. Para incrementar el rendimiento, se recomienda instalar [*OpenBLAS*](http://www.openblas.net/), el cual ofrece la posibilidad de usar múltiples núcleos del CPU en vez de uno solo, como normalmente ocurre.
Para instalar OpenBLAS:
```
yay -S openblas-lapack
```
Durante el proceso de instalación tendrás que remover los paquetes antiguos. Luego de culminada la instalación no es necesario configurar nada. R estará usando los CPU disponibles.

## Una última recomendación
Muchos paquetes de R requerirán un compilador de Fortran. Luego de la instalación es recomendable instalar gcc-fortran:
```
sudo pacman -S gcc-fortran
```

{{% alert note %}}
Si por desgracia usas Windows, puedes instalar RStudio siguiendo este [tutorial](https://www.youtube.com/watch?v=IyJuTU82ikM).
{{% /alert %}}
