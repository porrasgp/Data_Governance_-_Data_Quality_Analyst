# Carpeta `tables/` — una definición por cada tabla

Cada archivo `.tmdl` describe una tabla del modelo: sus **columnas**, sus **tipos de
dato** y la **consulta de Power Query** que trae los datos.

| Archivo | Tabla | Visible | Rol |
|---|---|---|---|
| `raw_data_ERP.tmdl` | `raw_data_ERP` | Sí | **Hechos**: líneas de factura. |
| `dim_cliente_ERP.tmdl` | `dim_cliente_ERP` | Sí | **Dimensión**: clientes. |
| `dim_prod_ERP.tmdl` | `dim_prod_ERP` | Sí | **Dimensión**: productos. |
| `LocalDateTable_d2434ed1-….tmdl` | Calendario local | No (oculta) | Fechas automáticas de `InvoiceDate`. |
| `DateTableTemplate_e8701a2b-….tmdl` | Plantilla de calendario | No (oculta) | Molde que usa Power BI para crear calendarios. |

## Cómo leer un archivo `.tmdl`

Ejemplo, tomado de `dim_cliente_ERP.tmdl`:

```text
table dim_cliente_ERP                  ← nombre de la tabla
    column CustomerID                  ← una columna
        dataType: int64                ← tipo: número entero
        summarizeBy: none              ← no sumar nunca esta columna
        sourceColumn: CustomerID       ← de qué columna del CSV viene
    ...
    partition dim_cliente_ERP = m      ← de dónde vienen los datos
        mode: import                   ← se copian dentro del modelo
        source = let ... in ...        ← la consulta de Power Query
```

Palabras frecuentes:

| Palabra | Significado |
|---|---|
| `dataType` | Tipo de dato: `int64` (entero), `string` (texto), `double` (decimal), `dateTime` (fecha y hora). |
| `summarizeBy` | Qué hace Power BI al arrastrar la columna a un gráfico: `sum`, `none`... |
| `lineageTag` | Identificador interno único; sirve para seguir al objeto aunque se le cambie el nombre. No tocar. |
| `partition ... = m` | Los datos vienen de una consulta en lenguaje **M** (Power Query). |
| `mode: import` | Los datos se importan al modelo (no es consulta en vivo). |
| `isHidden` | La columna o tabla no aparece al crear reportes. |

---

## `raw_data_ERP` — la tabla de hechos

Una fila por producto vendido en una factura.

| Columna | Tipo | `summarizeBy` | Comentario |
|---|---|---|---|
| `InvoiceNo` | Entero | `sum` | Número de factura. |
| `StockCode` | Texto | `none` | Código de producto. |
| `Quantity` | Entero | `sum` | Cantidad vendida. |
| `InvoiceDate` | Fecha y hora | `none` | Se relaciona con el calendario oculto. |
| `UnitPrice` | Decimal | `sum` | Precio unitario. |
| `CustomerID` | Entero | `none` | Se relaciona con `dim_cliente_ERP`. |
| `Country` | Texto | `none` | País de la venta. |

**Consulta de Power Query (en palabras simples):**

1. `Csv.Document(...)`: abre `raw_data_ERP.csv` (coma como separador, 7 columnas).
2. `Table.PromoteHeaders`: usa la primera fila como nombres de columna.
3. `Table.TransformColumnTypes`: asigna el tipo correcto a cada columna.

**Observaciones de gobierno de datos:**

- **`InvoiceNo` se resume con `sum`.** Sumar números de factura no tiene sentido. Debería ser `none`, o incluso tipo texto, porque es un identificador y no una cantidad.
- **`UnitPrice` también se resume con `sum`.** Sumar precios unitarios tampoco tiene sentido; en un reporte conviene usar promedio o calcular el importe (`Quantity * UnitPrice`) en una medida.
- **No hay una columna de importe de línea ni medidas.** Los totales de venta tendrían que calcularse en cada reporte.
- **`Country` está en esta tabla y `Pais` en `dim_cliente_ERP`.** Son dos países distintos (el de la venta y el del cliente), pero con nombres parecidos pueden confundirse. Conviene documentar la diferencia.

