# 📦 EDIWIN Parser Demo — Eurofiel & El Corte Inglés

Parser de PDFs EDIWIN con interfaz web en Streamlit.  
Convierte pedidos de **Eurofiel** y **El Corte Inglés (ECI)** en:

- Tablas limpias para análisis.
- Excels formateados automáticamente.
- Carpetas por modelo con su propio PDF filtrado.
- TXT EDI repartidos por modelo (LINPED, CABPED, etc.).

> Proyecto DEMO con datos ficticios y rutas locales, pensado como un ejemplo realista de automatización para logística/retail.

---

## 🧩 Problema que resuelve

En muchos entornos de logística y compras se reciben **pedidos en PDF generados por EDIWIN**.  
El flujo típico suele ser:

- Abrir el PDF a mano.
- Buscar modelos, colores, tallas.
- Copiar y pegar a Excel.
- Calcular totales.
- Separar información por modelo para enviarla a fábrica o a otros departamentos.
- Gestionar los ficheros **TXT EDI** (LINPED, CABPED, LOCPED, OBSPED, OBSLPED).

Este proceso es **lento, repetitivo y propenso a errores**.

Este proyecto automatiza todo ese trabajo:

- **Lee el PDF completo.**
- **Interpreta las líneas de detalle** (modelo, color, talla, unidades, precio).
- **Calcula totales y resumen por modelo/color.**
- **Genera Excels y PDFs organizados por modelo.**
- **Reparte los TXT EDIWIN por modelo**, listos para integrarse en otros sistemas.

---

## ✨ Funcionalidades principales

### 1. Interfaz web en Streamlit

Desde la interfaz (local):

- Selección del cliente: **Eurofiel** o **El Corte Inglés**.
- Subida de PDF EDIWIN.
- Vista previa coloreada por modelo.
- Descargas disponibles:
  - Excel global.
  - CSV.
  - Carpetas por modelo (PDF + Excel).
- Opcional: subida de TXT EDIWIN para repartirlos automáticamente por modelo.

---

### 2. Parser Eurofiel

- Divide el PDF por bloques basados en `Nº Pedido :`.
- Extrae:
  - Tipo de operación: `PEDIDO`, `REEMPLAZO`, `ANULACIÓN`.
  - Nº pedido, fecha de entrega, país, descripción.
  - Modelo (proveedor), patrón (cliente).
  - Precio neto.
  - Unidades totales y por talla.
- Detecta líneas de detalle usando regex:
  - EAN13
  - MODELO/COLOR/TALLA
  - PATRÓN/COLOR/TALLA
  - Cantidad, precio bruto y precio neto
- Agrupa en `DataFrame` listo para exportación.

---

### 3. Parser El Corte Inglés (ECI)

- Procesa página a página con `pdfplumber`.
- Clasifica por tipo de documento:
  - `Pedido`
  - `Reposicion`
  - `Anulacion Pedido`
- Extrae:
  - Nº pedido
  - Departamento
  - Fecha de entrega
  - Sucursal de entrega
- Identifica líneas de detalle:
  - Descripción multipartida
  - Modelo
  - Color
  - Código de talla (`003`, `004`, `034`…)
  - Talla humana usando equivalencias
- Pivota tallas hacia columnas:  
  `TIPO | N_PEDIDO | MODELO | COLOR | S | M | L | 34 | 36 | … | TOTAL_UNIDADES`

---

### 4. Excels formateados automáticamente

Al generar el Excel:

- Bordes finos en todas las celdas.
- Cabecera amarilla + negrita (estilo corporativo).
- Fila **TOTAL** destacada:
  - Total de pedidos.
  - Total de unidades.
  - Totales por talla.
- Oculta columnas de tallas sin unidades.
- Para ECI incluye:
  - **Hoja Resumen modelo+color**
  - **Hoja Resumen modelo**

---

### 5. Carpetas y PDFs por modelo

Para ambos proveedores:

- Crea una carpeta por modelo (o modelo+patrón en Eurofiel).
- Filtra el PDF original y guarda:
  - PDF únicamente con las páginas relevantes.
  - Excel filtrado con totales.
- Maneja páginas sin detalle arrastrándolas al último modelo detectado.

---

### 6. TXT EDIWIN repartidos por modelo

Permite subir:

