<!-- ---
layout: page
title: Trabajando con la ENOE y Python
description: Descarga y configura los datos de la Encuesta Nacional de Ocupación y Empleo para realizar análisis estadísticos
importance: 1
category: Learn
related_publications: true
date: 2025-01-20
featured: true
giscus_comments: true
bibliography: my_bib.bib
---

<p style="text-align: justify;">
La <b>Encuesta Nacional de Ocupación y Empleo</b> (ENOE), elaborada por el <b>Instituto Nacional de Estadística y Geografía</b> (INEGI)<d-cite key="inegi_enoe_wp"></d-cite>, es una de las fuentes más importantes para comprender el mercado laboral en México. Proporciona datos mensuales y trimestrales con alcance nacional, cubriendo las 32 entidades federativas y 39 ciudades. Este proyecto está inspirado en el libro <i>¿Cómo empezar a estudiar el mercado de trabajo en México? Una introducción al análisis estadístico con R aplicado a la Encuesta Nacional de Ocupación y Empleo</i><d-cite key="escotoanaruth_book"></d-cite>, y tiene como objetivo demostrar cómo trabajar con la ENOE utilizando Python, desde la preparación de los datos hasta su análisis.
</p>

<p style="text-align: justify;">
Este artículo es el primero de tres en los que se trabaja con la ENOE: 
</p>

1. **Configuración de la base de datos**: Desde la descarga de los datos hasta su limpieza y preparación para el análisis.
2. **Análisis estadístico y visualización**: Realización de análisis básicos y visualizaciones utilizando la base de datos configurada.
3. **Análisis avanzado y predicciones**: Propuesta de análisis más complejos, incluyendo modelado predictivo e interpretación de resultados.

A continuación, se detallan los pasos para obtener una **base de datos configurada** y lista para su análisis, comenzando por la extracción de los datos, su transformación y limpieza.

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
La base de datos de la ENOE está conformada por <b>cinco tablas principales</b>, relacionadas entre sí mediante campos comunes. Estas tablas son:
</p>

1. **Vivienda (VIV)**: Contiene información sobre las viviendas encuestadas, como el tipo de vivienda y su ubicación geográfica.
2. **Hogares (HOG)**: Almacena datos sobre los hogares dentro de las viviendas, como el número de habitaciones y el número de residentes.
3. **Sociodemográfico (SDEM)**: Incluye información sociodemográfica de los residentes, como edad, género, nivel educativo y condición laboral.
4. **Cuestionario de Ocupación y Empleo I (COE1)**: Contiene datos sobre ocupación y empleo (batería 1 a 5), como horas trabajadas y sector de actividad económica.
5. **Cuestionario de Ocupación y Empleo II (COE2)**: Complementa la información de COE1 con datos adicionales sobre ocupación y empleo (batería 6 en adelante).

<p style="text-align: justify;">
Las tablas se relacionan mediante <b>llaves primarias compuestas</b> por campos comunes, como:
</p>

- **CD_A**: Ciudad auto-representada.
- **ENT**: Entidad federativa.
- **CON**: Control.
- **V_SEL**: Vivienda seleccionada.
- **N_HOG**: Número de hogar.
- **H_MUD**: Hogar mudado.
- **N_REN**: Número de renglón (identificador único de residente).

<p style="text-align: justify;">
Estas relaciones permiten vincular las tablas para realizar análisis más complejos y multidimensionales.
</p>

### Factor de expansión
<p style="text-align: justify;">
La tabla <b>SDEM</b> contiene un campo llamado <b>"FAC"</b>, que es el <b>factor de expansión</b>. Este campo es crucial para realizar estimaciones poblacionales, ya que indica cuántas personas representa cada registro en la muestra. Para realizar análisis multidimensionales, es importante incluir este campo en la tabla final, ya que permite escalar los datos a nivel nacional.
</p>

### Diseño de un modelo multidimensional
<p style="text-align: justify;">
Crear una base de datos multidimensional a partir de la ENOE implica organizar la información de manera eficiente. Para ello, es necesario entender las tablas, sus relaciones y los conceptos clave. En este contexto, las <b>dimensiones</b> son las categorías por las cuales se pueden analizar los datos, como tiempo, ubicación, género, nivel educativo, etc. Por otro lado, los <b>hechos</b> son los datos numéricos asociados a estas categorías, como ingresos, horas trabajadas, sector de actividad económica, etc.
</p>

<p style="text-align: justify;">
Es crucial integrar correctamente las tablas y optimizar el modelo para consultas y análisis rápidos y eficientes.
</p>

---

## Descarga de datos
<p style="text-align: justify;">
Para este proyecto, se ha seleccionado la encuesta del <b>cuarto trimestre de 2022</b>, disponible en formato <code>.dta</code> (utilizado por STATA). Este formato contiene un archivo de valores separados por tabuladores que puede incluir diferentes tipos de datos organizados en filas y columnas. Tras descargar y descomprimir los archivos, se obtienen las siguientes cinco tablas:
</p>

