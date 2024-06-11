---
title: "Flujo de trabajo – Análisis general de biodiversidad en puntos de calor/fuegos en Colombia"
author: 
  - name: "Rincon-Parra VJ"
    email: "rincon-v@javeriana.edu.co"
  - name: "Sergio Rojas; Wilson Ramirez"
affiliation: "Instituto de Investigación de Recursos Biológicos Alexander von Humboldt - IAvH"
output: 
  md_document:
    variant: gfm
    preserve_yaml: true
    toc: false
---

Este flujo de trabajo se desarrolló para monitorear la biodiversidad
potencial asociada a puntos de fuego en Colombia. Sin embargo, en el
momento que se generó la información de fuegos no estaba disponible, por
lo que se estimó a partir de datos de puntos de calor. Este flujo
permite la estimación de métricas de biodiversidad asociada a los puntos
que se dispongan desde un análisis respaldado en la información de
Biomodelos y ecosistemas amenazados, en relación con Biomas IAvH como
unidad ecológica de análisis.

El flujo de trabajo está desarrollado en R software y automatizado para
analizarse en cualquier ventana espacial de análisis (por ejemplo,
departamentos) y temporal de análisis (dependiendo de los puntos de
entrada). Este documento parte de un ejemplo con una ventana espacial de
departamento (Antioquia) para la explicación detallada de cada paso, y
finaliza con un loop de estimación para todos los departamentos de
Colombia. Las entradas deben organizarse en una carpeta raíz
[script/input](https://github.com/vicjulrin/MonitoreoFuegosBiodiversidad/tree/main/script/input),
y nombrarse explícitamente en la parte inicial del código. Las salidas
se almacenarán en la carpeta
[script/output](https://github.com/vicjulrin/MonitoreoFuegosBiodiversidad/tree/main/script/output)
sobre la misma raíz del código.

## Tabla de contenido

- [1. Organizar directorio de trabajo](#ID_seccion1)
- [2. Organizar entorno de trabajo](#ID_seccion2)

<a id="ID_seccion1"></a>

## 1. Organizar directorio de trabajo

Las entradas de ejemplo de este ejercicio están almacenadas en
[https://drive.google.com/drive/home](https://github.com/vicjulrin/MonitoreoFuegosBiodiversidad/tree/main/script/input).
Están organizadas de esta manera que facilita la ejecución del código:

    script
    │- Script Reporte PuntosCalor 2024.R
    │    
    └-input
    │ │
    │ └- Biomodelos2023 # Disponible en http://geonetwork.humboldt.org.co/geonetwork/srv/spa/catalog.search#/metadata/0a1a6bdf-3231-4a77-8031-0dc3fa40f21b
    │ │   └- presente
    │ │   │   │- sp_1.tif
    │ │   │   │- ...
    │ │   │   │- sp_n.tif
    │ │   │- Biomodelos_registros.csv      
    │ │   │- listas_spp_natgeo_sib_2023.csv      
    │ │
    │ │- BIOMA_IAvH.gpkg # Disponible en http://www.ideam.gov.co/web/ecosistemas/mapa-ecosistemas-continentales-costeros-marinos
    │ │- IGAC_Depto.gpkg # Disponible en https://geoportal.igac.gov.co/contenido/datos-abiertos-cartografia-y-geografia
    │ │- PuntosCalor_nats_fire_nrt_1223_01_24.shp # Puntos de interes para el analisis
    │ │- redlist_ecosCol_Etter_2017.tif # Disponible en http://reporte.humboldt.org.co/biodiversidad/2017/cap2/204/
    │ │-TemplateReporte.docx # Plantilla de reporte
    │     
    └--output

<a id="ID_seccion2"></a>

## 2. Organizar entorno de trabajo

### 2.1 Cargar librerias/paquetes necesarios para el análisis

``` r
## Cargar librerias necesarias para el análisis ####
# Lista de librerias necesarias
packages_list<-list("this.path", "magrittr", "dplyr", "plyr", "pbapply", "data.table", "raster", "terra", "sf", "ggplot2", "ggpubr", 
                    "tidyr", "RColorBrewer", "reshape2", "ggnewscale", "MASS", "stars",
                    "tidyterra", "fasterize", "lwgeom", "officer", "ggspatial",
                    "future", "future.apply", "progressr")

## Revisar e instalar librerias necesarias
packagesPrev<- .packages(all.available = TRUE)
lapply(packages_list, function(x) {   if ( ! x %in% packagesPrev ) { install.packages(x, force=T)}    })

## Cargar librerias
lapply(packages_list, library, character.only = TRUE)
```

### 2.2 Establecer directorios de trabajo

El flujo de trabajo está diseñado para establecer el entorno de trabajo
automáticamente a partir de la ubicación del código. Esto significa que
tomará como “dir_work” la carpeta raiz donde se almacena el código
“~/scripts. De esta manera, se garantiza que la ejecución se realice
bajo la misma organización descrita en el paso de [1.Organizar el
directorio de trabajo](#ID_seccion1))

``` r
# Establecer directorio
dir_work<- this.path::this.path() %>% dirname() # espacio de trabajo donde se almacena el codigo actual

# definir carpeta input
input_folder<- file.path(dir_work, "input"); # "~/input"

input<- list(Bioma_IAvH= file.path(input_folder, "Bioma_IAvH.gpkg"), # Biomas IAvH - Mapa IGAC Biomas 2017 
          redlist_ecos= file.path(input_folder,"redlist_ecosCol_Etter_2017.tif"), # Mapa de ecosistemas amenazados Etter http://reporte.humboldt.org.co/biodiversidad/2017/cap2/204/
          biomodelos_folder= file.path(input_folder, "Biomodelos2023/presente"),  # Datos biomodelos de modelos, registrod y metadatos Descargado de http://geonetwork.humboldt.org.co/geonetwork/srv/spa/catalog.search#/metadata/0a1a6bdf-3231-4a77-8031-0dc3fa40f21b
          biomodelos_metadata= file.path(input_folder,"Biomodelos2023/listas_spp_natgeo_sib_2023.csv"),
          Biomodelos_records= file.path(input_folder,"Biomodelos2023/Biomodelos_registros.csv"),
          template_reporte= file.path(input_folder,"TemplateReporte.docx"), 
          n_density= 350,
          PuntosCalor= file.path(input_folder,"PuntosCalor_nats_fire_nrt_1223_01_24.shp"),
          IGAC_Deptos= file.path(input_folder, "IGAC_Depto.gpkg")
          )

# crear carpeta output
output<- file.path(dir_work, "output"); dir.create(output)
```
