---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Algunos trucos útiles al crear mapas interactivos de leaflet en R"
subtitle: ""
summary: "Recopilación de trucos sencillos que pueden ser útiles al empezar a usar leaflet para crear mapas interactivos en R, agregar datos a los mismos y publicarlos en la web. Se incluyen instrucciones para agregar estos mapas a sitios webs estáticos hechos con hugo."
authors: [admin]
tags: ["r", "mapas", "leaflet", "covid-19", "hugo"]
categories: []
date: 2020-04-25T16:17:49+02:00
lastmod: 2020-04-25T16:17:49+02:00
featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: ""
  focal_point: ""
  preview_only: false

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---
*Leaflet* es una biblioteca de  {{< icon name="js" pack="fab" >}}Javascript para hacer mapas y existe un paquete homónimo para {{< icon name="r-project" pack="fab" >}} que permite usar todo el potencial de *leaflet* en nuestros mapas y hacerlos interactivos y aptos para móviles.

Para crear un mapa en blanco se usa la función `leaflet`. La función `addTiles()` agrega "tiles" de OpenStreetMap.

``` r
library(leaflet)

m0 <- leaflet() %>%
  addTiles() %>%
  setView(lng=-75, lat=-12 , zoom=4) # Centrar el mapa en Suramérica

m0 # Visualizar el mapa
```

{{< hp5 "/leaflet/leaflet_m0.html" "600">}}

## Arreglar el "zoom-out" {{< icon name="search-minus" pack="fas" >}}

Uno de los problemas que más pueden fastidiar al crear un mapa de *leaflet* es que al hacer *zoom-out*{{< icon name="search-minus" pack="fas" >}} se pueden llegar a ver "varios" planetas Tierra, lo cual no tiene mucho sentido. Esto se puede arreglar definiendo límites al hacer *zoom-out*{{< icon name="search-minus" pack="fas" >}} y *zoom-in*{{< icon name="search-plus" pack="fas" >}}. Se pueden cambiar las opciones de la función `leaflet`. Un ejemplo es el siguiente:

``` r
library(leaflet)

m0 <- leaflet(options = leafletOptions(minZoom = 3, maxZoom = 8)) %>%
  addTiles() %>%
  setView(lng=-75, lat=-12 , zoom=4) # Centrar el mapa en Suramérica

m0
```
{{< hp5 "/leaflet/leaflet_m0_zoom.html" "600">}}

En este caso, iniciamos con un zoom a nivel 4, definido en `setView`, y sólo permitimos bajar un nivel más (3) y llegar a un máximo de 8. Estos límites serán mejor definidos al crear y visualizar el mapa y dependerán de la aplicación que se le quiera dar. Para dejar el mapa en un zoom fijo y no permitir *zoom-out* ni *zoom-in*, se pueden fijar `minZoom` y `maxZoom` al mismo nivel.

## Cambiar el estilo del mapa {{< icon name="brush" pack="fas" >}}

