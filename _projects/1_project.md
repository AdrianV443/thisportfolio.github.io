---
layout: distill
title: Trabajando con la ENOE y Python
description: Descarga y configura los datos de la Encuesta Nacional de Ocupación y Empleo para realizar análisis estadísticos
importance: 1
category: Learn
related_publications: true
date: 2025-01-20
featured: true
toc:
  - name: Flujo de trabajo
  - name: Entendiendo las tablas y sus relaciones
    subsections:
      - name: Factor de expansión
      - name: Diseño de modelo multidimensional
  - name: Descarga de datos
  - name: Carga de Archivos en Python
  - name: Validación y verificación
    subsections:
      - name: Validación de las llaves primarias
      - name: Verificación de integridad referencial
      - name: Manejo de valores faltantes
  - name: Fusión de tablas 
    subsections:
      - name: Función de limpieza de tablas
      - name: Visualización de la tabla final
      - name: Guardado de la base de datos
  - name: Conclusión
toc_float: true
toc_depth: 3
giscus_comments: true
bibliography: my_bib.bib
---

<p style="text-align: justify;">
<b>La Encuesta Nacional de Ocupación y Empleo</b><d-cite key="inegi_enoe_wp"></d-cite> (ENOE) elaborada por el <b>Instituto Nacional de Estadística y Geografía</b> (INEGI) es una de las fuentes más importantes para comprender el mercado laboral mexicano. Proporciona datos mensuales y trimestrales con alcance nacional abarcando cada una de las 32 entidades federativas y un total de 39 ciudades. Este proyecto está inspirado en el Libro <i>¿Cómo empezar a estudiar el mercado de trabajo en México? Una introducción al análisis estadístico con R aplicado a la Encuesta Nacional de Ocupación y Empleo</i> <d-cite key="escotoanaruth_book"></d-cite> y tiene como objetivo demostrar cómo trabajar con la ENOE utilizando Python, desde la preparación de los datos hasta su análisis.
</p>

<p style="text-align: justify;">
Este escrito es el primero de tres en los que se trabaja con la ENOE: 
</p>

1. **Configuración de la base de datos**: Desde la descarga de los datos hasta su limpieza y preparación para el análisis.
2. **Análisis estadístico y visualización**: Realización de análisis básicos y visualizaciones utilizando la base de datos configurada.
3. **Análisis avanzado y predicciones**: Propuesta de análisis más complejos, incluyendo modelado predictivo e interpretación de resultados.

<p style="text-align: justify;">
A continuación, se detallan los pasos para obtener una <b>base de datos configurada</b> y lista para su análisis, comenzando por la extracción de los datos, su transformación y limpieza.
</p>

---
## Flujo de trabajo
El proceso para trabajar con la ENOE sigue un flujo estructurado que garantiza la integridad y consistencia de los datos:

1. **Descarga de datos**: Obtener la base de datos de la ENOE desde el sitio oficial del INEGI.
2. **Carga de archivos**: Leer los datos en Python utilizando bibliotecas especializadas.
3. **Validación y verificación**: Asegurar la integridad de los datos y la correcta relación entre las tablas.
4. **Fusión de tablas**: Integrar las tablas para crear una base de datos multidimensional.
5. **Limpieza y preparación**: Eliminar duplicados, manejar valores faltantes y asegurar la consistencia de los datos.
6. **Visualización y guardado**: Explorar los datos procesados y guardar la base final para su uso posterior.

---
## Entendiendo las tablas y sus relaciones
<p style="text-align: justify;">
La base de datos de la ENOE está conformada por <b>cinco tablas principales</b> , que están relacionadas entre si mediante campos comunes. Estas tablas son:
</p>

1. <b>Vivienda (VIV)</b>: Contiene información sobre las viviendas encuestadas.
2. <b>Hogares (HOG)</b>: Contiene datos sobre los hogares dentro de las viviendas.
3. <b>Sociodemográfico (SDEM)</b>: Contiene información sociodemográfica de los residentes de los hogares.
4. <b>Cuestionario de Ocupación y Empleo I (COE1)</b>: Contiene datos sobre ocupación y empleo (batería 1 a 5).
5. <b>Cuestionario de Ocupación y Empleo II (COE2)</b>: Contiene datos adicionales sobre ocupación y empleo (batería 6 en adelante).