---

## `dim_cliente_ERP` — dimensión de clientes

| Columna | Tipo | `summarizeBy` | Comentario |
|---|---|---|---|
| `CustomerID` | Entero | `none` | Clave. Única en los datos (4,372 valores). |
| `NombreCliente` | Texto | `none` | Nombre completo. |
| `Pais` | Texto | `none` | País del cliente. |
| `CategoriaCliente` | Texto | `none` | Bronce, Plata, Oro o Platino. |

Consulta: abre el CSV (4 columnas), promueve encabezados y fija los tipos (entero
para `CustomerID`, texto para el resto).

Las columnas ya tienen `summarizeBy: none`, lo cual es correcto: no tiene sentido
sumar un código ni un nombre.

---

## `dim_prod_ERP` — dimensión de productos

| Columna | Tipo | `summarizeBy` | Comentario |
|---|---|---|---|
| `StockCode` | Texto | `none` | Clave. Es texto porque hay códigos con letras (`82613A`). |
| `Descripcion` | Texto | `none` | Nombre del producto. |
| `CategoriaProducto` | Texto | `none` | Una de 10 categorías. |

Consulta: abre el CSV (3 columnas) y promueve encabezados. **No tiene paso de
cambio de tipos**; todas las columnas quedan como texto, lo cual es correcto aquí.

**Observaciones de gobierno de datos:**

- **La consulta usa `QuoteStyle = QuoteStyle.None`**, que no interpreta las comillas. En el CSV hay 153 líneas con comillas porque la descripción incluye una coma (por ejemplo `"METAL SIGN,CUPCAKE SINGLE HOOK"`). Es muy probable que esas filas se corten mal. Se recomienda `QuoteStyle.Csv`.
- **No define la codificación del archivo.** El CSV es ISO-8859 y no UTF-8. Conviene indicar la codificación en `Csv.Document` (`Encoding = 1252`) para que símbolos como `£` se lean bien.
- **No hay relación con `raw_data_ERP`.** Ver `../relationships.tmdl`.

---

## Tablas de calendario ocultas

Power BI las crea automáticamente. No hay que editarlas.

### `LocalDateTable_d2434ed1-…`
Calendario de la columna `InvoiceDate`. Se genera con una fórmula de DAX:

```text
Calendar(Date(Year(MIN(raw_data_ERP[InvoiceDate])), 1, 1),
         Date(Year(MAX(raw_data_ERP[InvoiceDate])), 12, 31))
```

Es decir, desde el 1 de enero del primer año de ventas hasta el 31 de diciembre del
último. Incluye las columnas `Year`, `MonthNo`, `Month`, `QuarterNo`, `Quarter` y
`Day`, y una jerarquía **Año → Trimestre → Mes → Día** para hacer *drill-down* en
los gráficos.

### `DateTableTemplate_e8701a2b-…`
Una plantilla con la misma estructura, pero con un calendario de un solo día
(`2015-01-01`). Power BI la usa como molde cuando crea nuevos calendarios
automáticos. Es normal que aparezca aquí.

## Mejoras recomendadas

| Prioridad | Mejora |
|---|---|
| Alta | Crear la relación `StockCode` entre `raw_data_ERP` y `dim_prod_ERP`. |
| Alta | Cambiar `dim_prod_ERP` a `QuoteStyle.Csv` y fijar la codificación. |
| Media | Poner `summarizeBy: none` en `InvoiceNo` y revisar `UnitPrice`. |
| Media | Agregar medidas: ventas totales (`Quantity * UnitPrice`), número de facturas, número de clientes. |
| Media | Reemplazar las rutas fijas de los CSV por un parámetro. |
| Baja | Agregar descripciones (`description`) a tablas y columnas como diccionario de datos. |
