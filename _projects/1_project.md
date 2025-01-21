---
layout: page
title: Trabajando con la ENOE y Python
description: Descarga y configura los datos de la Encuesta Nacional de Ocupación y Empleo para realizar análisis estadísticos
# img: assets/img/enoe1_p.png
importance: 1
category: Learn
related_publications: true
toc:
    sidebar: left
giscus_comments: false
---
<p style="text-align: justify;">
<b>La Encuesta Nacional de Ocupación y Empleo</b> (ENOE) elaborada por el <b>Instituto Nacional de Estadística y Geografía</b> (INEGI) es una fuente clave para comprender el mercado laboral mexicano. Proporciona datos mensuales y trimestrales con alcance nacional abarcando cada una de las 32 entidades federativas y un total de 39 ciudades. Este proyecto está inspirado en el Libro <i>¿Cómo empezar a estudiar el mercado de 
trabajo en México? Una introducción al análisis estadístico con R aplicado a la Encuesta Nacional de Ocupación y Empleo</i> <b>{% cite escotoanaruth_book %}</b> y tiene fines demostrativos.
</p>
<p style="text-align: justify;">
El objetivo es mostrar el procesamiento de la ENOE para realizar análisis estadísticos descriptivos, asegurando la integridad de los datos para conseguir información valiosa. A continuación se detallan los pasos que van desde la extracción de la base de datos pasando por su transformación y limpieza hasta obtener una base configurada y lista para ser analizada.
</p>
---

## Flujo de trabajo

1. <b>Descarga de datos</b>: Obtener la base de datos de INEGI
2. <b>Carga de archivos</b>: Leer los datos en Python utilizando bibliotecas adecuadas
3. <b>Fusión de tablas</b>: Integrar las tablas para crear una base multidimensional
4. <b>Limpieza y preparación</b>: Asegurar la integridad y consistencia de los datos
5. <b>Visualización</b>: Explorar el contenido procesado

---

## Descarga de datos
<p style="text-align: justify;">
Para este proyecto, seleccionamos la base del <b> cuarto trimestre del 2022</b>. en formato <code>.dta</code> (especialmente usado por STATA). Este formato contiene un 
archivo de valores separado por tabuladores que puede incluir diferentes tipos de datos separados en filas y columnas.
</p>
Tras descargar y descomprimir los archivos se muestran cinco tablas:

- `ENOEN_COE1T422.dta`
- `ENOEN_COE2T422.dta`
- `ENOEN_HOGT422.dta`
- `ENOEN_VIVT422.dta`
- `ENOEN_SDEMT422.dta`

## Carga de archivos en Python
A continuación en un Jupyter Notebook instalaremos pyreadstat con la siguiente línea de código:
```python
# Librería que permite manejar archivos STATA en python
!pip install pyreadstat
```
Es importante que el Notebook se encuentre dentro de la misma carpeta que las tablas con la finalidad de evitar conflictos con las rutas de los archivos.

Una vez instalada la importamos a nuestro entorno:
```python
import pyreadstat
```

Cargamos las tablas y aplicamos formatos legibles para facilitar su comprensión:
```python
# Cuestionario de ocupación y empleo 1
coet1422, meta1 = pyreadstat.read_dta('ENOEN_COE1T422.dta', apply_value_formats=True)
# Cuestionario de ocupación y empleo 2
coet2422, meta2 = pyreadstat.read_dta('ENOEN_COE2T422.dta', apply_value_formats=True)
# Tabla Hogares
hogt422, meta3 = pyreadstat.read_dta('ENOEN_HOGT422.dta', apply_value_formats=True)
# Tabla Vivienda
vivt422, meta4 = pyreadstat.read_dta('ENOEN_VIVT422.dta', apply_value_formats=True)
# Tabla Sociodemografico
sdemt422, meta5 = pyreadstat.read_dta('ENOEN_SDEMT422.dta', apply_value_formats=True)
```
<p style="text-align: justify;">
Lo que hace <code>pyreadstat.read_dta</code> con el parámetro <code>apply_value_formats=True</code> es configurar los datos dentro de las tablas mostrándolos como nombres de 
categorías en lugar de un número entero facilitando así la lectura de la tabla.
Solo se usarán las variables con los nombres de cada tabla ya que todas las variables con el nombre "meta" solo contienen metadatos que por esta ocasión 
no se toman en cuenta.
</p>
---

