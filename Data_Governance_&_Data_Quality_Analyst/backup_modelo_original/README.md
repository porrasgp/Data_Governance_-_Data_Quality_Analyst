# Carpeta `backup_modelo_original/` — copia de seguridad del modelo

Aquí está guardada la **copia original del modelo semántico de Power BI**, antes de
hacerle cambios. Si algo sale mal al modificar el modelo, se puede volver a este
punto de partida.

## ¿Qué contiene?

```text
backup_modelo_original/
└── definition/                 ← todo el modelo, escrito como archivos de texto
    ├── database.tmdl           Versión de compatibilidad del modelo
    ├── model.tmdl              Configuración general y lista de tablas
    ├── relationships.tmdl      Cómo se conectan las tablas
    ├── cultures/               Idioma y formato regional
    └── tables/                 Una definición por cada tabla
```

Todo está en formato **TMDL** (*Tabular Model Definition Language*).

## ¿Qué es TMDL?

Normalmente un reporte de Power BI se guarda en un único archivo binario `.pbix`
que no se puede leer ni comparar. **TMDL** guarda el mismo modelo como **archivos de
texto legibles**, uno por tabla.

| Ventaja | Qué significa en la práctica |
|---|---|
| Se puede leer | Abres un archivo y ves las columnas, tipos y consultas. |
| Funciona con Git | Puedes ver exactamente qué cambió entre dos versiones. |
| Se puede revisar | Sirve para auditar el modelo sin abrir Power BI. |

> Para Gobierno de Datos esto es muy útil: el modelo queda **documentado y versionado**
> en lugar de estar escondido dentro de un archivo cerrado.

## Qué es este respaldo (y qué no)

| Incluye | No incluye |
|---|---|
| Tablas y columnas | Páginas del reporte y gráficos |
| Tipos de datos | Medidas DAX (el modelo no tiene ninguna) |
| Relaciones entre tablas | El archivo `raw_data_ERP.csv` (falta en el repositorio) |
| Consultas de Power Query | Los archivos de proyecto `.pbip` / `.pbism` |

Solo está la carpeta `definition/`; **no se puede abrir directamente** en Power BI.
Hay que colocarla dentro de un proyecto PBIP.

## Cómo restaurarlo

1. En Power BI Desktop, activa las funciones de vista previa del **formato de proyecto de Power BI (PBIP)** y del **formato TMDL**, y guarda un reporte como proyecto.
2. Cierra Power BI Desktop.
3. Copia el contenido de `definition/` a la carpeta `<NombreDelProyecto>.SemanticModel/definition/`.
4. Abre el proyecto otra vez.
5. En **Transformar datos**, corrige las rutas de las tres consultas (ver abajo).
6. Pulsa **Actualizar**.

## Atención: las rutas de los CSV

Las tres consultas de Power Query tienen rutas fijas de otro equipo:

```text
C:\Users\Admin\Downloads\Prueba_técnica_-_Data_Governance_&_Data_Quality_Analyst\data\<archivo>.csv
```

Antes de actualizar, cámbialas por la ruta real de la carpeta `data/`.
Una mejor práctica es crear un **parámetro** con la carpeta de datos, así solo se
cambia en un lugar.

## Detalle por carpeta

| Carpeta | Guía |
|---|---|
| `definition/` | [`definition/README.md`](definition/README.md) |
| `definition/tables/` | [`definition/tables/README.md`](definition/tables/README.md) |
| `definition/cultures/` | [`definition/cultures/README.md`](definition/cultures/README.md) |

## Nota

Los archivos `README.md` de estas carpetas son solo documentación. Si Power BI
Desktop se queja de archivos extra al abrir el proyecto, muévelos fuera de
`definition/` o elimínalos en la copia de trabajo.