Además de las capas de OpenStreetMap usadas por defecto en *leaflet*, existen muchas otras disponibles. Una lista de diferentes estilos/capas se puede encontrar [aquí](http://leaflet-extras.github.io/leaflet-providers/preview/index.html). Uno de mis temas favoritos es *Positron* de [**CartoDB**](https://github.com/CartoDB/basemap-styles). Este mapa se puede cargar de la siguiente manera:

``` r
m0_positron <- leaflet(options = leafletOptions(minZoom = 3, maxZoom = 8)) %>%
  addProviderTiles(providers$CartoDB.Positron) %>%
  setView(lng=-75, lat=-12 , zoom=4)

m0_positron
```
{{< hp5 "/leaflet/leaflet_m0_positron.html" "300">}}

También se puede cargar sin las etiquetas usando `addProviderTiles(providers$CartoDB.PositronNoLabels)`.

{{< hp5 "/leaflet/leaflet_m0_sin_etiq.html" "300">}}

**CartoDB** también provee un tema oscuro que puede ser usado de igual forma con o sin etiquetas: `addProviderTiles(providers$CartoDB.DarkMatter)` o `addProviderTiles(providers$CartoDB.DarkMatterNoLabels)`, respectivamente.

Acá una visualización del tema oscuro sin etiquetas:

{{< hp5 "/leaflet/leaflet_m0_oscuro.html" "300">}}

## Agregar datos al mapa {{< icon name="database" pack="fas" >}}

Para este ejemplo vamos a agregar datos del paquete [`{coronavirus}`](https://github.com/RamiKrispin/coronavirus). Más detalles sobre el uso de estos datos se pueden conseguir en esta [otra entrada]({{< ref "/post/coronavirus-r/index.md" >}}). Para cargar estos datos hacemos lo siguiente:

``` r
## Cargar datos
datacov <- coronavirus %>%
  # Seleccionar datos de interés
  select(country = Country.Region, type, cases, Lat, Long) %>%
  # Agrupar por país
  group_by(country, Lat, Long, type) %>%
  # Totalizar los casos
  summarise(total_cases = sum(cases)) %>%
  # Crear columnas para cada tipo de caso
  pivot_wider(names_from = type, values_from = total_cases) %>%
  # Ordenar por número de casos confirmados
  arrange(-confirmed) %>%
  ungroup()

# Cambios los nombres a español
datacov <- datacov %>%
  mutate(country = ifelse(country == "Peru", "Perú", country)) %>%
  mutate(country = ifelse(country == "Dominican Republic", "República Dominicana", country)) %>%
  mutate(country = ifelse(country == "Brazil", "Brasil", country)) %>%
  mutate(country = ifelse(country == "Haiti", "Haití", country)) %>%
  mutate(country = ifelse(country == "Panama", "Panamá", country)) %>%
  mutate(country = ifelse(country == "Mexico", "México", country))
```

Usando la función `addCircleMarkers()`, se puede graficar la variable *confirmed* (número de casos confirmados) sobre el mapa, usando el logaritmo del número de casos para definir el radio de los círculos graficados. Latitud y longitud están incluidos en los datos en las variables `Long` y `Lat`. Usando el parámetro `popup` se puede crear una caja de información con datos a ser mostrados al hace click en cada círculo. En este caso se agrega el nombre del país y número de casos confirmados y fallecidos.

``` r
m1 <- leaflet(datacov, options = leafletOptions(minZoom = 3, maxZoom = 8)) %>%
  addProviderTiles(providers$CartoDB.PositronNoLabels) %>%
  addCircleMarkers(
    lng = ~Long,
    lat = ~Lat,
    weight = 3,
    radius = ~log(confirmed)*3, # Adaptar según se desee
    color = "#ff7761",
    stroke = FALSE, fillOpacity = 0.9,
    popup = paste(datacov$country, "<br>",
                  "<b>Casos confirmados: </b>", datacov$confirmed, "<br>",
                  "<b>Fallecidos: </b>", datacov$death)
  ) %>%
setView(lng=-75, lat=-12 , zoom=4)

m1
```
{{< hp5 "/leaflet/leaflet_m1.html" "600">}}

En el siguiente ejemplo se incluye una nueva capa con el número de fallecidos, la cual estará debajo de la de número de casos confirmados para garantizar que el área cliqueable siga activa en el círculo más grande.

``` r
m2 <- leaflet(datacov, options = leafletOptions(minZoom = 3, maxZoom = 8)) %>%
  addProviderTiles(providers$CartoDB.PositronNoLabels) %>%
  addCircleMarkers(
    lng = ~Long,
    lat = ~Lat,
    weight = 3,
    radius = ~log(death),
    color = "#881600",
    stroke = FALSE, fillOpacity = 0.9,
  ) %>%
  addCircleMarkers(
    lng = ~Long,
    lat = ~Lat,
    weight = 3,
    radius = ~log(confirmed)*3,
    color = "#ff7761",
    stroke = FALSE, fillOpacity = 0.7,
    popup = paste(datacov$country, "<br>",
                  "<b>Casos confirmados: </b>", datacov$confirmed, "<br>",
                  "<b>Fallecidos: </b>", datacov$death)
  ) %>%
  setView(lng=-75, lat=-12 , zoom=4)

m2
```
{{< hp5 "/leaflet/leaflet_m2.html" "600">}}

## Mostrar etiquetas al desplazar el cursor {{< icon name="mouse-pointer" pack="fas" >}}

Para mostrar información al desplazar el cursor sobre los círculos, se puede usar el argumento `label` dentro de la función `addCircleMarkers()`. Una etiqueta con información del país y la cantidad de casos confirmados se agrega usando el siguiente parámetro:

```r
label = paste0(as.character(datacov$country)," | Casos confirmados: ", as.character(datacov$confirmed)),
```

### Personalizar las etiquetas
Estas etiquetas pueden ser personalizadas usando el argumento `labelOptions`. En el siguiente mapa se agregan las etiquetas anteriores y se hacen algunos cambios a los colores y la tipografía de las mismas:

```r
m3 <- leaflet(datacov, options = leafletOptions(minZoom = 3, maxZoom = 8)) %>%
  addTiles() %>%
  addProviderTiles(providers$CartoDB.PositronNoLabels) %>%
  addCircleMarkers(
    lng = ~Long,
    lat = ~Lat,
    weight = 3,
    radius = ~log(death),
    color = "#881600",
    stroke = FALSE, fillOpacity = 0.9,
  ) %>%
  addCircleMarkers(
    lng = ~Long,
    lat = ~Lat,
    weight = 3,
    radius = ~log(confirmed)*3,
    color = "#ff7761",
    stroke = FALSE, fillOpacity = 0.7,
    popup = paste(datacov$country, "<br>",
                  "<b>Casos confirmados: </b>", datacov$confirmed, "<br>",
                  "<b>Fallecidos: </b>", datacov$death),
    label = paste0(as.character(datacov$country)," | Casos confirmados: ", as.character(datacov$confirmed)),
    labelOptions = labelOptions(style = list(
                                "color" = "#FFFFF3",
                                "font-family" = "Roboto",
                                "font-weight" = "bold",
                                "box-shadow" = "3px 3px rgba(0,0,0,0.25)",
                                "font-size" = "16px",
                                "background-color" = "#566270",
                                "border-color" = "rgba(0,0,0,0.5)"))
  ) %>%
  setView(lng=-75, lat=-12 , zoom=4)

m3
```
{{< hp5 "/leaflet/leaflet_m3.html" "600">}}

### Personalizar las cajas *popup*
La información mostrada en las cajas *popup* que aparecen al hacer click también puede ser personalizada. Sin embargo, el paquete `{leaflet}` es bastante limitado en este aspecto. Una forma de modificar el estilo a gusto es usando la función `browsable()` del paquete `{htmltools}`. Un ejemplo usando el mapa generado anteriormente es el siguiente:

```r
browsable(
  tagList(list(
    tags$head(
      tags$style(
        ".leaflet-popup-content-wrapper {
        background: black;
        color: #ffffff;
        font-size: 15px;
        padding: 2px;
        border-radius: 0px;
        }"
      )
    ),
    m3
  ))
)
```

## Guardar y publicar el mapa {{< icon name="save" pack="fas" >}}

El resultado se puede guardar como un archivo html usando el paquete `{htmlwidgets}`.

```r
library(htmlwidgets)
saveWidget(m3, "~/leaflet_m3.html")
```

Este archivo html se puede subir e incrustar en un sitio web usando el siguiente código:

```html
<iframe src="leaflet_m0.html" frameborder="0" width="100%" height="600px"></iframe>
```

### Publicar en un sitio web Hugo

Para publicar en un sitio estático de hugo se puede agregar un nuevo *shortcode* en `/layouts/partials/shortcodes`. Esto se hace creando un nuevo archivo en ese directorio, al cual, en este caso, llamaremos `hp5.html`. El contenido de dicho archivo sería el siguiente:

```html
<iframe src="{{.Get 0}}" width="100%" height="{{.Get 1}}" frameborder="0" allowfullscreen="allowfullscreen" allow="geolocation *; microphone *; camera *; midi *; encrypted-media *"></iframe>
```

Aquí se define la forma en que se mostrará el mapa. En este caso, con un ancho de "100%" y una altura variable en píxeles. El *shortcode* hp5, tal como está definido, requerirá dos parámetros: 1. localización del archivo y 2. altura (*height*) del mapa a ser mostrado.

Una vez definido el *shortcode*, podemos usarlo en cualquier publicación de hugo para mostrar el mapa creado y mostrarlo con un altura de `"600"` píxeles. Como se muestra en este ejemplo:

```markdown
{{</* hp5 "/leaflet/leaflet_m0.html" "600"*/>}}
```

¡Feliz mapeo!
