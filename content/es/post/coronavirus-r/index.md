---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Seguimiento de la pandemia de COVID-19 con R"
subtitle: "Paquetes y visualizaciones"
summary: "Paquetes y visualizaciones en R sobre la pandemia de COVID-19. Distribución
global de casos y evolución en Latinoamérica."
authors: [Jorge Saturno]
tags: ["coronavirus", "covid-19", "pandemia","r"]
categories: []
date: 2020-03-28T19:33:59+01:00
lastmod: 2020-03-28T19:33:59+01:00
featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: "Jorge Saturno, CC-BY 4.0"
  focal_point: ""
  preview_only: false

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---

Probablemente sientes que estamos inundados con información sobre la evolución de la pandemia de COVID-19. Sin embargo, nada mejor que manejar los datos uno mismo y analizarlos desde distintos puntos de vista. Creo que siempre hay alguien que observará algo en los datos que se nos ha escapado a millones de observadores y creo que cualquier aporte en esta emergencia mundial es valioso. Aquí dejo algunos de los recursos que he estado usando en los últimos días para seguir la evolución de casos en el mundo, especialmente en Latinoamérica.

## De dónde vienen los datos
La fuente de datos más completa de casos de COVID-19 es la que provee la Universidad John Hopkins (JHU) y que actualizan varias veces al día en su [repositorio](https://github.com/CSSEGISandData/COVID-19) de GitHub. Estos datos son organizados por la universidad usando como fuentes la Organización Mundial de la Salud y fuentes nacionales (ministerios de salud y otras fuentes oficiales). Estos datos están divididos en tres archivos, los cuales contienen series de tiempo de casos confirmados, fallecidos y recuperados. Las tablas incluyen la localización hasta el nivel provincia/estado, con coordenadas geográficas. Desde el 22/03/2020, JHU dejó de reportar el número de recuperados.

A pesar de que podríamos cargar estos datos en R directamente desde el repositorio de la JHU, existen formas más cómodas de hacerlo. Actualmente existen dos paquetes de R que pueden hacerlo por nosotros.

## Paquetes de R dedicados a la pandemia de COVID-19

Existe un par de paquetes de R de los que tengo conocimiento: `coronavirus` y `nCov2019`. He trabajado con ambos por varios días y puedo confirmar que ambos están siendo actualizados a diario (hasta el día de esta publicación).

### Paquete `coronavirus`

El paquete [`coronavirus`](https://ramikrispin.github.io/coronavirus/) es desarrollado por [Rami Krispin](https://ramikrispin.github.io/), un científico de datos de California, Estados Unidos. Este paquete es muy práctico porque carga los datos de la Universidad John Hopkins y los ordena en formato tidy para ser usado en R fácilmente.

Para instalar este paquete:

``` r
devtools::install_github("RamiKrispin/coronavirus")
```

Es necesario instalar el paquete de GitHub y no el disponible en CRAN para obtener los datos más actuales.

Una vista rápida a los datos provistos por este paquete:

```r
require(knitr)
kable(head(coronavirus), row.names = FALSE, align = "c", caption = NULL)
```

| Province.State | Country.Region | Lat | Long |    date    | cases |   type    |
|:--------------:|:--------------:|:---:|:----:|:----------:|:-----:|:---------:|
|                |  Afghanistan   | 33  |  65  | 2020-01-22 |   0   | confirmed |
|                |  Afghanistan   | 33  |  65  | 2020-01-23 |   0   | confirmed |
|                |  Afghanistan   | 33  |  65  | 2020-01-24 |   0   | confirmed |
|                |  Afghanistan   | 33  |  65  | 2020-01-25 |   0   | confirmed |
|                |  Afghanistan   | 33  |  65  | 2020-01-26 |   0   | confirmed |
|                |  Afghanistan   | 33  |  65  | 2020-01-27 |   0   | confirmed |

Lo que más me ha gustado de este paquete, además del formato tidy, es que incorpora latitud y longitud de cada sitio y esto simplifica mucho las cosas.

### Distribución geográfica de los casos y tasa de letalidad

Usando los datos disponibles podremos hacer un mapa de los casos acumulados hasta el momento y la tasa de letalidad.

Para calcular los casos acumulados y la tasa de letalidad:

``` r
require(tidyverse)

datacov <- coronavirus %>%
  select(country = Country.Region, type, cases, Lat, Long) %>%
  group_by(country, Lat, Long, type) %>%
  summarise(total_cases = sum(cases)) %>%
  pivot_wider(names_from = type, values_from = total_cases) %>%
  arrange(-confirmed) %>%
  mutate(deadRate = death/confirmed * 100)
```

Con estos datos organizados podemos graficar el mapa de la siguiente forma:

```r
require(maps, hrbrthemes, viridis)

fecha <- format(max(coronavirus$date), "%d/%m/%Y")
mundo <- map_data("world")
cortes <- c(1, 20, 100, 1000, 5000, 10000, 30000)

p1 <- ggplot() +
  geom_polygon(data = mundo, aes(x=long, y = lat, group = group), fill="grey50",
               alpha=0.3) +
  geom_point(data = datacov, aes(x=Long, y=Lat, size=confirmed, color=deadRate),
             stroke=F, alpha=0.7) +
  scale_size_continuous(name="Casos", range=c(4,24),breaks=cortes,
                          labels = c("1-19", "20-99", "100-999", "1.000-4.999",
                                     "5.000-9.999", "10.000-29.999","30.000+")
                                     ) +
  scale_color_viridis(option="magma", name="Tasa de letalidad (%)",
                      limits=c(0,10)) +
  labs(title = paste0("Casos de COVID-19 al ", fecha),
       subtitle = "Número de casos y tasa de letalidad") +
  theme_ipsum_rc(grid = F) +
  theme(
    axis.text.x = element_blank(),
    axis.text.y = element_blank(),
    axis.title.x = element_blank(),
    axis.title.y = element_blank()
  )

print(p1)

```

{{< figure library="true" src="COVID19_mapa_20200327.png" title="Mapa de casos de COVID-19 al 27/03/2020 usando datos provistos por el paquete de R `coronavirus`" lightbox="true" >}}

Cabe destacar que la tasa de letalidad puede ser una cifra engañosa, ya que no todos los países están diagnosticando el COVID-19 de la misma forma y la gran mayoría de los países reportan menos números de casos de los que realmente existen. Este [estudio](https://cmmid.github.io/topics/covid19/severity/global_cfr_estimates.html) muestra el porcentaje de casos que están reportando varios países, casi todos por debajo del 50 %.

### Paquete `nCov2019`

El paquete [`nCov2019`](https://guangchuangyu.github.io/nCov2019/) es desarrollado por [Guangchuang Yu](https://github.com/GuangchuangYu), un profesor de bioinformática de la Universidad Médica del Sur, en Guangzhou, China. Este paquete provee varios sets de datos con distintos niveles de resolución geográfica, uno _global_ donde se detallan los casos por países, hasta uno más detallado que llega a escala de ciudades. Es un paquete muy bien organizado pero le encuentro dos detalles que no me han gustado: no viene en formato tidy (esto es cuestión de gustos) y no provee las coordenadas geográficas. Esto último se puede resolver usando `ggmap::geocode`, función con la cual se pueden obtener las coordenadas de las ciudades desde _Google Maps_. En este caso, se requiere de una Google API key.

Desde este paquete he extraído los datos de casos en países latinoamericanos y he calculado la cantidad de casos confirmados a partir del día en que cada país sobrepasó los 100 casos:

``` r
require(tidyverse)

# Países latinos
latinos <- c("Argentina", "Bolivia", "Brazil", "Chile", "Colombia", "Costa Rica",
             "Cuba", "Ecuador", "El Salvador", "Guatemala", "Haiti", "Honduras",
             "Mexico", "Nicaragua", "Panama", "Paraguay", "Peru", "Dominican Republic",
             "Uruguay", "Venezuela", "Puerto Rico")

d <- load_nCov2019() # Carga los datos más recientes
dd <- d['global'] %>%
  as_tibble %>%
  rename(confirm=cum_confirm) %>%
  filter(confirm > 100 & country %in% latinos) %>%
  group_by(country) %>%
  mutate(days_since_100 = as.numeric(time - min(time))) %>%
  ungroup


```

Para cambiar los nombres de los países a español (en los que se requiere):

```r
dd <- dd %>%
  mutate(country = ifelse(country == "Peru", "Perú", country)) %>%
  mutate(country = ifelse(country == "Dominican Republic", "Rep. Dominicana", country)) %>%
  mutate(country = ifelse(country == "Brazil", "Brasil", country)) %>%
  mutate(country = ifelse(country == "Haiti", "Haití", country)) %>%
  mutate(country = ifelse(country == "Panama", "Panamá", country)) %>%
  mutate(country = ifelse(country == "Mexico", "México", country))
```

Un vistazo a los datos muestra lo siguiente:

```r
kable(dd, row.names = FALSE, align = "c", caption = NULL)
```

|    time    |     country     | confirm | cum_heal | cum_dead | days_since_100 |
|:----------:|:---------------:|:-------:|:--------:|:--------:|:--------------:|
| 2020-03-14 |     Brasil      |   151   |    0     |    0     |       0        |
| 2020-03-15 |     Brasil      |   151   |    0     |    0     |       1        |
| 2020-03-16 |     Brasil      |   203   |    1     |    0     |       2        |
| 2020-03-17 |     Brasil      |   234   |    2     |    0     |       3        |
| 2020-03-17 |      Chile      |   156   |    0     |    0     |       0        |
| 2020-03-18 |     Brasil      |   349   |    2     |    1     |       4        |
| 2020-03-18 |      Chile      |   201   |    0     |    0     |       1        |
| 2020-03-18 |     Ecuador     |   111   |    0     |    2     |       0        |

La variable _confirm_ muestra casos confirmados acumulados hasta la fecha.

### Evolución de casos en Latinoamérica y tiempo de duplicación

Antes de mostrar la serie de tiempo de los casos confirmados en cada país calculamos el crecimiento exponencial asumiendo dos escenarios: 1) duplicación de casos cada 2 días, y 2) duplicación de casos cada 3 días.

Partiendo de la siguiente ecuación de crecimiento exponencial:

$$ N_t = N_0 e^{rt} \ $$

donde $N_t$ es el número de casos en un tiempo $t > 0$, $N_0$ es el número de casos iniciales (aquí usamos un inicio de 100 casos, $N_0 = 100$) y $r$ es la tasa media de crecimiento.

El tiempo de duplicación se define como el tiempo en que el número de casos se duplica, es decir:

$$ 2N_0 = N_0 e^{rt} \ $$

de aquí se obtiene que

$$ e^{rt} = 2 \ $$

al tomar logaritmo natural de ambos lados, se obtiene:

$$ rt = log(2) \ $$

por lo tanto, el tiempo de duplicación vendría siendo

$$ t_{dupl} = log(2) / r \ $$

Usando estas ecuaciones podemos definir funciones para la duplicación de casos en dos y tres días, respectivamente:

``` r
dup2dias <- function(x)100*exp(log(2)/2*x)
dup3dias <- function(x)100*exp(log(2)/3*x)
```

En el gráfico a continuación, los datos de la evolución de casos en Latinoamérica son comparados con las curvas de crecimiento exponencial calculadas con las funciones anteriores. Las curvas artificiales son agregadas usando `stat_function`.

``` r
require(hrbrthemes)

## Obtener valores máximos y fecha
fecha2 <- format(max(dd$time), "%d/%m/%Y")
dd_today <- dd[dd$time == fecha2,]
dd_max_confirm <- max(dd$confirm)
dd_max_days <- max(dd$days_since_100)

# Crear etiquetas para las curvas de predicción
etiquetas_curvas <- tibble(
  etiq = c("Se duplica \ncada 2 días", "Se duplica \ncada 3 días"),
  x = c(dd_max_days - 4, dd_max_days),
  y = c(max(dd_today$confirm), dup3dias(dd_max_days)),
  country = c("xxx", "yyy")
)

p2 <- ggplot(dd, aes(days_since_100, confirm, color = country)) +
  geom_line(size = 1.5) +
  geom_point(data = dd_today, pch = 21, aes(size = cum_dead)) +
  stat_function(fun=dup2dias, geom="line", linetype=3, colour = "grey20") +
  stat_function(fun=dup3dias, geom="line", linetype=3, colour = "grey20") +
  scale_size(name = "Total \nfallecidos") +
  scale_x_continuous(expand = expansion(add = c(0,1))) +
  scale_y_continuous(limits = c(0, dd_max_confirm)) +
  theme_ipsum_rc(subtitle_family = "Roboto Condensed") +
  theme(
    panel.grid.minor = element_blank(),
    panel.grid.major.x = element_blank(),
    legend.position = "right",
    plot.margin = margin(3,4,3,3,"mm"),
  ) +
  guides(colour=FALSE) +
  coord_cartesian(clip = "off") +
  geom_text_repel(data = dd_today, aes(label = country), size = 5,
                  family = "Roboto Condensed", force = 1, bg.color = "white",
                  segment.color = "grey80") +
  geom_point(data = etiquetas_curvas, aes(x = x, y = y), colour = "#FFFFFF00") +
  geom_text_repel(data = etiquetas_curvas, aes(x = x, y = y, label = etiq),
                  colour = "grey20", size = 3, point.padding = 0,
                  family = "Roboto Condensed") +
  labs(x = "Número de días desde el caso 100", y = NULL,
       title = "COVID-19 en Latinoamérica",
       subtitle = paste0("Evolución de casos confirmados al ", fecha2))

print(p2)
```
{{< figure library="true" src="COVID19_casos_latam.png" title="Evolución de casos de COVID-19 en Latinoamérica comparados con curvas de duplicación de casos cada dos y tres días" lightbox="true" >}}

Es de esperarse que las medidas de confinamiento lleven a estas curvas a disminuir su velocidad de crecimiento y a estar muy por debajo de la curva de duplicación cada 3 días. Espero que eso ocurra pronto. Mientras tanto, evita el contacto social, lávate las manos, cuida de tu salud y sé solidario con tus vecinos.

{{< alert warning >}}
Esta no es una fuente oficial de información sobre la pandemia de COVID-19. Para consultas y referencias visite el [sitio oficial de la Organización Mundial de la Salud](https://www.who.int/es/emergencies/diseases/novel-coronavirus-2019).
{{< /alert >}}
