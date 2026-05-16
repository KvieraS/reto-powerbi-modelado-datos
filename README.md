# Reto Power BI - Modelado de Datos

Este repositorio contiene la resolución del reto de modelado de datos en Power BI.

El objetivo principal del ejercicio es cargar diferentes tablas procedentes de ficheros CSV, preparar los datos, crear las relaciones entre tablas y construir un modelo de datos siguiendo buenas prácticas de Power BI.

Durante el desarrollo se ha priorizado el uso de un modelo en estrella, evitando en la medida de lo posible un modelo en copo de nieve.

---

## Estructura del proyecto

La estructura del repositorio es la siguiente:

```text
Modelado de Datos/
│
├── data/
│   ├── DimCalendar.csv
│   ├── DimChannel.csv
│   ├── DimGeography.csv
│   ├── DimProduct.csv
│   ├── DimProductCategory.csv
│   ├── DimProductSubcategory.csv
│   ├── DimPromotion.csv
│   ├── DimStores.csv
│   └── factsales.zip
│
├── powerbi/
│   └── reto_modelado_powerbi.pbix
│
├── .gitignore
└── README.md
```

---

## Fuente de datos

Los datos utilizados en este reto se encuentran dentro de la carpeta `data/`.

Aunque el enunciado original permite trabajar mediante conexión a una base de datos Access, en este caso se han utilizado los ficheros CSV proporcionados, por lo que la carga de datos se ha realizado mediante el conector **Texto/CSV** de Power BI.

La mayoría de las tablas se incluyen directamente en formato CSV. En el caso de `FactSales.csv`, debido a su tamaño, se ha incluido comprimido dentro del archivo:

```text
factsales.zip
```

Dentro de este archivo comprimido se encuentra el fichero `FactSales.csv`, necesario para revisar o reconstruir el modelo completo.

---

## Herramientas utilizadas

- Power BI Desktop
- Power Query
- DAX
- Git
- GitHub
- Ficheros CSV

---

## Tablas utilizadas

Las tablas utilizadas inicialmente en Power BI son:

- `FactSales`
- `DimCalendar`
- `DimChannel`
- `DimStores`
- `DimProduct`
- `DimPromotion`
- `DimGeography`
- `DimProductSubcategory`
- `DimProductCategory`

La tabla principal del modelo es `FactSales`, que actúa como tabla de hechos.

El resto de tablas se utilizan como dimensiones para analizar las ventas desde diferentes perspectivas.

---

## Objetivo del reto

El objetivo del reto es aplicar los conceptos aprendidos sobre modelado de datos en Power BI, incluyendo:

- Carga de datos desde ficheros CSV.
- Transformación de datos en Power Query.
- Creación de relaciones entre tablas.
- Diseño de un modelo en estrella.
- Creación de una tabla calendario.
- Ocultación de campos técnicos utilizados en las relaciones.
- Organización del proyecto en un repositorio de GitHub.

---

## Transformaciones realizadas en Power Query

Durante la preparación de los datos se han realizado varias transformaciones para dejar el modelo preparado para el análisis.

Las principales transformaciones realizadas son:

- Carga de los ficheros CSV desde la carpeta `data/`.
- Revisión del separador correcto de los archivos.
- Corrección y asignación de tipos de datos.
- Conversión de campos de fecha.
- Creación de una columna de fecha limpia para la tabla de ventas.
- Combinación de tablas relacionadas.
- Reducción del número de tablas cargadas en el modelo final.
- Preparación del modelo siguiendo una estructura en estrella.

---

## Tratamiento de fechas

El campo original de fecha de la tabla `FactSales` no estaba directamente en formato fecha, ya que venía con una estructura de fecha y hora.

Por este motivo, se creó una columna limpia llamada `FechaVenta`, dejando únicamente la parte de fecha.

Esta columna se utiliza para relacionar la tabla de ventas con la tabla calendario creada en Power BI.

---

## Uniones realizadas

Para simplificar el modelo y evitar una estructura en copo de nieve, se han realizado varias combinaciones en Power Query.

---

### Unión de productos

Se ha combinado la tabla `DimProduct` con las tablas:

- `DimProductSubcategory`
- `DimProductCategory`

La relación lógica utilizada ha sido:

```text
DimProduct[ProductSubcategoryKey]
↓
DimProductSubcategory[ProductSubcategoryKey]

DimProductSubcategory[ProductCategoryKey]
↓
DimProductCategory[ProductCategoryKey]
```

Después de la combinación, la tabla `DimProduct` contiene también la información de subcategoría y categoría del producto.

De esta forma, el análisis por producto, subcategoría y categoría se puede realizar desde una única dimensión.

---

### Unión de tiendas y geografía

También se ha combinado la tabla `DimStores` con la tabla `DimGeography`.

La relación lógica utilizada ha sido:

```text
DimStores[GeographyKey]
↓
DimGeography[GeographyKey]
```

Después de la combinación, la tabla `DimStores` contiene también la información geográfica asociada a cada tienda.