<p style="text-align: justify;">
Las tablas se relacionan mediante llaves primarias compuestas por campos comunes, como:
</p>

- <b>CD_A</b>: Ciudad auto-representada.
- <b>ENT</b>: Entidad federativa.
- <b>CON</b>: Control.
- <b>V_SEL</b>: Vivienda seleccionada.
- <b>N_HOG</b>: Número de hogar.
- <b>H_MUD</b>: Hogar mudado.
- <b>N_REN</b>: Número de renglón (identificador único de residente).

<p style="text-align: justify;">
Estas relaciones permiten vincular las tablas para realizar análisis más complejos.
</p>
---

### Factor de expansión
<p style="text-align: justify;">
La tabla <b>SDEM</b> contiene un campo llamado <b>"FAC"</b>, que es el factor de expansión. Este campo es crucial para realizar estimaciones poblacionales, ya que indica cuántas personas representa cada registro en la muestra. Para realizar análisis multidimensionales, es importante incluir este campo en la tabla final, ya que permite escalar los datos a nivel nacional.<d-cite key="comosehacelaenoe"></d-cite>
</p>

### Diseño de modelo multidimensional
<p style="text-align: justify;">
Crear una base de datos multidimensional a partir de la ENOE implica organizar la información de manera eficiente. Para ello, es necesario entender las tablas, sus relaciones y los conceptos clave. En este contexto, las <b>dimensiones</b> son las categorías por las cuales se pueden analizar los datos, como tiempo, ubicación, género, nivel educativo, etc. Por otro lado, los <b>hechos</b> son los datos numéricos asociados a estas categorías, como ingresos, horas trabajadas, sector de actividad económica, etc.
</p>

<p style="text-align: justify;">
Es crucial integrar correctamente las tablas y optimizar el modelo para consultas y análisis rígidos y eficientes
</p>


## Descarga de datos
<p style="text-align: justify;">
Se ha seleccionado la encuesta del <b> cuarto trimestre del 2022</b>, disponible en formato <code>.dta</code> (utilizado por STATA). Este formato contiene un archivo de valores separado por tabuladores que puede incluir diferentes tipos de datos separados en filas y columnas.
Tras descargar y descomprimir los archivos se obtienen las siguientes cinco tablas:
</p>

- `ENOEN_COE1T422.dta`
- `ENOEN_COE2T422.dta`
- `ENOEN_HOGT422.dta`
- `ENOEN_VIVT422.dta`
- `ENOEN_SDEMT422.dta`

## Carga de archivos en Python
A continuación en un Jupyter Notebook se instala e importa la biblioteca pyreadstat con la siguiente línea de código:

```python
# Instalación de la librería pyreadstat
!pip install pyreadstat
# Importación de la librería
import pyreadstat
```
  <p style="text-align: justify;">
  Es importante que el archivo de Jupyter Notebook se encuentre en la misma carpeta que las tablas para evitar conflictos con las rutas de los archivos. Una vez instalada e importada la biblioteca, se procede a cargar las tablas y aplicar formatos legibles para facilitar su comprensión:
  </p>

```python
# Carga de tablas con formatos legibles
coet1422, meta1 = pyreadstat.read_dta('ENOEN_COE1T422.dta', apply_value_formats=True)
coet2422, meta2 = pyreadstat.read_dta('ENOEN_COE2T422.dta', apply_value_formats=True)
hogt422, meta3 = pyreadstat.read_dta('ENOEN_HOGT422.dta', apply_value_formats=True)
vivt422, meta4 = pyreadstat.read_dta('ENOEN_VIVT422.dta', apply_value_formats=True)
sdemt422, meta5 = pyreadstat.read_dta('ENOEN_SDEMT422.dta', apply_value_formats=True)
```

<p style="text-align: justify;">
El parámetro <code>apply_value_formats=True</code> convierte los códigos numéricos en nombres de categorías, lo que facilita la lectura de las tablas. En este proyecto, solo se utilizarán las variables con los nombres de cada tabla, ya que las variables "meta" contienen metadatos que no son relevantes para este análisis.
</p>
---

## Validación y verificación
<p style="text-align: justify;">
La etapa de la validación de datos es un paso que ayuda a mantener su integridad, mejorando la consistencia de los análisis posteriores y a reproducir el proceso. En esta parte se verifica que las llaves primarias sean consistentes en cada tabla.
</p>

- Primero se asignan los indicadores a las llaves primarias

