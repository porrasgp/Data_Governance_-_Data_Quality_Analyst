# Carpeta `definition/` — el corazón del modelo

Esta carpeta contiene la definición completa del modelo. Cada archivo responde una
pregunta distinta:

| Archivo / carpeta | Pregunta que responde |
|---|---|
| `database.tmdl` | ¿Con qué versión del motor funciona el modelo? |
| `model.tmdl` | ¿Qué tablas hay y cómo se comporta el modelo en general? |
| `relationships.tmdl` | ¿Cómo se conectan las tablas entre sí? |
| `cultures/` | ¿Qué idioma y formato regional usa? |
| `tables/` | ¿Cómo es cada tabla por dentro? |

---

## `database.tmdl`

```text
database
	compatibilityLevel: 1606
```

Indica el **nivel de compatibilidad** (1606), es decir, qué funciones del motor
tabular de Power BI puede usar el modelo. Es como la versión de un formato de archivo.
No hace falta tocarlo.

## `model.tmdl`

Es la "portada" del modelo.

| Línea | Qué significa |
|---|---|
| `culture: en-US` | Idioma y formato regional del modelo: inglés de EE. UU. |
| `defaultPowerBIDataSourceVersion: powerBI_V3` | Usa la versión actual de fuentes de datos de Power BI. |
| `sourceQueryCulture: en-US` | Las consultas interpretan fechas y números con formato de EE. UU. (por ejemplo, `12/31/2026` y `1,234.50`). |
| `valueFilterBehavior: independent` | Los filtros sobre valores actúan de manera independiente. |
| `returnErrorValuesAsNull` | Si un cálculo da error, devuelve vacío en vez de romper el reporte. |
| `__PBI_TimeIntelligenceEnabled = 1` | Activa la fecha automática (agrupar por año, trimestre, mes y día). |
| `PBI_QueryOrder` | El orden de las consultas: `dim_cliente_ERP`, `dim_prod_ERP`, `raw_data_ERP`. |
| `PBI_ProTooling = ["DevMode"]` | El modelo se creó con el modo desarrollador (archivos de texto). |
| `ref table ...` | La lista de las **5 tablas** del modelo. |

Las 5 tablas son tres visibles (`dim_cliente_ERP`, `dim_prod_ERP`, `raw_data_ERP`)
y dos ocultas de calendario creadas por Power BI.

## `relationships.tmdl`

Una **relación** une dos tablas por una columna en común, como un `BUSCARV` de
Excel pero permanente. Este modelo tiene **dos**:

```text
 raw_data_ERP.CustomerID  ───────►  dim_cliente_ERP.CustomerID
 (muchas facturas)                  (un solo cliente)
 Detectada automáticamente por Power BI

 raw_data_ERP.InvoiceDate ───────►  LocalDateTable_….Date
 (cada fecha de factura)            (calendario oculto)
 Creada automáticamente para la jerarquía de fechas
```

| Relación | Tipo | Para qué sirve |
|---|---|---|
| `raw_data_ERP[CustomerID]` → `dim_cliente_ERP[CustomerID]` | Muchos a uno (la predeterminada) | Ver las ventas por país y categoría de cliente. |
| `raw_data_ERP[InvoiceDate]` → calendario oculto | Por parte de fecha (`datePartOnly`) | Agrupar las ventas por año, trimestre, mes y día. |

### Lo que falta

**No hay relación entre `raw_data_ERP[StockCode]` y `dim_prod_ERP[StockCode]`.**
Por eso, hoy el modelo no puede mostrar las ventas por categoría de producto.
Para corregirlo, en Power BI se crea una relación de **muchos a uno** de
`raw_data_ERP` hacia `dim_prod_ERP` por `StockCode`. La clave de `dim_prod_ERP`
es única (4,070 códigos sin repetir), así que la relación es válida.

### Una relación es una promesa de calidad

Una relación funciona bien solo si la clave del lado "uno" es **única** y si todas
las claves del lado "muchos" **existen** en la otra tabla. Es un buen control de
calidad de datos: si hay facturas con un `CustomerID` que no existe en
`dim_cliente_ERP`, esas ventas aparecerán sin cliente (en blanco).

## Carpetas

- [`tables/`](tables/README.md): cada tabla, columna por columna.
- [`cultures/`](cultures/README.md): la configuración de idioma.
