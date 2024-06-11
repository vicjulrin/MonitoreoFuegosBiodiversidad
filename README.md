---
title: "Flujo de trabajo – Modelo de distribución de Vaccinium meridionale en el altiplano cundiboyacense, Colombia"
author: 
  - name: "Rincon-Parra VJ"
    email: "rincon-v@javeriana.edu.co"
  - name: "Sergio Rojas; Wilson Ramirez"
affiliation: "Instituto de Investigación de Recursos Biológicos Alexander von Humboldt - IAvH"
output: 
  md_document:
    variant: gfm
    preserve_yaml: true
    toc: true
---

- [1. Organizar directorio de trabajo](#organizar-directorio-de-trabajo)
- [2. Organizar entorno de trabajo](#organizar-entorno-de-trabajo)

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

## 1. Organizar directorio de trabajo

Las entradas de ejemplo de este ejercicio están almacenadas en
[https://drive.google.com/drive/home](https://github.com/vicjulrin/MonitoreoFuegosBiodiversidad/tree/main/script/input)
y deben organizarse en la carpeta raíz del código de la siguiente
manera:

- script -Script Reporte PuntosCalor 2024.R
  - input
    - Biomodelos2023 \# folder disponible para descarga en
      <http://geonetwork.humboldt.org.co/geonetwork/srv/spa/catalog.search#/metadata/0a1a6bdf-3231-4a77-8031-0dc3fa40f21b>
      - presente
        - sp_1.tif
        - sp_2.tif … …
        - sp_n.tif
    - BIOMA_IAvH.gpkg
    - IGAC_Depto.gpkg
    - PuntosCalor_nats_fire_nrt_1223_01_24.shp
    - redlist_ecosCol_Etter_2017.tif
    - TemplateReporte.docx
  - output

## 2. Organizar entorno de trabajo

ss