```python
idvivienda = ['cd_a', 'ent', 'con', 'v_sel']
idhogar = ['cd_a', 'ent', 'con', 'v_sel', 'n_hog', 'h_mud']
idpersona = ['cd_a', 'ent', 'con', 'v_sel', 'n_hog', 'h_mud', 'n_ren']
```

### Validación de las llaves primarias
Validar la consistencia puede hacerse de la siguiente manera:
```python
# Verificar duplicados en las llaves primarias
print("Duplicados en VIVIENDA:", vivt422.duplicated(subset=idvivienda).sum())
print("Duplicados en HOGAR:", hogt422.duplicated(subset=idhogar).sum())
print("Duplicados en SDEM:", sdemt422.duplicated(subset=idpersona).sum())
print("Duplicados en COE1:", coet1422.duplicated(subset=idpersona).sum())
print("Duplicados en COE2:", coet2422.duplicated(subset=idpersona).sum())
```
`Output`
```
Duplicados en VIVIENDA: 0
Duplicados en HOGAR: 0
Duplicados en SDEM: 0
Duplicados en COE1: 0
Duplicados en COE2: 0
```
El código busca duplicados para cada identificador asociado a una tabla y los imprime según los encuentre. En este caso no se encontró ninguno

### Verificación de integridad referencial
<p style="text-align: justify;">
Asegura de que las relaciones entre tablas sean válidas. Por ejemplo, verifica que todas las variables de <code>idhogar</code> en la tabla <code>sdem</code> existan en la tabla <code>hog</code>, y que todos los <code>idvivienda</code> en <code>hog</code> existan en <code>viv</code>
</p>

```python
# Verificar integridad referencial entre HOG y VIV
hog_viv_check = hogt422[~hogt422[idvivienda].isin(vivt422[idvivienda].to_dict('list')).all(axis=1)]
print("Registros en HOG sin correspondencia en VIV:", len(hog_viv_check))

# Verificar integridad referencial entre SDEM y HOG
sdem_hog_check = sdemt422[~sdemt422[idhogar].isin(hogt422[idhogar].to_dict('list')).all(axis=1)]
print("Registros en SDEM sin correspondencia en HOG:", len(sdem_hog_check))
```
`Output`
```
Registros en HOG sin correspondencia en VIV: 0
Registros en SDEM sin correspondencia en HOG: 13758
```

Si encuentra registros sin correspondencia puede decidir si eliminarlos o investigar la causa, en este caso se omitieron

### Manejo de valores faltantes
Un paso más de la verificación es el manejo de valores faltantes de las llaves primarias, por ejemplo:
```python
# Verificar valores faltantes en las llaves primarias
print("Valores faltantes en VIVIENDA:", vivt422[idvivienda].isnull().sum())
print("Valores faltantes en HOGAR:", hogt422[idhogar].isnull().sum())
print("Valores faltantes en SDEM:", sdemt422[idpersona].isnull().sum())
print("Valores faltantes en COE1:", coet1422[idpersona].isnull().sum())
print("Valores faltantes en COE2:", coet2422[idpersona].isnull().sum())
```
`Output`
```
Valores faltantes en VIVIENDA: cd_a     0
ent      0
con      0
v_sel    0
dtype: int64
Valores faltantes en HOGAR: cd_a     0
ent      0
con      0
v_sel    0
n_hog    0
h_mud    0
dtype: int64
Valores faltantes en SDEM: cd_a     0
ent      0
con      0
v_sel    0
n_hog    0
h_mud    0
n_ren    0
dtype: int64
Valores faltantes en COE1: cd_a     0
ent      0
con      0
v_sel    0
n_hog    0
h_mud    0
n_ren    0
dtype: int64
Valores faltantes en COE2: cd_a     0
ent      0
con      0
v_sel    0
n_hog    0
h_mud    0
n_ren    0
dtype: int64
```
<p style="text-align: justify;">
La etapa de validación y verificación es importante para la construcción de la base de datos porque disminuye la perdidad de información e incrementa la confianza en los resultados
</p>

