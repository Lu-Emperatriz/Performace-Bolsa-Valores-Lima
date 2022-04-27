# **Desempeño Financiero en la Bolsa de Valores de Lima (2015-2020)**

[**Ver el dashboard en línea en Tableau**](https://tabsoft.co/3sUUAek)


<center><a href="https://tabsoft.co/38C1f5B"><img src="https://imgur.com/8duoiFH.png" width="700" /></a></center>


* * *
## _Contenido_

* **[0. Definición del Problema](#Definición-del-Problema)**
* **[1. Preparación de Datos](#Preparación-de-Datos)**
  * [1.1. Fuente de datos](##Fuente-de-datos)
  * [1.2. Extracción de Datos](##Extracción-de-Datos)
* **[2. Limpieza de Datos](#Limpieza-de-Datos)**
* **[3. Modelamiento de Datos](#Modelamiento-de-Datos)**
* **[4. Visualización y Análisis de Datos](#Visualización-y-Análisis-de-Datos)**
* **[5. Conclusiones y Recomendaciones](#Conclusiones-y-Recomendaciones)**

***


# Definición del Problema
El mercado accionario del Perú mejor conocido como la Bolsa de Valores de Lima (BVL) es considerado como uno de los mercados de acciones más pequeño de Latinoamérica a pesar de su crecimiento favorable en el último quinquenio, al duplicar su tamaño de deuda pública de 8.1% en 2015 a 15.7% del PBI en 2019. 

### ❓ **<u>Preguntas de análisis</u>**: 
* ¿Las empresa tienen capacidad para recaudar fondos y cubrir sus obligaciones?
* ¿Qué sector presenta mayor rentabilidad?
* ¿La empresas con mayores ingresos generan mejores márgenes de utilidad?



### 🎯 **<u>Objetivo</u>**<br>
Ofrecer una visión global del estado financiero histórico de las empresas que cotizan en la Bolsa de Valores de Lima, este análisis te servirá de soporte para la toma de decisiones de inversión.

<br>

<center>

| Alcance de estudio | Tipo de Análisis | Tecnologías a usar |
| --- | :-: | --: |
| Longitudinal: Del periodo 2015 al 2020 | Descriptivo | SQL-Python-Tableau-Markdown |

</center>



# Preparación de Datos

## Fuente de datos
Para realizar el análisis se recopiló la informacion financiera de las empresas del 2015 al 2020, disponible en la [_web de la Bolsa de Valores de Lima_](https://www.bvl.com.pe/emisores/listado-emisores). Los estados financieros usados fueron el Estado de Resultados y Estado de Situación Financiera. Además, para evitar problemas con el cálculo de indicadores, se descartaron las empresas que no presentaron información durante este intervalo de tiempo de tal forma que sean homogéneos y comparables. Fueron descartadas las empresas Atria Energia, Medrock, Minera IRL Limited.

Los ratios financieros calculados para responder a las preguntas fueron:
1. **Ratio de Liquidez | Current ratio(CR):** Indica capacidad de empresas para recaudar fondos y cubrir sus obligaciones  (Activo corriente / Pasivo corriente)
2. **Ratio de Deuda | Debt ratio (DR):** Mide qué tan bien el flujo de efectivo de una empresa puede cubrir su deuda a largo plazo (Pasivo total /Activo total)
3. **Rentabilidad sobre Activos | Return to Assets (ROA):** Mide qué tan rentable es una empresa en relación con sus activos totales (Utilidad Neta / Activo total)
4. **Margen Neto | Net margin (MN):** Mide cuánto ingreso neto o ganancia se genera con ingresos de las ventras (Utilidad neta / Ventas netas)

## Extracción de Datos
Para la extracción se usó scraping en Python con el entorno Selenium y la librería Pandas (**ver código en Python** [**AQUÍ.**](https://github.com/Lu-Emperatriz/Lima-Stock-Performance/blob/main/pythonScript.ipynb)).

El conjunto de datos contiene los siguientes atributos:

<center>

| Ratio | Descripción |
| --- | :-: |
| TASSET | Activo total |
| TLIABILITY | Pasivo total |
| CURR\_ASS | Activo corriente |
| CURR\_LIAB | Pasivo corriente |
| NET\_SALES | Ventas netas es el total de ganancias |
| NET\_PROF | Ganancias netas o utilidad neta, despues de restar todos los intereses, operaciones e impuestos. |

</center>

# Limpieza de Datos
Se exportaron **seis tablas en total**: 
* _'SECTOR'_ con registros del sector al que pertenece cada empresa y la URL de la información.
* _'TAB15'_ que contiene los datos del año 2015, _'TAB16'_ con los datos del periodo 2016, y de igual manera para las tablas restantes _'TAB17', 'TAB18', 'TAB19'_ y _'TAB20'._ 

Luego se agregó un _ID identity_ para identificar a las empresas y el año correspondiente.


```sql
--1. Agregar ID identity a cada tabla
ALTER TABLE TAB15
ADD ID int identity(1,1)
GO
--2. Agregar año a tabla 
ALTER TABLE TAB15 ADD year integer
UPDATE TAB15 SET year = 2015
```

Finalmente, se unieron las tablas en una nueva vista, que incluye el sector al que pertenece cada empresa y el cálculo de los ratios correspondientes.

¡Veamos el código!


```sql
--1. UNION todas las tablas (alias 'a' para TAB15, 'b' para TAB16, ... y 'f' para TAB20)
--2. CÁLCULO DE FÓRMULAS (Donde 'CR' es el alias para Ratio de liquidez, 'DB' es el índice de deuda y 'NM' es el margen neto)

SELECT a.id, a.year, z.NAME, z.sec, a.TASSET, a.TLIABILITY, a.CURR_ASS, a.CURR_LIAB, a.NET_SALES, a.NET_PROF
	,a.CURR_ASS/a.CURR_LIAB AS CR, a.TLIABILITY/a.TASSET AS DR, a.NET_PROF/a.TASSET AS ROA,  a.NET_SALES/a.NET_PROF AS NM
FROM TAB15 a, SECTOR z	WHERE a.id = z.ID  
UNION ALL 	(

SELECT b.id , b.year, z.NAME, z.sec, b.TASSET, b.TLIABILITY , b.CURR_ASS, b.CURR_LIAB, b.NET_SALES, b.NET_PROF
	,b.CURR_ASS/b.CURR_LIAB AS CR, b.TLIABILITY/b.TASSET AS DR, b.NET_PROF/b.TASSET AS ROA,  b.NET_SALES/b.NET_PROF AS NM
FROM TAB16 b, SECTOR z 	WHERE b.id = z.ID
UNION ALL 	(

SELECT d.id , d.year, z.NAME, z.sec, d.TASSET, d.TLIABILITY, d.CURR_ASS, d.CURR_LIAB, d.NET_SALES, d.NET_PROF
	,d.CURR_ASS/d.CURR_LIAB AS CR, d.TLIABILITY/d.TASSET AS DR, d.NET_PROF/d.TASSET AS ROA,  d.NET_SALES/d.NET_PROF AS NM
FROM TAB18 d, SECTOR z 	WHERE d.id = z.ID
UNION ALL 	(

SELECT c.id , c.year, z.NAME, z.sec, c.TASSET, c.TLIABILITY, c.CURR_ASS, c.CURR_LIAB, c.NET_SALES, c.NET_PROF
	,c.CURR_ASS/c.CURR_LIAB AS CR, c.TLIABILITY/c.TASSET AS DR, c.NET_PROF/c.TASSET AS ROA,  c.NET_SALES/c.NET_PROF AS NM
FROM TAB17 c, SECTOR z	WHERE c.id = z.ID)
UNION ALL 	(

SELECT e.id , e.year, z.NAME, z.sec, e.TASSET, e.TLIABILITY, e.CURR_ASS, e.CURR_LIAB, e.NET_SALES, e.NET_PROF
	,e.CURR_ASS/e.CURR_LIAB AS CR, e.TLIABILITY/e.TASSET AS DR, e.NET_PROF/e.TASSET AS ROA,  e.NET_SALES/e.NET_PROF AS NM
FROM TAB19 e, SECTOR z	WHERE e.id = z.ID
UNION ALL 	(

SELECT f.id , f.year, z.NAME, z.sec, f.TASSET, f.TLIABILITY, f.CURR_ASS, f.CURR_LIAB, f.NET_SALES, f.NET_PROF
	,f.CURR_ASS/f.CURR_LIAB AS CR, f.TLIABILITY/f.TASSET AS DR, f.NET_PROF/f.TASSET AS ROA,  f.NET_SALES/f.NET_PROF AS NM
FROM TAB20 f, SECTOR z	WHERE f.id = z.ID	))))

ORDER BY z.NAME, year
```

# Modelamiento de Datos

💡 El modelado de datos conecta las diferentes bases de datos extraídas con el fin de estructurar y organizar los datos para su mejor comprensión.

Primero se estableció como _primary key_ a la columna _ID_ en cada tabla, esta 'llave' es el atributo que concide en las dos tablas a relacionarse. La estructura final se muestra a continuación:

<center><img src="https://imgur.com/Q91NPpo.png" width="300" height="400" /></center>

# Visualización y Análisis de Datos

Para una mejor comprensión de los datos se construyó un dashboard dinámico en Tableau.  
Pruébalo [**AQUÍ**](https://tabsoft.co/38C1f5B)

✔️ **¿Las empresas tienen capacidad para recaudar fondos y cubrir sus obligaciones?**

En el boxplot RATIO DE LIQUIDEZ POR SECTOR se refleja la capacidad de las empresas de los sector es Industrial, Minería y Agrario para satisfacer sus deudas a corto plazo, pues aproximadamente el 75% (cuartil inferior) de las compañías tienen una liquidez mayor a 1. El sector de Servicios por su parte, tiene una liquidez media de 1.3, pero la mitad de estas tienen una liquidez negativa.

<center><img src="https://imgur.com/DZsV16I.png" height="150" widht=auto/></center>

Por otra parte, la línea de tendencia positiva mostrada en el gráfico FINANCIAMIENTO POR DEUDAS sugiere que las empresas prefieren financiar sus actividades con financiación externa en lugar de fondos propios, con excepción del sector Minería. Se puede deducir que esta decición con respecto a la financiación depende de las actividades productivas específicas de cada empresa.

<center><img src="https://imgur.com/7UWHYai.png" height="140" widht=auto /></center>


✔️ **¿Qué sector presenta mayor rentabilidad?**

El ROA indica qué tal eficiente es empresa para generar utilidades con su disponibilidad de activos, para este análisis se consideran óptimos a los valores de ROA que superen el 5%. Puesto que el valor del ROA varía de acuerdo a la indutria, se analizaron tomando en cuenta el periodo de tiempo.

<center><img src="https://imgur.com/AFArrAm.png" height="110" widht=90 /></center>


En el sector Industrial y Minería Sector se nota una ineficiencia en uso de activos, pues sus utilidades tienen tendencia a disminuir con el paso del tiempo como muestra la visualización EVOLUCIÓN DEL ROA.

✔️ **¿La empresas con mayores ingresos generan mejores márgenes de utilidad?**

No. El gráfico MARGEN DE UTILIDAD NETA, indica que el mayor margen neto no corresponde a las empresas que generan más ventas, por el contrario, empresas pequeñas como "Cementos Pacasmayo" tienen mayor margen de utilidad que grandes compañías como "Petroperú".

<center><img src="https://imgur.com/L8u7soe.png" height="350" /></center>

# Conclusiones y Recomendaciones

Los resultados encontrados en este estudio sirven como referencia al inversor acerca de la toma de decisiones concerniente a su cartera de valores. De este estudio se puede concluir que:

- El sector Minería se destaca sobre los demás debido a su valor de liquidez y capacidad de cubrir sus obligaciones, no obstante, del 2016 al 2020 se observó una caída en su rentabilidad.
- Las empresas más grandes no son necesariamente las más eficientes en sus recursos, posiblemente por el requerimiento de mayores costos operacionales.
- El valor de los indicadores dependerán del ratio y del sector, por lo que se recomienda analizar los libros de cuentas de las empresas a detalle.

Se invita a los interesados a realizar un análisis más exhaustivo haciendo uso adecuado de esta herramienta para tomar sus decisiones de acuerdo a sus objetivos personales.

* * *

### 📌 **Conoce más de mis proyectos [AQUÍ](https://github.com/Lu-Emperatriz)**

Lucero Emperatriz.