## Fusión de tablas
<p style="text-align: justify;">
La fusión es el paso más importante porque nos ayuda a comprender como se construye la base de datos con la que podemos obtener indicadores clave, es esta parte la que muestra la multidimensionalidad sobre la que se trabaja.
</p>
<p style="text-align: justify;">
Realizar la fusión de las tablas es un proceso simple pero repetitivo y cada paso es de suma importancia para minimizar la perdida de información.
</p>
<p style="text-align: justify;">
El siguiente gráfico corresponde al modelo <b>entidad-relación</b> que nos proporciona el INEGI, en se observan columnas que representan a cada tabla, a su vez, estas contienen identificadores y nombres de variables ubicados en la parte superior e inferior respectivamente, por ejemplo las tablas <b>COE2T</b> y <b>COE1T</b> tienen los mismos cinco identificadores. El modelo es la representación gráfica de la interrelación existente entre estas tablas
</p>

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/modelo_er_enoen_2022_4t.jpg" title="modelo_entidad_relacion" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Fuente INEGI.
</div>

Se asignan variables que contendrán los identificadores de cada tabla:
```python
idvivienda = ['cd_a', 'ent', 'con', 'v_sel']
idhogar = ['cd_a', 'ent', 'con', 'v_sel', 'n_hog', 'h_mud']
# Se toma en cuenta "cd_a" o "ciudad autorrepresentada".
idpersona = ['cd_a', 'ent', 'con', 'v_sel', 'n_hog', 'h_mud', 'n_ren']
```

Los siguientes pasos requieren que usemos la librería pandas.
```python
import pandas as pd
```

El siguiente código nos permite unir las primeras dos tablas:
```python
coet422 = pd.merge(coet1422, coet2422, on=idpersona)
coet422.columns
```
La función *merge* está uniendo la tabla coet1422 con coet2422 utilizando los indicadores dentro de la variable `idpersona` como puntos de coincidencias para unir los datos. La tabla que nos devuelve contiene columnas "duplicadas" pero fácilmente diferenciables por los sufijos ``_x`` y ``_y``.

Por ejemplo si queremos observar las coincidencias de las columnas `n_ent_x` y `n_ent_y` es posible creando una *crosstab* o tabla cruzada, a continuación el código para hacerlo:
```python
# Crosstab de pandas
tabla = pd.crosstab(coet422['n_ent_x'], coet422['n_ent_y'])
print(tabla)
```
``Output:``
```
n_ent_y             Cuarta entrevista  Primera entrevista  Quinta entrevista  \
n_ent_x                                                                        
Cuarta entrevista               64109                   0                  0   
Primera entrevista                  0               65103                  0   
Quinta entrevista                   0                   0              63929   
Segunda entrevista                  0                   0                  0   
Tercera entrevista                  0                   0                  0   

n_ent_y             Segunda entrevista  Tercera entrevista  
n_ent_x                                                     
Cuarta entrevista                    0                   0  
Primera entrevista                   0                   0  
Quinta entrevista                    0                   0  
Segunda entrevista               64334                   0  
Tercera entrevista                   0               65395  
```
<p style="text-align: justify;">

El resultado de la tabla cruzada muestra el número de veces en el que las categorías de una serie coinciden con las de la otra. Este paso es útil para observar como se manejaron los datos una vez fusionados.
<br>
Particularmente para este proyecto se considera una función para manejar la limpieza de las tablas eliminando aquellas columnas redundantes generadas a partir del paso anterior, sin embargo es totalmente opcional
</p>

```python
# Función para limpiar tablas
def limpiar_tabla(tabla):
    columnas_eliminar = [col for col in tabla.columns if col.endswith('_y')]
    tabla = tabla.drop(columns = columnas_eliminar)
    tabla = tabla.rename(columns={col:col .split('_')[0] for col in tabla.columns if col.endswith('_x')})
    return tabla
```

Y se ejecuta sobre la primera tabla creada.
```python
coet422 = limpuar_tabla(coet422)
print(coet422.shape)
```
`Output:`
```
(322870, 235)
```

Se repite el proceso uniendo todas las tablas.
```python
# Fusión y limpieza 
sdemcoet422 = pd.merge(sdemt422, coet422, on=idpersona)
sdemcoet422 = limpiar_tabla(sdemcoet422)
print(f'sdemcoet422: {sdemcoet422.shape}')
# Fusion y limpieza
vivhogt422 = pd.merge(vivt422, hogt422, on=idvivienda)
vivhogt422 = limpiar_tabla(vivhogt422)
print(f'vivhogt422: {vivhogt422.shape}')
# Limpieza
completat422 = pd.merge(sdemcoet422, vivhogt422, on=idhogar)
completat422 = limpiar_tabla(completat422)
print(f'completat422: {completat422.shape}')
```

Finalmente mostramos un fragmento de la tabla `completat422` que contiene toda la información necesaria para crear análisis estadísticos.

