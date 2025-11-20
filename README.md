# 📦 EDIWIN Parser Demo — Eurofiel & El Corte Inglés

Parser de PDFs EDIWIN con interfaz web en Streamlit.  
Convierte pedidos de **Eurofiel** y **El Corte Inglés (ECI)** en:

- Tablas limpias para análisis.
- Excels formateados automáticamente.
- Carpetas por modelo con su propio PDF filtrado.
- TXT EDI repartidos por modelo (LINPED, CABPED, etc.).

> Proyecto DEMO, con datos ficticios y rutas locales, pensado como ejemplo realista de automatización para logística/retail. 

---

## 🧩 Problema que resuelve

En muchos entornos de logística y compras se reciben **pedidos en PDF generados por EDIWIN**.  
El flujo típico suele ser:

- Abrir el PDF a mano.
- Buscar modelos, colores, tallas.
- Copiar y pegar a Excel.
- Calcular totales.
- Separar información por modelo para enviarla a fábrica o a otros departamentos.
- Gestionar también los ficheros **TXT EDI** (LINPED, CABPED, LOCPED, OBSPED, OBSLPED).

Este proceso es **lento, repetitivo y propenso a errores**.

Este proyecto automatiza todo ese trabajo:

- **Lee el PDF completo**.
- **Interpreta las líneas de detalle** (modelo, color, talla, unidades, precio).
- **Calcula totales y resumen por modelo / color**.
- **Genera Excels y PDFs organizados por modelo**.
- **Reparte los TXT EDIWIN en carpetas por modelo**, listos para integrarse en otros sistemas. 

---

## ✨ Funcionalidades principales

### 1. Interfaz web en Streamlit

Desde la web (local):

- Seleccionas el cliente: **Eurofiel** o **El Corte Inglés**.
- Subes un PDF EDIWIN.
- Ves una **vista previa** coloreada por modelo.
- Descargas:
  - Excel global.
  - CSV.
  - Carpetas por modelo (con PDF y Excel).
- (Opcional) Subes los **TXT EDIWIN** y los reparte por modelo en las mismas carpetas. 

---

### 2. Parser Eurofiel

- Divide el PDF en pedidos usando el patrón `Nº Pedido :`.
- Extrae de cada bloque:
  - Tipo de operación: `PEDIDO`, `REEMPLAZO`, `ANULACIÓN`.  
  - Nº de pedido, fecha de entrega, país, descripción.
  - Modelo (código proveedor), patrón (código cliente).
  - Precio neto.
  - Unidades totales.
  - Unidades por talla (`XXS, XS, S, M, L, XL, XXL, 34, 36, ... 48`). 
- Detecta automáticamente las líneas de detalle con expresiones regulares y separa:
  - `EAN13`
  - Código proveedor: `MODELO/COLOR/TALLA`
  - Código cliente: `PATRON/COLOR/TALLA`
  - Cantidad, precios bruto/neto. 
- Agrupa la información en un `DataFrame` de pandas, listo para exportar.

---

### 3. Parser El Corte Inglés (ECI)

- Procesa página a página con `pdfplumber`.
- Detecta el tipo de documento por texto: `Pedido`, `Reposicion`, `Anulacion Pedido`.
- Extrae de cada página:
  - Nº de pedido, departamento, fecha de entrega.
  - Sucursal de entrega.
- Localiza las líneas de detalle (nº + EAN13…) y obtiene:
  - Descripción completa (incluyendo líneas partidas).
  - Modelo.
  - Color.
  - Código de talla (por ejemplo `003` en `FLOR003`).
  - Talla humana usando un mapa de equivalencias:  
    `001→XXS, 002→XS, 003→S, 004→M, ... 034→34, 036→36, ...`. 
- Agrupa por (modelo, color, sucursal…) y pivota las tallas a columnas, generando:
`TIPO | N_PEDIDO | MODELO | COLOR | ... | S | M | L | 34 | 36 | ... | TOTAL_UNIDADES`


### 4. Excels formateados automáticamente

