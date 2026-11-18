# Data Governance & Data Quality Analyst

Prueba técnica de **Gobierno de Datos y Calidad de Datos**. El repositorio contiene
el modelo de Power BI y las tablas dimensionales de un sistema **ERP** de ventas,
listos para revisar su calidad.

## ¿De qué trata?

Una empresa vende productos a clientes de muchos países y registra cada venta en su
ERP. Antes de hacer reportes hay que responder preguntas de gobierno de datos:

- ¿Los datos son **completos** (sin campos vacíos)?
- ¿Son **únicos** (sin clientes ni productos repetidos)?
- ¿Son **consistentes** (los países y las categorías se escriben siempre igual)?
- ¿Las tablas están **relacionadas** correctamente en el modelo?

```text
  CSV del ERP ──► Power BI (modelo semántico) ──► Revisión de calidad ──► Reportes confiables
```

## ¿Qué hay en el repositorio?

```text
Data_Governance_-_Data_Quality_Analyst/
├── README.md                              ← estás aquí
├── LICENSE                                Licencia MIT
└── Data_Governance_&_Data_Quality_Analyst/
    ├── README.md                          Visión general y hallazgos de calidad
    ├── data/                              Tablas dimensionales en CSV (clientes y productos)
    └── backup_modelo_original/            Copia de seguridad del modelo de Power BI (TMDL)
        └── definition/
            ├── cultures/                  Idioma del modelo
            └── tables/                    Definición de cada tabla
```

## Guías por carpeta

| Carpeta | Guía | Qué explica |
|---|---|---|
| Proyecto | [`README.md`](Data_Governance_&_Data_Quality_Analyst/README.md) | El caso, el modelo y los hallazgos. |
| `data/` | [`data/README.md`](Data_Governance_&_Data_Quality_Analyst/data/README.md) | Columnas, tamaños y calidad de los CSV. |
| `backup_modelo_original/` | [`README.md`](Data_Governance_&_Data_Quality_Analyst/backup_modelo_original/README.md) | Qué es el respaldo y cómo restaurarlo. |
| `definition/` | [`README.md`](Data_Governance_&_Data_Quality_Analyst/backup_modelo_original/definition/README.md) | Los archivos del modelo y las relaciones. |
| `tables/` | [`README.md`](Data_Governance_&_Data_Quality_Analyst/backup_modelo_original/definition/tables/README.md) | Cada tabla, columna por columna. |
| `cultures/` | [`README.md`](Data_Governance_&_Data_Quality_Analyst/backup_modelo_original/definition/cultures/README.md) | La configuración de idioma. |

## Resumen de los datos

| Tabla | Filas | Qué es |
|---|---|---|
| `dim_cliente_ERP` | 4,372 | Clientes con país y categoría (Bronce, Plata, Oro, Platino). |
| `dim_prod_ERP` | 4,070 | Productos con descripción y categoría. |
| `raw_data_ERP` | no incluida | Líneas de factura (la tabla de hechos). Ver aviso abajo. |

## Avisos importantes

1. **Falta `raw_data_ERP.csv`.** El modelo la usa, pero el archivo no está en el repositorio. Sin ella no se pueden actualizar los datos del modelo.
2. **Las rutas de los CSV son de otro equipo.** Apuntan a `C:\Users\Admin\Downloads\...`. Hay que cambiarlas en Power Query al restaurar el modelo.
3. **Faltan la relación de productos y las medidas.** Ver los hallazgos en el README del proyecto.

## Licencia

MIT. Consulta [`LICENSE`](LICENSE).