Esto permite analizar las ventas por tienda, país, región o continente sin necesidad de mantener una dimensión geográfica separada en el modelo final.

---

## Modelo de datos

El modelo final se ha diseñado siguiendo un esquema en estrella.

La tabla central del modelo es:

- `FactSales`

Las dimensiones principales del modelo son:

- `DimProduct`
- `DimStores`
- `DimChannel`
- `DimPromotion`
- `Tabla Calendario`

Este diseño permite que las dimensiones filtren a la tabla de hechos, manteniendo un modelo limpio, sencillo y eficiente.

---

## Relaciones creadas

Las relaciones principales del modelo son:

```text
DimProduct[ProductKey]        1 → * FactSales[ProductKey]

DimStores[StoreKey]           1 → * FactSales[StoreKey]

DimChannel[ChannelKey]        1 → * FactSales[channelKey]

DimPromotion[PromotionKey]    1 → * FactSales[PromotionKey]

Tabla Calendario[Date]        1 → * FactSales[FechaVenta]
```

Todas las relaciones se han configurado con:

- Cardinalidad de uno a muchos.
- Dirección de filtro cruzado única.
- Relación activa.

La dirección del filtro va desde las dimensiones hacia la tabla de hechos.

---

## Tabla calendario

Se ha creado una tabla calendario en Power BI mediante DAX.

La tabla contiene los siguientes campos solicitados en el enunciado:

- `Date`
- `Año`
- `Mes`
- `Día`
- `Número de la semana`
- `Número del día de la Semana`

Código DAX utilizado:

```DAX
Tabla Calendario =
ADDCOLUMNS(
    CALENDAR(
        MIN(FactSales[FechaVenta]),
        MAX(FactSales[FechaVenta])
    ),
    "Año", YEAR([Date]),
    "Mes", FORMAT([Date], "MMMM"),
    "Día", DAY([Date]),
    "Número de la semana", WEEKNUM([Date], 2),
    "Número del día de la Semana", WEEKDAY([Date], 2)
)
```

Esta tabla permite analizar las ventas desde una perspectiva temporal y facilita el uso de campos como año, mes, día o número de semana.

---

## Ocultación de campos técnicos

Para mejorar la usabilidad del modelo, se han ocultado los campos utilizados únicamente para crear relaciones entre tablas.

Esto permite que en la vista de informe solo aparezcan los campos útiles para el análisis.

---

### Campos ocultados en `FactSales`

- `DateKey`
- `FechaVenta`
- `channelKey`
- `StoreKey`
- `ProductKey`
- `PromotionKey`

---

### Campos ocultados en `DimProduct`

- `ProductKey`
- `ProductSubcategoryKey`
- `ProductCategoryKey`

---

### Campos ocultados en `DimStores`

- `StoreKey`
- `GeographyKey`

---

### Campos ocultados en `DimChannel`

- `ChannelKey`

---

### Campos ocultados en `DimPromotion`

- `PromotionKey`

---

## Archivo Power BI

El archivo Power BI del reto se encuentra dentro de la carpeta:

```text
powerbi/
```

El archivo incluido es:

```text
reto_modelado_powerbi.pbix
```

Este fichero contiene el modelo de datos desarrollado en Power BI.

---

## Archivo FactSales

El archivo `FactSales.csv` original supera el límite permitido por GitHub para archivos individuales, por lo que no se ha subido directamente en formato CSV.

Para poder incluirlo en el repositorio, se ha añadido una versión comprimida del fichero con el nombre:

```text
factsales.zip
```

Dentro de este archivo comprimido se encuentra el fichero `FactSales.csv`, necesario para revisar o reconstruir el modelo de datos completo.

---

## Resultado final

El resultado final es un modelo de datos preparado para el análisis en Power BI.

El modelo permite analizar ventas por:

- Fecha
- Año
- Mes
- Producto
- Categoría de producto
- Subcategoría de producto
- Tienda
- País o región
- Canal
- Promoción

---

## Buenas prácticas aplicadas

Durante el desarrollo del reto se han aplicado varias buenas prácticas de modelado en Power BI:

- Uso de una tabla de hechos central.
- Uso de dimensiones para filtrar la tabla de hechos.
- Priorización del modelo en estrella.
- Reducción de relaciones innecesarias.
- Combinación de tablas auxiliares en Power Query.
- Creación de una tabla calendario independiente.
- Relaciones de uno a muchos.
- Dirección de filtro cruzado única.
- Ocultación de claves técnicas en la vista de informe.
- Organización del proyecto en carpetas separadas.

---

## Conclusión

Este reto ha permitido practicar los conceptos principales de modelado de datos en Power BI.

Durante el desarrollo se ha trabajado la carga de datos desde ficheros CSV, la transformación de datos en Power Query, la combinación de tablas, la creación de una tabla calendario, el diseño de relaciones uno a muchos y la ocultación de campos técnicos.

El modelo final sigue una estructura en estrella, lo que facilita el análisis y mejora la claridad del modelo para futuros informes.