- Al exportar a Excel:

   - Bordes finos en todas las celdas.

   - Cabecera en amarillo + negrita (estilo corporativo).

   - Fila TOTAL resaltada en amarillo + negrita:

      - Total de pedidos.

      - Total de unidades.

      - Totales por talla.

   - Oculta automáticamente columnas de tallas que estén todo a 0 (más limpio para el usuario).

   - Dos hojas de resumen en el caso de ECI:

      - Resumen modelo+color.

      - Resumen modelo.

### 5. Carpetas y PDFs por modelo

Para ambos proveedores:

   - Crea una carpeta por modelo (y patrón en el caso de Eurofiel).

   - Filtra el PDF original y guarda dentro:

      - Un PDF con sólo las páginas relevantes para ese modelo.

      - Un Excel con los datos filtrados y totales.

Para esto se construye un mapa modelo → páginas buscando el código de modelo dentro del texto de cada página y arrastrando páginas sin detalle al último modelo detectado.

### 6. TXT EDIWIN repartidos por modelo

La aplicación permite subir los ficheros TXT generados por EDIWIN:

   - CABPED_*.TXT

   - LINPED_*.TXT

   - LOCPED_*.TXT

   - OBSPED_*.TXT

   - OBSLPED_*.TXT

Y hace lo siguiente:

   - A partir de LINPED detecta para cada línea:

      - Nº de pedido interno.

      - Modelo (y patrón en Eurofiel).

   - Construye un mapa:

      - Eurofiel → pedido_int → {MODELO_PATRON}

      - ECI → pedido → {MODELO}

   - Crea en cada carpeta de modelo una copia filtrada de los TXT con solo los registros que afectan a ese modelo.

Esto deja un set de ficheros EDI por modelo, muy útil para depuración o integraciones posteriores.

### 7. Arquitectura del proyecto
ediwin-parser-demo/
│
├── src/
│   └── app.py              # Lógica de parsers + interfaz Streamlit
│
├── samples/                # Datos DEMO (PDF + TXT ficticios)
│   ├── eci/
│   ├── eci_txt/
│   ├── eurofiel/
│   ├── eurofiel_txt/
│   └── README.md
│
├── output/                 # Carpeta generada automáticamente por la app
│   ├── eurofiel/
│   └── eci/
│
├── requirements.txt
├── .gitignore
└── README.md


En la demo, todas las rutas son locales (output/...). No hay rutas de red ni datos sensibles.

### 8. Stack tecnológico

   - Python

   - Streamlit – interfaz web y carga de ficheros.

   - pandas – modelado de datos, agrupaciones, pivots.

   - pdfplumber – extracción de texto desde PDF.

   - pypdf – filtrado y escritura de PDFs por modelo.

   - openpyxl – estilo y formato avanzado en Excel (bordes, colores, totales).

   - Expresiones regulares (re) para parseo de líneas y detección de patrones (modelos, tallas, etc.).

### 9. Puesta en marcha
1. Clonar el repositorio
   - git clone https://github.com/tuusuario/ediwin-parser-demo.git
   - cd ediwin-parser-demo

2. Crear entorno virtual (recomendado)
   - python -m venv .venv
   # Linux/macOS
   - source .venv/bin/activate       
   # o Windows
   - .\.venv\Scripts\activate        

3. Instalar dependencias
   - pip install -r requirements.txt

4. Lanzar la aplicación
   - streamlit run src/app.py


