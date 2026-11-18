# Carpeta `data/` — las tablas dimensionales del ERP

Aquí están los datos de **clientes** y de **productos** en formato CSV. Son las
"tablas de referencia" que dan contexto a las ventas.

> **Falta un archivo.** El modelo de Power BI también usa `raw_data_ERP.csv` (las
> líneas de factura), pero **no está en este repositorio**. Si lo tienes, colócalo
> en esta carpeta con ese nombre exacto.

## ¿Qué es una dimensión?

Una dimensión responde *quién*, *qué*, *dónde* o *cuándo*. Por sí sola no tiene
números de ventas, pero sirve para **agrupar y filtrar** las ventas. Por ejemplo:
"ventas por categoría de cliente" necesita saber la categoría de cada cliente.

## Archivos

| Archivo | Filas | Columnas | Codificación |
|---|---|---|---|
| `dim_cliente_ERP.csv` | 4,372 | 4 | ASCII (compatible con UTF-8) |
| `dim_prod_ERP.csv` | 4,070 | 3 | ISO-8859 (**no** es UTF-8) |

---

## `dim_cliente_ERP.csv`

**Una fila = un cliente.**

| Columna | Tipo | Ejemplo | Significado |
|---|---|---|---|
| `CustomerID` | Número entero (5 dígitos) | `12347` | Código único del cliente. |
| `NombreCliente` | Texto | `Ella Reyes Morales` | Nombre completo (ficticio). |
| `Pais` | Texto | `Iceland` | País del cliente. |
| `CategoriaCliente` | Texto | `Oro` | Nivel del cliente. |

```csv
CustomerID,NombreCliente,Pais,CategoriaCliente
12346,John Sanchez Rodriguez,United Kingdom,Bronce
12347,Ella Reyes Morales,Iceland,Oro
```

### Datos útiles

| Categoría | Clientes | % |
|---|---|---|
| Plata | 2,087 | 48 % |
| Bronce | 1,119 | 26 % |
| Oro | 903 | 21 % |
| Platino | 263 | 6 % |

- Hay **37 países**. El **Reino Unido concentra 3,950 clientes (90 %)**; le siguen Alemania (95), Francia (87) y España (30).

### Calidad

| Control | Resultado |
|---|---|
| `CustomerID` repetido | 0 |
| Filas duplicadas | 0 |
| Campos vacíos | 0 |
| Espacios sobrantes | 0 |
| Nombres de país a revisar | `EIRE` (3, nombre antiguo de Irlanda), `USA` (4), `RSA` (1), `Unspecified` (4), `European Community` (1) |

---

## `dim_prod_ERP.csv`

**Una fila = un código de producto.**

| Columna | Tipo | Ejemplo | Significado |
|---|---|---|---|
| `StockCode` | Texto | `82613A` | Código único del producto. Es texto porque hay letras. |
| `Descripcion` | Texto | `METAL SIGN,CUPCAKE SINGLE HOOK` | Nombre del producto. |
| `CategoriaProducto` | Texto | `Cocina y Comedor` | Una de 10 categorías. |

```csv
StockCode,Descripcion,CategoriaProducto
10002,INFLATABLE POLITICAL GLOBE,Hogar General
82613A,"METAL SIGN,CUPCAKE SINGLE HOOK",Cocina y Comedor
```

Observa que la segunda descripción lleva **comillas** porque contiene una coma. Sin
ellas, la coma se leería como separador de columnas.

### Categorías de producto

| Categoría | Productos |
|---|---|
| Hogar General | 1,609 |
| Decoracion del Hogar | 647 |
| Cocina y Comedor | 495 |
| Joyeria y Bisuteria | 329 |
| Papeleria y Tarjeteria | 305 |
| Bolsos y Accesorios | 223 |
| Navidad | 168 |
| Textil y Costura | 121 |
| Jardin y Exterior | 93 |
| Juguetes y Ninos | 80 |

### Calidad

| Control | Resultado |
|---|---|
| `StockCode` repetido | 0 |
| Filas duplicadas | 0 |
| Campos vacíos | 0 |
| Descripción `DESCRIPCION NO DISPONIBLE` | **176** (texto de relleno, no un vacío real) |
| Códigos que **no son productos** | **16** (envíos, descuentos, comisiones, vales...) |
| Líneas con comillas (descripción con coma) | 153 |
| Descripciones con minúsculas | 63 |

Los 16 códigos que no son mercancía son `POST`, `DOT` y `C2` (envío), `D`
(descuento), `BANK CHARGES`, `AMAZONFEE` y `CRUK` (comisiones), `B` (deuda
incobrable), `M` / `m` (ajuste manual), `S` (muestras) y `gift_0001_10` a
`gift_0001_50` (vales de regalo). **Los 16 están clasificados como `Hogar
General`**, lo que distorsiona esa categoría.

Los códigos con prefijo `DCGS` (por ejemplo `DCGS0069`, un collar para perros)
**sí son productos**, aunque tengan un formato de código distinto. Varios de ellos
tienen la descripción de relleno o el texto `ebay`.

---

## Cómo se usan en Power BI

Cada CSV se convierte en una tabla del modelo mediante una consulta de Power Query
(`Csv.Document`). Las consultas guardadas apuntan a una carpeta de otro equipo:

```text
C:\Users\Admin\Downloads\Prueba_técnica_-_Data_Governance_&_Data_Quality_Analyst\data\...
```

Hay que cambiar esa ruta por la tuya al restaurar el modelo. Más detalle en
[`../backup_modelo_original/README.md`](../backup_modelo_original/README.md).

## Cómo revisarlos rápido con Python

```python
import pandas as pd

clientes  = pd.read_csv("dim_cliente_ERP.csv")
productos = pd.read_csv("dim_prod_ERP.csv", encoding="cp1252")   # no es UTF-8

print(clientes["CustomerID"].is_unique)          # True
print(clientes["CategoriaCliente"].value_counts())
print((productos["Descripcion"] == "DESCRIPCION NO DISPONIBLE").sum())   # 176
```