## Fusión de tablas
<p style="text-align: justify;">
La fusión de las tablas es un paso crucial para construir una base de datos multidimensional que permita realizar análisis complejos. Este proceso implica unir las tablas utilizando las llaves primarias compuestas por campos comunes.
</p>
<p style="text-align: justify;">
Realizar la fusión de las tablas es un proceso simple pero cada paso es de suma importancia para minimizar la perdida de información.
</p>
<p style="text-align: justify;">
El siguiente gráfico corresponde al modelo <b>entidad-relación</b> que nos proporciona el INEGI, en el se observan columnas que representan a cada tabla, a su vez, estas contienen identificadores y nombres de variables ubicados en la parte superior e inferior respectivamente, por ejemplo las tablas <b>COE2T</b> y <b>COE1T</b> tienen los mismos identificadores mencionados anteriormente. El modelo es la representación gráfica de las dimensiones de la encuesta.
</p>

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/modelo_er_enoen_2022_4t.jpg" title="modelo_entidad_relacion" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">Fuente: INEGI, <i>Encuesta  Nacional de Ocupación y Empleo.</i></div>


Los siguientes pasos nos permiten unir las tablas:

- Importa la librería `pandas`.
```python
import pandas as pd
```
- En seguida une por medio de `merge`
```python
# Unión con inner join para asegurar correspondencia
coet422 = pd.merge(coe1t422, coe2t422, on=idpersona, how='inner')
sdemcoet422 = pd.merge(sdemt422, coet422, on=idpersona, how='inner')
vivhogt422 = pd.merge(vivt422, hogt422, on=idvivienda, how='inner')
# Tabla resultante
completat422 = pd.merge(sdemcoet422, vivhogt422, on=idhogar, how='inner')
```
<p style="text-align: justify;">
En la primer linea de código la función <i><code>merge</code></i> une la tabla <code>coet1422</code> con <code>coet2422</code> utilizando los indicadores dentro de la variable <code>idpersona</code> como puntos de coincidencias para unir los datos y usa <code>inner</code> para unir todas las coincidencias. La tabla devuelta contiene los datos correspondientes pero también columnas "duplicadas" que son fácilmente diferenciables por los sufijos <code>_x</code> y <code>_y</code>.
</p>

<!-- 
Por ejemplo si queremos observar las coincidencias de las columnas <code>n_ent_x</code> y <code>n_ent_y</code> es posible creando una <i>crosstab</i> o tabla cruzada, a continuación el código para hacerlo:
```python
# Crosstab de pandas
tabla = pd.crosstab(coet422['n_ent_x'], coet422['n_ent_y'])
print(tabla)
```
``Output:``
<pre>
  <code class="plaintext">
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
  </code>
</pre> -->

<p style="text-align: justify;">
<!-- El resultado de la tabla cruzada muestra el número de veces en el que las categorías de una serie coinciden con las de la otra. Este paso es útil para observar como se manejaron los datos una vez fusionados.
<br> -->
<br>
Particularmente para este proyecto se considera una función para manejar la limpieza de las tablas eliminando aquellas columnas redundantes generadas a partir del paso anterior, sin embargo el uso de esta función es totalmente opcional
</p>

### Función de limpieza de tablas
```python
def limpiar_tabla(tabla):
    # Eliminar columnas duplicadas
    columnas_unicas = []
    for col in tabla.columns:
        if col.endswith('_x'):
            col_original = col[:-2]  # Eliminar '_x'
            if col_original not in columnas_unicas:
                columnas_unicas.append(col_original)
                tabla = tabla.rename(columns={col: col_original})
        elif col.endswith('_y'):
            tabla = tabla.drop(columns = col)  # Eliminar columnas con '_y'
        else:
            columnas_unicas.append(col)
    return tabla
```
Y se ejecuta sobre `completat422`.

```python
completat422 = limpiar_tabla(completat422)
print(completat422.shape)
```

Finalmente mostramos un fragmento de la tabla `completat422` que contiene toda la información necesaria para crear análisis estadísticos.

```python
# Función tail() para devolver las primeras "n" filas del DataFrame
completat422.head()
```
### Visualización de la tabla final
<div class="l-body-outset" style="overflow-x: auto; padding: 10px; margin-bottom: 20px;">
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }
    .dataframe tbody tr th {
        vertical-align: top;
    }
    /* .dataframe thead th {
        text-align: center;
        border-bottom: 2px transparent;
    } */
    .dataframe {
        width: 100%;
        border-collapse: collapse;
        margin: 0 auto;
    }
    .dataframe td, .dataframe th {
        padding: px;
        text-align: center;
        border: 2px solid #030000;
    }
    .dataframe tbody tr:nth-child(even) {
        background-color: #541515;
    }
    .dataframe tbody tr:hover {
        background-color: #541515;
    }