Se abrirá la app en el navegador (normalmente en http://localhost:8501).

### 10. Uso de la aplicación
A. Flujo Eurofiel

   1. En la web, selecciona Cliente → Eurofiel.

   2. Sube un PDF de ejemplo de la carpeta samples/eurofiel/.

   3. La app mostrará:

      - Tabla de pedidos con columnas por talla.

      - Total de unidades por modelo.

   4. Opciones de descarga:

      - Excel con:

         - Hoja Pedidos (detalle).

         - Hoja Resumen por modelo con fila TOTAL.

      - CSV con columnas de tallas activas.

   5. Botón: “Crear carpetas y PDFs por modelo en Eurofiel”

   - Genera en output/eurofiel/:

      - Carpeta por modelo+patrón.

      - PDF filtrado.

      - Excel filtrado con totales.

   6. Opcional: subir TXT EDIWIN (LINPED, CABPED, LOCPED, OBSPED, OBSLPED) desde samples/eurofiel_txt/ y pulsar
   - “Repartir TXT EDIWIN por modelo en carpetas EUROFIEL”.

B. Flujo El Corte Inglés (ECI)

   1. Selecciona Cliente → El Corte Inglés.

   2. Sube el PDF de ejemplo ECI_DEMO_PARSER_FINAL.pdf de samples/eci/.

   3. Verás:

      - Vista previa con tallas pivotadas por columnas.

      - Suma de unidades por talla y TOTAL_UNIDADES.

   4. Resúmenes:

      - Resumen por MODELO + COLOR.

      - Resumen por MODELO con totales.

   5. Descargas:

      - Excel con:

         - Hoja Pedidos.

         - Hoja Resumen modelo+color.

         - Hoja Resumen modelo.

      - CSV sin tallas vacías.

   6. Botón: “Crear carpetas y PDFs por modelo en ECI”

      - Genera en output/eci/ una carpeta por modelo con:

         - PDF filtrado.

         - Excel filtrado con fila TOTAL.

   7. Opcional: subir TXT EDIWIN de ejemplo desde samples/eci_txt/ y pulsar
   - “Repartir TXT EDIWIN por modelo en carpetas ECI”.

### 11. Detalles técnicos interesantes

Este proyecto demuestra:

   - Parseo robusto de PDFs con estructuras semi–fijas (EDIWIN).

   - Uso intensivo de expresiones regulares para extraer:

      - Números de pedido, fechas, departamentos, sucursales.

      - Códigos EAN, códigos internos, modelos, patrones.

      - Tallas incrustadas al final del color (FLOR003 → FLOR + 003).

   - Conversión de datos crudos en tablas analíticas con pandas, incluyendo:

      - groupby por modelo/color.

      - pivot_table para tallas como columnas.

      - Cálculo de sumatorios y totales globales.

   - Generación de Excels de nivel usuario final (no solo datos):

      - Cabeceras y totales destacados.

      - Bordes en todas las celdas.

      - Ocultado de tallas sin unidades.

   - Trabajo con TXT EDI reales en codificación latin-1, respetando saltos de línea y estructura.

### 12. Limitaciones de la demo

   - Los PDFs y TXT incluidos en samples/ son ficticios, diseñados para mostrar el funcionamiento sin exponer datos reales.

   - El parser asume un formato EDIWIN similar al de los ejemplos. Si el layout del PDF cambia mucho, habría que ajustar las expresiones regulares.

   - No hay integración directa con sistemas de terceros (ERP, redes, etc.).
   Toda la salida se genera en la carpeta local output/.

### 13. Próximas mejoras

Algunas ideas de evolución natural del proyecto:

   - Más proveedores / formatos EDI.

   - Test unitarios para los parsers (Eurofiel/ECI).

   - Imagen Docker para despliegue rápido.

   - Deploy en Streamlit Cloud u otra plataforma.

   - Configuración de mapeos de tallas y reglas via YAML/JSON (en vez de estar embebido en código).

### 14. 👤 Autor

Aitor Susperregui (@elvasco.x)

Desarrollador en formación, con background real en logística y tratamiento de pedidos.
Este proyecto nace de una necesidad real de automatizar tareas repetitivas en almacén
y se ha convertido en un ejemplo práctico de:

   - Parseo de PDFs.

   - Tratamiento de datos con Python.

   - Creación de herramientas internas con Streamlit.

Si quieres contactar conmigo para hablar de desarrollo, automatización o trabajo con EDI/PDF:

   -  Email: tsuspe@icloud.com

   - Telefono: +34 682 714 237 (WhatsApp / Telegram)

   -  Instagram / Marca personal: @elvasco.x