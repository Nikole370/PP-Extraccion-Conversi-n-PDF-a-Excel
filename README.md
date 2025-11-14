## 📑 Script para Extracción y Conversión de **Listados CPE SUNAT (BVE)** de PDF a Excel/CSV 📄

Este script está diseñado específicamente para procesar los **listados de Comprobantes de Pago Electrónico (CPE) de Boletas de Venta Electrónicas (BVE)** emitidos por la **Superintendencia Nacional de Aduanas y de Administración Tributaria (SUNAT)**, obtenidos a través de su portal web de consultas.

Se extraen datos estructurados (CPE) de archivos PDF, utilizando múltiples estrategias (extracción de tablas y *fallback* basado en expresiones regulares) para unificar y limpiar los resultados en un formato estándar exportable a Excel y CSV.

---

### 📥 Proceso y Requisitos de Entrada Específicos

El script requiere que el usuario cargue un archivo **PDF** generado tras realizar una consulta en el portal de SUNAT.

* **Paso Previo Requerido (SUNAT):**
    1.  Acceder al portal **SUNAT SOL Operaciones en Línea**.
    2.  Navegar a la opción o apartado **Consultas Boleta de Venta, Nota de Crédito y Débito Electrónicas**.
    3.  Filtrar la búsqueda por un rango de fechas específico (por ejemplo, desde el **inicio del primer día de setiembre** hasta el **último día de setiembre**).
    4.  Generar el reporte en formato **PDF** y guardarlo localmente.
* **Ejecución del Script:**
    1.  Ejecutar la celda de **librerías** en el entorno del script (por ejemplo, Google Colab).
    2.  Ejecutar la siguiente celda interactiva, la cual solicitará la carga del archivo PDF generado en el paso anterior.
* **Contenido esperado del PDF (Listado SUNAT):** Listados con las siguientes piezas de información en líneas:
    * **Número de CPE** (patrón: `EBxx - xxxx`).
    * **Receptor** (nombre o RUC).
    * **Importe Total** (formato monetario, ej. `S/150.00`).
    * **Fecha de Emisión** (formato: `dd/mm/yyyy`).

> 💡 **Nota:** Si bien el script fue inicialmente desarrollado para este escenario específico de listados BVE de SUNAT, su robustez de extracción puede hacerlo útil para otros listados o reportes de CPE en formato PDF con una estructura similar.
---

### 📤 Salida Generada

El script genera un único archivo unificado que contiene la información estandarizada de todos los PDFs procesados.

* **Nombres de archivos de salida:** Utilizan el nombre base del **PDF de SUNAT** procesado:
    1.  **CSV:** `/content/{nombre_archivo_base}_listado.csv`
    2.  **XLSX:** `/content/{nombre_archivo_base}_listado.xlsx`
* **Estructura de la tabla de salida (Orden de Columnas):**

| Columna | Descripción | Formato |
| :--- | :--- | :--- |
| **`__archivo__`** | Nombre del PDF original de donde proviene el registro. | `string` |
| **`Nro_CPE`** | Número del comprobante. | `string` |
| **`Receptor`** | Nombre o RUC del receptor. | `string` |
| **`Importe_Total`** | Monto total del comprobante. | `float` (numérico) |
| **`Fecha_Emision`** | Fecha de emisión. | `DD/MM/YYYY` (string) |

---

### 🛠️ Descripción de Librerías

| Librería | Propósito |
| :--- | :--- |
| **`pandas`** | Manejo de datos estructurados (DataFrames), limpieza, unificación y exportación a formatos CSV/Excel. |
| **`pdfplumber`** | Herramienta principal para la apertura de PDFs, extracción de texto y detección de tablas. |
| **`re`** | Módulo para la implementación de expresiones regulares, esencial para el mecanismo de *fallback* de extracción de datos por línea. |
| **`unicodedata`** | Se usa para la limpieza de texto, específicamente para la normalización y eliminación de acentos en los nombres de columnas. |
| **`google.colab.files`** | Permite la subida interactiva del archivo PDF de SUNAT en un entorno de Google Colab. |

---

### 💡 Función Central: `clean_df(df, source_name)`

Esta función es la encargada de la **estandarización y limpieza final** del DataFrame, crucial para manejar las variaciones de formato en los listados de SUNAT. Su rol es asegurar la integridad y el formato de los datos extraídos antes de la exportación.

* Aplica un **renombrado inteligente** de las columnas extraídas (ej. "Nro Comprobante" se convierte en "Nro\_CPE").
* Utiliza **heurísticas** (patrones de texto o posición) para identificar columnas clave faltantes (`Receptor`, `Importe_Total`, `Fecha_Emision`).
* **Limpia y convierte** `Importe_Total` a un valor numérico (`float`).
* Convierte `Fecha_Emision` a tipo `datetime` para validación y asegura que el formato final de exportación sea **DD/MM/YYYY**.

---