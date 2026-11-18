# Proyecto: Data Governance & Data Quality Analyst

Esta carpeta es el proyecto en sí: los datos de origen (`data/`) y la copia de
seguridad del modelo de Power BI (`backup_modelo_original/`).

## ¿Qué es un "modelo" de Power BI?

Es el conjunto de **tablas**, **relaciones** y **cálculos** que están detrás de un
reporte. Imagina un archivo de Excel con varias hojas conectadas entre sí: el
modelo define qué hojas hay y cómo se enlazan.

Este modelo tiene tres tablas conectadas con un esquema **estrella**:

```text
                      ┌──────────────────────┐
                      │   dim_cliente_ERP    │   quién compra
                      │  CustomerID (clave)  │
                      └──────────▲───────────┘
                                 │  relación por CustomerID
 ┌────────────────────┐   ┌──────┴───────────┐   ┌─────────────────────┐
 │   dim_prod_ERP     │   │   raw_data_ERP   │──►│ Calendario (oculto) │
 │  StockCode (clave) │ ? │  una línea de    │   │ por InvoiceDate     │
 │  qué se vende      │···│  factura         │   │ año/trim/mes/día    │
 └────────────────────┘   └──────────────────┘   └─────────────────────┘
          (el "?" significa que esta relación NO existe en el modelo; ver hallazgos)
```

- **`raw_data_ERP`** (tabla de hechos): una fila por producto vendido en una factura.
- **`dim_cliente_ERP`** (dimensión): datos de cada cliente.
- **`dim_prod_ERP`** (dimensión): datos de cada producto.
- **Tabla de calendario**: Power BI la crea sola (oculta) a partir de `InvoiceDate`
  para poder agrupar por año, trimestre, mes y día.

## Columnas de cada tabla

| Tabla | Columna | Significado |
|---|---|---|
| `raw_data_ERP` | `InvoiceNo` | Número de factura. |
| | `StockCode` | Código del producto vendido. |
| | `Quantity` | Cantidad vendida. |
| | `InvoiceDate` | Fecha y hora de la factura. |
| | `UnitPrice` | Precio por unidad. |
| | `CustomerID` | Cliente que compró. |
| | `Country` | País de la venta. |
| `dim_cliente_ERP` | `CustomerID` | Código del cliente (clave). |
| | `NombreCliente` | Nombre completo. |
| | `Pais` | País del cliente. |
| | `CategoriaCliente` | Bronce, Plata, Oro o Platino. |
| `dim_prod_ERP` | `StockCode` | Código del producto (clave). |
| | `Descripcion` | Nombre del producto. |
| | `CategoriaProducto` | Una de 10 categorías (Hogar General, Cocina y Comedor, ...). |

## Hallazgos de calidad de datos

Se revisaron los dos CSV disponibles. **Lo que está bien:**

| Control | `dim_cliente_ERP` | `dim_prod_ERP` |
|---|---|---|
| Claves únicas (sin repetidos) | Sí, 4,372 | Sí, 4,070 |
| Filas completamente duplicadas | 0 | 0 |
| Campos vacíos | 0 | 0 |
| Espacios sobrantes al inicio o final | 0 | 0 |

**Lo que hay que mejorar:**

| # | Hallazgo | Dimensión de calidad | Por qué importa |
|---|---|---|---|
| 1 | **`dim_prod_ERP` no está relacionada con `raw_data_ERP`.** El modelo solo relaciona clientes y fechas. | Integridad | Los reportes no podrán filtrar las ventas por categoría de producto. |
| 2 | **176 productos tienen la descripción `DESCRIPCION NO DISPONIBLE`.** | Completitud | Un valor de relleno oculta que falta información. |
| 3 | **16 códigos de `dim_prod_ERP` no son productos**: `POST`, `DOT`, `C2` (envíos), `D` (descuento), `BANK CHARGES`, `AMAZONFEE`, `CRUK` (comisiones), `B` (deuda incobrable), `M`/`m` (manual), `S` (muestras) y 5 vales `gift_*`. | Validez | Los 16 aparecen como categoría `Hogar General`, lo que infla esa categoría. Convendría excluirlos o darles una categoría propia como "Servicios y ajustes". |
| 4 | **`Hogar General` es la categoría más grande** (1,609 de 4,070 productos, casi 40 %). | Precisión | Parece una categoría "comodín". |
| 5 | **Países escritos de forma distinta o ambigua** en `Pais`: `EIRE` (nombre antiguo de Irlanda), `USA`, `RSA`, `Unspecified` (4) y `European Community` (1). | Consistencia | Los mismos países pueden quedar divididos o sin identificar. |
| 6 | **`dim_prod_ERP.csv` no está en UTF-8** (es ISO-8859). Contiene símbolos como `£`. | Validez | Si se lee con otra codificación, los caracteres especiales se ven mal. |
| 7 | **153 líneas de `dim_prod_ERP.csv` tienen comillas** porque la descripción incluye una coma (por ejemplo `"METAL SIGN,CUPCAKE SINGLE HOOK"`). La consulta de Power Query usa `QuoteStyle.None`, que **no interpreta las comillas**. | Exactitud | Esas filas probablemente se separan en columnas de más y la categoría queda mal asignada. Se recomienda usar `QuoteStyle.Csv`. |
| 8 | **Descripciones con formato mixto**: la mayoría en MAYÚSCULAS y 63 con minúsculas (`Damaged`, `mailout`, `ebay`). | Consistencia | Complica búsquedas y agrupaciones por texto. |
| 9 | **133 descripciones aparecen en más de un código** (incluye el texto de relleno del hallazgo 2; el resto son variantes como `82613A`, `82613B`, `82613C`). | Unicidad | Es normal en variantes, pero conviene confirmarlo con el negocio. |
| 10 | **El modelo no tiene medidas** (ventas totales, número de clientes, etc.). | Gobierno | Sin medidas definidas, cada reporte podría calcular las ventas de forma distinta. |

> Los hallazgos 1 y 10 salen de los archivos del modelo. Los hallazgos 2 a 9 salen
> del análisis de los CSV. Los datos de `raw_data_ERP` no se pudieron revisar
> porque el archivo no está en el repositorio. El hallazgo 7 es una deducción a
> partir del comportamiento documentado de `QuoteStyle.None`; confírmalo en Power
> BI antes de corregir.

## Cómo restaurar el modelo

1. Consigue `raw_data_ERP.csv` y colócalo en la carpeta `data/`.
2. Crea un proyecto de Power BI en formato **PBIP** con el modo desarrollador activado.
3. Copia la carpeta `backup_modelo_original/definition/` dentro de la carpeta `definition/` del modelo semántico del proyecto.
4. En Power Query, cambia las tres rutas `C:\Users\Admin\Downloads\...` por la ruta real de tu carpeta `data/`.
5. Actualiza los datos y revisa las relaciones.

Detalles de cada archivo en [`backup_modelo_original/README.md`](backup_modelo_original/README.md).

## Siguientes pasos recomendados

- Agregar `raw_data_ERP.csv` al repositorio (o una muestra) para completar el análisis.
- Crear la relación `raw_data_ERP[StockCode]` → `dim_prod_ERP[StockCode]`.
- Excluir o reclasificar los 16 códigos que no son productos.
- Normalizar los nombres de país (`EIRE` → `Ireland`, `USA` → `United States`, etc.).
- Cambiar la consulta de `dim_prod_ERP` a `QuoteStyle.Csv` y fijar la codificación.
- Definir medidas de negocio en el modelo.
- Documentar un **diccionario de datos** con responsable (*data owner*) por tabla.