- `CABPED_*.TXT`
- `LINPED_*.TXT`
- `LOCPED_*.TXT`
- `OBSPED_*.TXT`
- `OBSLPED_*.TXT`

Procesa:

- A partir de **LINPED** detecta:
  - Pedido interno
  - Modelo
  - (Patrón en Eurofiel)
- Construye el mapa:
  - **Eurofiel → pedido_int → {MODELO_PATRON}**
  - **ECI → pedido → {MODELO}**
- Copia cada TXT filtrado dentro de la carpeta de cada modelo.

Útil para depuración o integraciones posteriores.

---

## 🧱 Arquitectura del proyecto

```text
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
```

> En esta versión DEMO, todas las rutas son locales (`output/...`). No hay datos reales ni rutas de red.

---

## 🛠 Stack tecnológico

- **Python**
- **Streamlit** – interfaz web
- **pandas** – tratamiento de datos
- **pdfplumber** – extracción de texto de PDF
- **pypdf** – filtrado y creación de PDFs por modelo
- **openpyxl** – Excel con formato avanzado
- **regex** – detección de patrones complejos

---

## 🚀 Puesta en marcha

### 1. Clonar el repositorio

```bash
git clone https://github.com/tuusuario/ediwin-parser-demo.git
cd ediwin-parser-demo
```

### 2. Crear entorno virtual (recomendado)

```bash
python -m venv .venv
source .venv/bin/activate       # Linux/macOS
.\.venv\Scriptsctivate        # Windows
```

### 3. Instalar dependencias

```bash
pip install -r requirements.txt
```

### 4. Lanzar la aplicación

```bash
streamlit run src/app.py
```

Abrirá la app en `http://localhost:8501`.

---

## 📥 Uso de la aplicación

### A. Eurofiel

1. Seleccionar **Cliente → Eurofiel**.
2. Subir PDF desde `samples/eurofiel/`.
3. Vista previa:
   - Tabla por talla
   - Totales por modelo  
4. Descargas:
   - Excel (Pedidos + Resumen)
   - CSV  
5. Botón: **Crear carpetas y PDFs por modelo**  
6. Opcional: subir TXT de ejemplo desde `samples/eurofiel_txt/`.

---

### B. El Corte Inglés (ECI)

1. Seleccionar **Cliente → El Corte Inglés**.
2. Subir `ECI_DEMO_PARSER_FINAL.pdf`.
3. Vista previa:
   - Tallas pivotadas
   - Totales por modelo
4. Descargas:
   - Excel (Pedidos + Resumen modelo+color + Resumen modelo)
   - CSV  
5. Botón: **Crear carpetas y PDFs por modelo**  
6. Opcional: TXT desde `samples/eci_txt/`.

---

## 🔬 Detalles técnicos interesantes

El proyecto demuestra:

- Parseo robusto de PDFs con estructuras semi-fijas.
- Uso intensivo de **regex**: modelos, tallas, códigos EAN, patrones EDI.
- Transformación avanzada con pandas:
  - `groupby`
  - `pivot_table`
  - sumatorios y totales
- Excel con formato profesional (bordes, estilos).
- Gestión de TXT EDI reales (`latin-1`), respetando estructura original.
- Web app funcional, ligera y usable.

---

## ⚠️ Limitaciones de la demo

- Los PDFs y TXT incluidos en `samples/` son **ficticios**.
- El parser depende del layout estándar de EDIWIN.
- No hay integración con ERP ni rutas de red.
- La salida siempre es local en `output/`.

---

## 🧭 Próximas mejoras

- Más proveedores/formatos EDI.
- Test unitarios para los parsers.
- Imagen Docker para despliegue rápido.
- Versión cloud (Streamlit Cloud / HuggingFace Spaces).
- Configuración de reglas vía YAML/JSON.

---

## 👤 Autor

**Aitor Susperregui — @elvasco.x**

Desarrollador en formación con experiencia real en logística y tratamiento de pedidos.  
Este proyecto nace de la necesidad de automatizar procesos repetitivos en almacén y se convierte en un caso práctico de:

- Parseo de PDFs
- Procesamiento de datos con Python
- Interfaces internas con Streamlit

Contacto:

- 📧 **tsuspe@icloud.com**
- 📱 **+34 682 714 237** (WhatsApp / Telegram)
- 🖤 Instagram / Marca personal: **@elvasco.x**