```python
# Función tail() para devolver las últimas "n" filas del DataFrame
completat422.tail()
```
<!-- <div> -->
<div class="fake-img l-page-outset">
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>r</th>
      <th>loc</th>
      <th>mun</th>
      <th>est</th>
      <th>est_d_tri</th>
      <th>est_d_men</th>
      <th>ageb</th>
      <th>t_loc_tri</th>
      <th>t_loc_men</th>
      <th>cd_a</th>
      <th>...</th>
      <th>r_pre</th>
      <th>p_dia</th>
      <th>p_mes</th>
      <th>p_anio</th>
      <th>r_def</th>
      <th>d_dia</th>
      <th>d_mes</th>
      <th>d_anio</th>
      <th>e_obs</th>
      <th>inf</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>311078</th>
      <td>Entrevista completa</td>
      <td>NaN</td>
      <td>2.0</td>
      <td>20.0</td>
      <td>476.0</td>
      <td>420.0</td>
      <td>0.0</td>
      <td>Localidades menores de 2 500 habitantes</td>
      <td>Localidades menores de 2 500 habitantes</td>
      <td>Complemento urbano-rural</td>
      <td>...</td>
      <td>Entrevista completa</td>
      <td>6.0</td>
      <td>12.0</td>
      <td>22.0</td>
      <td>Entrevista completa</td>
      <td>6.0</td>
      <td>12.0</td>
      <td>22.0</td>
      <td>No</td>
      <td>2.0</td>
    </tr>
    <tr>
      <th>311079</th>
      <td>Entrevista completa</td>
      <td>NaN</td>
      <td>2.0</td>
      <td>20.0</td>
      <td>476.0</td>
      <td>420.0</td>
      <td>0.0</td>
      <td>Localidades menores de 2 500 habitantes</td>
      <td>Localidades menores de 2 500 habitantes</td>
      <td>Complemento urbano-rural</td>
      <td>...</td>
      <td>Entrevista completa</td>
      <td>6.0</td>
      <td>12.0</td>
      <td>22.0</td>
      <td>Entrevista completa</td>
      <td>6.0</td>
      <td>12.0</td>
      <td>22.0</td>
      <td>No</td>
      <td>2.0</td>
    </tr>
    <tr>
      <th>311080</th>
      <td>Entrevista completa</td>
      <td>NaN</td>
      <td>2.0</td>
      <td>20.0</td>
      <td>476.0</td>
      <td>420.0</td>
      <td>0.0</td>
      <td>Localidades menores de 2 500 habitantes</td>
      <td>Localidades menores de 2 500 habitantes</td>
      <td>Complemento urbano-rural</td>
      <td>...</td>
      <td>Entrevista completa</td>
      <td>6.0</td>
      <td>12.0</td>
      <td>22.0</td>
      <td>Entrevista completa</td>
      <td>6.0</td>
      <td>12.0</td>
      <td>22.0</td>
      <td>No</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>311081</th>
      <td>Entrevista completa</td>
      <td>NaN</td>
      <td>2.0</td>
      <td>20.0</td>
      <td>476.0</td>
      <td>420.0</td>
      <td>0.0</td>
      <td>Localidades menores de 2 500 habitantes</td>
      <td>Localidades menores de 2 500 habitantes</td>
      <td>Complemento urbano-rural</td>
      <td>...</td>
      <td>Entrevista completa</td>
      <td>6.0</td>
      <td>12.0</td>
      <td>22.0</td>
      <td>Entrevista completa</td>
      <td>6.0</td>
      <td>12.0</td>
      <td>22.0</td>
      <td>No</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>311082</th>
      <td>Entrevista completa</td>
      <td>NaN</td>
      <td>2.0</td>
      <td>20.0</td>
      <td>476.0</td>
      <td>420.0</td>
      <td>0.0</td>
      <td>Localidades menores de 2 500 habitantes</td>
      <td>Localidades menores de 2 500 habitantes</td>
      <td>Complemento urbano-rural</td>
      <td>...</td>
      <td>Entrevista completa</td>
      <td>6.0</td>
      <td>12.0</td>
      <td>22.0</td>
      <td>Entrevista completa</td>
      <td>6.0</td>
      <td>12.0</td>
      <td>22.0</td>
      <td>No</td>
      <td>99.0</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 349 columns</p>
</div>
---


## Conclusión
<p style="text-align: justify;">

El método presentado aquí pretende ser una herramienta más para el acercamiento al estudio de las ciencias sociales a partir de los datos de la Encuesta Nacional de Ocupación y Empleo (ENOE). Es posible ampliar este análisis aplicando métodos estadísticos o visualizaciones con bibliotecas como <code>matplotlib</code> o <code>seaborn</code>. 
</p>

<p style="text-align: justify;">

Se espera acercar al lector al estudio de Python como herramienta para construir estructuras de datos, también para quien ya tiene conocimiento sobre el lenguaje de programación y les sea de interés analizar el mercado nacional de trabajo. Si bien no se dan descripciones básicas técnicas se espera promover su uso y aplicación.
</p>

