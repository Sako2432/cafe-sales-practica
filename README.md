# Limpieza de datos — Cafe Sales (Dirty Data for Cleaning Training)

Actividad individual de la materia **Análisis y visualización de la información** (Universidad de Guadalajara, CUCEI).
**Autor:** Jorge Isaac Quintero Carreón — 219515362

## Objetivo

Aplicar técnicas de limpieza y transformación de datos con Python sobre `dirty_cafe_sales.csv`, un dataset diseñado intencionalmente con problemas de calidad, y exportar una versión limpia lista para análisis.

## Contenido del repositorio

| Archivo | Descripción |
|---|---|
| `dirty_cafe_sales.csv` | Dataset original ("sucio"), sin modificar. |
| `limpieza_cafe_sales.ipynb` | Notebook con todo el proceso: carga, inspección, identificación de problemas, limpieza, validación y exportación. |
| `cafe_sales_clean.csv` | Versión limpia del dataset (9,974 registros). |
| `tabla_resumen.csv` | Tabla resumen de problemas encontrados y decisiones tomadas. |
| `README.md` | Este archivo. |

## Resumen del procedimiento

1. **Inspección inicial**: el dataset original tiene 10,000 filas y 8 columnas, todas cargadas como texto. Los valores faltantes aparecen de tres formas distintas: celdas vacías (`NaN`) y los textos literales `"ERROR"` y `"UNKNOWN"` usados como marcadores de dato corrupto o desconocido.
2. **Estandarización de faltantes**: `"ERROR"` y `"UNKNOWN"` se reemplazan por `NaN` en todas las columnas, para poder tratarlos de forma uniforme.
3. **Corrección de tipos de datos**: `Quantity`, `Price Per Unit` y `Total Spent` se convierten a numérico; `Transaction Date` se convierte a fecha.
4. **Reconstrucción de valores faltantes usando relaciones reales de los datos** (en vez de imputar con la media/moda de forma ciega):
   - Cada `Item` tiene un `Price Per Unit` fijo → se construyó un mapa Item→Precio para rellenar precios faltantes, y precios que identifican a un único producto para rellenar `Item` faltante.
   - `Total Spent = Quantity × Price Per Unit` se cumple en el 100 % de las filas completas → se usó para reconstruir el campo faltante cuando los otros dos eran conocidos.
5. **Eliminación de filas irrecuperables**: 26 filas (0.26 %) no tenían suficiente información cruzada para determinar el monto de la venta y se eliminaron.
6. **Categoría "Desconocido"** para lo que no se pudo reconstruir ni tenía sentido imputar: `Item` en casos de precio ambiguo (495 filas), y `Payment Method` / `Location`, que tenían ~32 % y ~40 % de datos faltantes respectivamente — imputar con la moda habría introducido un sesgo fuerte, así que se prefirió declarar el dato como desconocido en vez de inventarlo.
7. **`Transaction Date`** faltante (460 filas) se dejó como valor nulo (`NaT`); no existe otra columna de la que se pueda derivar la fecha, así que no se imputó.
8. **Duplicados y valores atípicos**: se revisaron explícitamente y no se encontraron (no había filas duplicadas ni valores de `Quantity`/`Price Per Unit` fuera de un rango razonable), por lo que no se aplicó ninguna acción sobre ellos.
9. **Validación final**: se comprobó que `Total Spent = Quantity × Price Per Unit` se cumple en el 100 % de las 9,974 filas finales antes de exportar.

## Tabla resumen

| Problema encontrado | Registros afectados | Acción realizada | Justificación |
|---|---|---|---|
| Valores faltantes (NaN/ERROR/UNKNOWN) en Item, Quantity, Price Per Unit, Total Spent | 2,483 | Reconstrucción cruzada vía Item↔Precio y Total=Quantity×Precio; lo irrecuperable se marcó "Desconocido" o se eliminó (26 filas) | Preserva información real en vez de inventarla con la media |
| Valores faltantes en Payment Method y Location | 7,139 | Se mantienen las filas, categoría marcada como "Desconocido" | Imputar con la moda sesgaría fuertemente el análisis dado el alto % de faltantes |
| Valores faltantes en Transaction Date | 460 | Se dejan como NaT, sin imputar | No existe otra columna de la que derivar la fecha |
| Duplicados | 0 | Ninguna (verificado, no existían) | No aplicar limpieza innecesaria sobre datos válidos |
| Tipos de datos incorrectos (4 columnas cargadas como texto) | 10,000 | Conversión con `pd.to_numeric()` / `pd.to_datetime()` tras limpiar marcadores | Necesario para poder operar y validar los datos numéricamente |
| Valores atípicos | 0 | Ninguna (verificado, rangos razonables) | No hay evidencia de valores anómalos que justifique eliminarlos |

*(Tabla completa con más detalle en `tabla_resumen.csv`.)*

## Cómo reproducir

```bash
pip install pandas numpy jupyter
jupyter nbconvert --to notebook --execute --inplace limpieza_cafe_sales.ipynb
```

Esto vuelve a generar `cafe_sales_clean.csv` y `tabla_resumen.csv` a partir de `dirty_cafe_sales.csv`.

## Declaración de IA generativa y tecnologías asistidas por IA

*(Sección a completar por el autor conforme al formato de entrega del curso.)*