</style>
<table border="0" class="dataframe">
  <thead>
    <tr style="text-align: center;">
      <th></th>
      <th>r_def</th>
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
      <th>0</th>
      <td>Entrevista completa</td>
      <td>NaN</td>
      <td>39.0</td>
      <td>40.0</td>
      <td>188.0</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>Localidades mayores de 100000 habitantes</td>
      <td>NaN</td>
      <td>Guadalajara</td>
      <td>...</td>
      <td>Entrevista completa</td>
      <td>27.0</td>
      <td>9.0</td>
      <td>22.0</td>
      <td>Entrevista completa</td>
      <td>27.0</td>
      <td>9.0</td>
      <td>22.0</td>
      <td>No</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Entrevista completa</td>
      <td>NaN</td>
      <td>39.0</td>
      <td>40.0</td>
      <td>188.0</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>Localidades mayores de 100000 habitantes</td>
      <td>NaN</td>
      <td>Guadalajara</td>
      <td>...</td>
      <td>Entrevista completa</td>
      <td>27.0</td>
      <td>9.0</td>
      <td>22.0</td>
      <td>Entrevista completa</td>
      <td>27.0</td>
      <td>9.0</td>
      <td>22.0</td>
      <td>No</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Entrevista completa</td>
      <td>NaN</td>
      <td>39.0</td>
      <td>40.0</td>
      <td>188.0</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>Localidades mayores de 100000 habitantes</td>
      <td>NaN</td>
      <td>Guadalajara</td>
      <td>...</td>
      <td>Entrevista completa</td>
      <td>28.0</td>
      <td>9.0</td>
      <td>22.0</td>
      <td>Entrevista completa</td>
      <td>28.0</td>
      <td>9.0</td>
      <td>22.0</td>
      <td>No</td>
      <td>2.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Entrevista completa</td>
      <td>NaN</td>
      <td>39.0</td>
      <td>40.0</td>
      <td>188.0</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>Localidades mayores de 100000 habitantes</td>
      <td>NaN</td>
      <td>Guadalajara</td>
      <td>...</td>
      <td>Entrevista completa</td>
      <td>28.0</td>
      <td>9.0</td>
      <td>22.0</td>
      <td>Entrevista completa</td>
      <td>28.0</td>
      <td>9.0</td>
      <td>22.0</td>
      <td>No</td>
      <td>2.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Entrevista completa</td>
      <td>NaN</td>
      <td>39.0</td>
      <td>40.0</td>
      <td>188.0</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>Localidades mayores de 100000 habitantes</td>
      <td>NaN</td>
      <td>Guadalajara</td>
      <td>...</td>
      <td>Entrevista completa</td>
      <td>28.0</td>
      <td>9.0</td>
      <td>22.0</td>
      <td>Entrevista completa</td>
      <td>28.0</td>
      <td>9.0</td>
      <td>22.0</td>
      <td>No</td>
      <td>2.0</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 370 columns</p>
</div>

### Guardado de la base de datos
<p style="text-align: justify;">
Una vez finalizada la fusión y limpieza de las tablas, es importante guardar la base de datos para su uso posterior. En este caso, se guarda en formato <code>.csv</code>, un formato simple y ampliamente compatible:
</p>

```python
completat422.to_csv('completat422.csv', index=false)
```
## Conclusión
<p style="text-align: justify;">
El método presentado en este artículo proporciona una herramienta para el estudio del mercado laboral mexicano utilizando la Encuesta Nacional de Ocupación y Empleo (ENOE). A través de Python, es posible construir una base de datos multidimensional que permita realizar análisis estadísticos y visualizaciones avanzadas. Este enfoque no solo facilita el manejo de grandes volúmenes de datos, sino que también promueve el uso de herramientas de programación en el análisis de ciencias sociales.
</p>

<p style="text-align: justify;">
Se espera que este artículo sea de utilidad tanto para quienes se acercan por primera vez al análisis de datos con Python como para aquellos que buscan profundizar en el estudio del mercado laboral mexicano. Para ampliar este análisis, se pueden aplicar métodos estadísticos avanzados y visualizaciones utilizando bibliotecas como <code>matplotlib</code> o <code>seaborn</code>.
</p>

