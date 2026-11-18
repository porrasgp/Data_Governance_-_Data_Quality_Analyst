# Carpeta `cultures/` — idioma y formato regional

Esta carpeta tiene un solo archivo: `en-US.tmdl`.

```text
cultureInfo en-US
```

## ¿Qué es una "cultura"?

La cultura define cómo se escriben y se interpretan los **números, las fechas y los
textos** del modelo.

| | `en-US` (este modelo) | `es-CR` (Costa Rica) |
|---|---|---|
| Separador decimal | punto: `1,234.50` | coma: `1 234,50` |
| Formato de fecha | mes/día/año: `12/31/2026` | día/mes/año: `31/12/2026` |

Este modelo usa inglés de EE. UU. (`en-US`), en coincidencia con las columnas del
ERP, que están en inglés (`InvoiceNo`, `UnitPrice`, `Country`...).

## Por qué importa en calidad de datos

Si la cultura no coincide con el formato de los datos, la lectura puede fallar **sin
avisar**. Por ejemplo, `03/04/2026` se leería como el 4 de marzo con `en-US` y
como el 3 de abril con `es-CR`. Al cambiar de cultura, valida siempre las
fechas y los importes.

## ¿Hay que cambiarla?

No, mientras los datos de origen estén en formato inglés. Solo habría que revisarla
si se cargan datos con fechas o decimales en formato regional distinto.