- `ENOEN_COE1T422.dta`
- `ENOEN_COE2T422.dta`
- `ENOEN_HOGT422.dta`
- `ENOEN_VIVT422.dta`
- `ENOEN_SDEMT422.dta`

---

## Carga de archivos en Python
<p style="text-align: justify;">
Para trabajar con los archivos <code>.dta</code> en Python, se utiliza la biblioteca <b>pyreadstat</b>, que permite leer y manipular este tipo de archivos. A continuación, se muestra cómo instalar e importar la biblioteca:
</p>

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
# Carga de las tablas con formatos legibles
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

## Fusión de tablas
<p style="text-align: justify;">
La fusión de las tablas es un paso crucial para construir una base de datos multidimensional que permita realizar análisis complejos. Este proceso implica unir las tablas utilizando las llaves primarias compuestas por campos comunes. A continuación, se muestra cómo realizar la fusión paso a paso:
</p>

### Modelo entidad-relación
<p style="text-align: justify;">
El siguiente gráfico corresponde al <b>modelo entidad-relación</b> proporcionado por el INEGI. En él se observan las relaciones entre las tablas, identificadas por columnas que representan a cada tabla y sus variables. Por ejemplo, las tablas <b>COE1T</b> y <b>COE2T</b> comparten los mismos cinco identificadores, lo que permite su unión.
</p>

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/modelo_er_enoen_2022_4t.jpg" title="modelo_entidad_relacion" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">Fuente: INEGI, <i>Encuesta Nacional de Ocupación y Empleo.</i></div>

### Asignación de identificadores
<p style="text-align: justify;">
Para realizar la fusión, se asignan variables que contienen los identificadores de cada tabla:
</p>

```python
idvivienda = ['cd_a', 'ent', 'con', 'v_sel']
idhogar = ['cd_a', 'ent', 'con', 'v_sel', 'n_hog', 'h_mud']
idpersona = ['cd_a', 'ent', 'con', 'v_sel', 'n_hog', 'h_mud', 'n_ren']
```

### Unión de tablas
<p style="text-align: justify;">
La fusión se realiza utilizando la función <code>pd.merge</code> de la biblioteca <b>pandas</b>. A continuación, se muestra cómo unir las tablas <b>COE1T</b> y <b>COE2T</b>:
</p>

```python
import pandas as pd

# Unión de COE1T y COE2T
coet422 = pd.merge(coet1422, coet2422, on=idpersona)
```

<p style="text-align: justify;">
La tabla resultante contiene columnas duplicadas, identificadas por los sufijos <code>_x</code> y <code>_y</code>. Para manejar estas duplicaciones, se utiliza una función de limpieza:
</p>

```python
def limpiar_tabla(tabla):
    columnas_eliminar = [col for col in tabla.columns if col.endswith('_y')]
    tabla = tabla.drop(columns=columnas_eliminar)
    tabla = tabla.rename(columns={col: col.split('_')[0] for col in tabla.columns if col.endswith('_x')})
    return tabla

# Limpieza de la tabla COE1T y COE2T
coet422 = limpiar_tabla(coet422)
```

<p style="text-align: justify;">
Este proceso se repite para unir todas las tablas:
</p>

```python
# Unión de SDEMT y COET
sdemcoet422 = pd.merge(sdemt422, coet422, on=idpersona)
sdemcoet422 = limpiar_tabla(sdemcoet422)

# Unión de VIVT y HOGT
vivhogt422 = pd.merge(vivt422, hogt422, on=idvivienda)
vivhogt422 = limpiar_tabla(vivhogt422)

# Unión final de todas las tablas
completat422 = pd.merge(sdemcoet422, vivhogt422, on=idhogar)
completat422 = limpiar_tabla(completat422)
```

---

## Guardado de la base de datos
<p style="text-align: justify;">
Una vez finalizada la fusión y limpieza de las tablas, es importante guardar la base de datos para su uso posterior. En este caso, se guarda en formato <code>.csv</code>, un formato simple y ampliamente compatible:
</p>

```python
completat422.to_csv('completat422.csv', index=False)
```

---

## Conclusión
<p style="text-align: justify;">
El método presentado en este artículo proporciona una herramienta para el estudio del mercado laboral mexicano utilizando la Encuesta Nacional de Ocupación y Empleo (ENOE). A través de Python, es posible construir una base de datos multidimensional que permita realizar análisis estadísticos y visualizaciones avanzadas. Este enfoque no solo facilita el manejo de grandes volúmenes de datos, sino que también promueve el uso de herramientas de programación en el análisis de ciencias sociales.
</p>

<p style="text-align: justify;">
Se espera que este artículo sea de utilidad tanto para quienes se acercan por primera vez al análisis de datos con Python como para aquellos que buscan profundizar en el estudio del mercado laboral mexicano. Para ampliar este análisis, se pueden aplicar métodos estadísticos avanzados y visualizaciones utilizando bibliotecas como <code>matplotlib</code> o <code>seaborn</code>.
</p>

--- -->
