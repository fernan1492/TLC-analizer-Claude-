# 🔬 TLC Analyzer

**Herramienta para análisis cuantitativo de imágenes de Cromatografía en Capa Delgada (TLC) directamente en el navegador.**

Sin instalación. Sin servidor. Sin conexión a internet. Abrí el archivo HTML en cualquier navegador moderno y empezá a analizar.

> 📄 [English version → README.md](README.md)

---

## ¿Qué hace?

TLC Analyzer te permite cargar una foto de tu placa de TLC y:

- Marcar el **frente del solvente** y la **línea de origen** para calcular valores de Rf
- Definir **calles verticales** y nombrarlas (ej: "Control", "Muestra 1", "Fracción 3")
- **Detectar manchas automáticamente** usando un algoritmo sobre la oscuridad de los píxeles, o dibujarlas manualmente
- Medir la **densidad óptica integrada** (oscuridad por píxel × área) como indicador de cantidad relativa de compuesto
- Comparar **cantidades relativas** del mismo compuesto entre diferentes calles
- Visualizar **densitogramas por calle** (perfil de densidad óptica desde el origen hasta el frente)
- Exportar todos los datos a **CSV**

---

## Inicio rápido

1. **Descargá** `tlc-analyzer.html`
2. **Abrilo** en Chrome, Firefox, Edge o Safari
3. Seguí el flujo de 6 pasos:

```
① Cargar imagen → ② Marcar frente → ③ Marcar origen → ④ Agregar calles → ⑤ Detectar manchas → ⑥ Comparar
```

No requiere npm, Python ni ninguna instalación adicional.

---

## Tutorial paso a paso

### Paso 1 — Cargar la imagen de TLC

Arrastrá y soltá tu foto de TLC sobre la zona de carga, o hacé clic para explorar archivos.

- Compatible con **JPG, PNG, TIFF** y cualquier formato de imagen estándar
- La imagen se procesa completamente en tu navegador; nada se sube a ningún servidor
- Imágenes de mayor resolución dan mediciones de densidad óptica más precisas

> **Consejo:** Buena iluminación y fondo oscuro en la placa dan mejores resultados. Evitá reflejos y sombras. Lo ideal es escanear la placa con un transiluminador UV o un escáner de cama plana en lugar de una foto de cámara.

---

### Paso 2 — Marcar el frente del solvente

1. Seleccioná el modo **▲ Frente** (activo por defecto)
2. Hacé clic en la imagen sobre la **línea del frente del solvente** (el punto más alto al que llegó el solvente)

Aparecerá una línea verde punteada etiquetada `FRENTE`.

---

### Paso 3 — Marcar el origen

1. Seleccioná el modo **▼ Origen**
2. Hacé clic sobre la **línea de siembra** (donde aplicaste las muestras)

Aparecerá una línea naranja punteada etiquetada `ORIGEN`.

> **¿Por qué es necesario?** Rf = (distancia recorrida por la mancha) / (distancia recorrida por el solvente), ambas medidas desde el origen. Sin estas dos líneas, el Rf no puede calcularse.

---

### Paso 4 — Definir las calles

1. Seleccioná el modo **| Calles**
2. Hacé clic una vez en la imagen **en el centro de cada calle** (columna vertical)
3. Cada calle recibe un color único y un nombre por defecto ("Calle 1", "Calle 2", etc.)
4. **Renombrá las calles** en el panel izquierdo haciendo clic en el campo de nombre — usá nombres descriptivos como `Control`, `Muestra A`, `Std 10µg`

Las calles se ordenan automáticamente por posición horizontal. Podés eliminar calles individuales con el botón ✕.

> **Consejo:** Ajustá el slider **Ancho de calle** antes de la detección automática para que coincida con el ancho físico de tus calles. Un valor más grande captura más señal de la calle; demasiado ancho y vas a capturar señal de calles vecinas.

---

### Paso 5 — Detectar manchas

#### Opción A: Detección automática (recomendada)

1. Ajustá los sliders:
   - **Threshold** — qué tan oscuro debe ser un píxel para considerarse parte de una mancha. Aumentá si hay falsos positivos (ruido de fondo detectado); disminuí si se pierden manchas reales.
   - **Min área** — tamaño mínimo de mancha en píxeles. Aumentá para filtrar artefactos pequeños.
2. Hacé clic en **⚡ Auto-detectar**

El algoritmo usa **flood-fill (BFS)**: encuentra regiones conectadas de píxeles oscuros dentro de la columna de cada calle, acotadas entre el origen y el frente.

#### Opción B: Dibujo manual

1. Seleccioná el modo **● Manchas**
2. Hacé clic y arrastrá sobre la imagen para dibujar un rectángulo alrededor de una mancha
3. La mancha se asigna automáticamente a la calle más cercana

Podés combinar ambos métodos: detectar automáticamente primero y luego agregar manchas perdidas o corregir falsos positivos manualmente.

---

### Paso 6 — Comparar y analizar

Una vez detectadas las manchas, el panel derecho muestra tres vistas:

#### 📊 Pestaña Cantidades

Esta es la comparación cuantitativa central. Para cada grupo de manchas con valores de Rf similares (dentro de ±0.07), un gráfico de barras muestra el **volumen óptico relativo** entre todas las calles.

El **volumen óptico** se calcula como:

```
Volumen = Σ (255 − valor_gris_del_píxel)  para cada píxel dentro del bounding box de la mancha
```

- Un píxel completamente negro aporta 255; un píxel blanco aporta 0
- Manchas más grandes y/o más oscuras tienen mayor volumen
- Dentro de cada grupo de Rf, las barras se normalizan al máximo (100%)

En la parte superior también se muestra la **carga óptica total por calle** (suma de todas las manchas), útil para verificar consistencia en la siembra.

> **Nota metodológica:** El volumen óptico es proporcional a la concentración solo en la zona lineal de respuesta del colorante. Para cuantificación absoluta, incluí una serie de diluciones de un estándar conocido en la misma placa y construí una curva de calibración.

#### 📈 Pestaña Densitograma

Para cada calle se muestra un gráfico de densidad óptica promedio vs. posición (de origen a frente). Es análogo a la traza de un cromatograma. Las posiciones de las manchas detectadas se marcan con líneas verticales punteadas.

Útil para:
- Verificar la precisión de la detección de manchas
- Identificar manchas solapadas o mal resueltas
- Comparar anchos de pico entre calles

#### 🗃 Pestaña Tabla

Tabla de datos completa con:

| Columna | Descripción |
|---------|-------------|
| Calle | Nombre de la calle |
| Rf | Factor de retención (0 = origen, 1 = frente) |
| Área px² | Cantidad de píxeles en el bounding box de la mancha |
| Vol. óptico | Densidad óptica integrada |
| Rel % | Porcentaje respecto al volumen máximo de todas las manchas |

Hacé clic en **⬇ Exportar CSV** para descargar la tabla completa y analizarla en Excel, R o Python.

---

## Referencia de parámetros

| Parámetro | Valor por defecto | Efecto |
|-----------|-------------------|--------|
| Threshold | 35 | Oscuridad mínima para considerar un píxel como "dark". Rango 5–100. |
| Min área | 300 px² | Tamaño mínimo de mancha reportada. Filtra ruido. |
| Ancho de calle | 40 px | Semi-ancho de la columna escaneada por calle. |

---

## Limitaciones y consideraciones

- **Aproximación por bounding box:** El área de la mancha se mide como el rectángulo envolvente, no la forma exacta. Esto sobreestima ligeramente el área de manchas de forma irregular.
- **Manchas solapadas:** Si dos manchas se superponen, pueden detectarse como una sola. Se recomienda separación manual dibujando rectángulos individuales.
- **Uniformidad de iluminación:** Una iluminación no uniforme de la placa (sombras, reflejos) introduce error sistemático en la densidad óptica.
- **Sin cuantificación absoluta:** Sin curva de calibración, los resultados son comparaciones relativas únicamente.
- **Placas TLC con manchas de color:** El algoritmo usa intensidad en escala de grises. Para manchas de color (ej: manchas con ninhidrina), la herramienta funciona igual pero puede ser necesario ajustar el threshold según la visibilidad de las manchas.

---

## Detalles técnicos

- **Lenguaje:** JavaScript puro + HTML5 Canvas — sin frameworks ni dependencias externas
- **Acceso a píxeles:** `ImageData` a resolución completa se cachea al cargar para análisis rápido
- **Detección de manchas:** Etiquetado de componentes conectados via BFS iterativo dentro de la columna de cada calle
- **Agrupamiento por Rf:** Manchas dentro de ±0.07 unidades de Rf se tratan como el mismo compuesto para comparación entre calles
- **Todo el procesamiento es del lado del cliente** — tus imágenes nunca salen del navegador

---

## Estructura de archivos

```
tlc-analyzer.html    ← La aplicación completa (archivo único autocontenido)
README.md            ← Documentación en inglés
README.es.md         ← Este archivo
```

---

## Casos de uso típicos

- Comparar el **rendimiento de una reacción** entre múltiples condiciones (cada calle = una condición)
- Monitorear la **pureza de fracciones de columna** (cada calle = una fracción)
- Verificar la **consistencia de siembra** entre calles antes de densitometría
- **Docencia** — los estudiantes pueden anotar y medir fotos de sus propias placas

---

## Contribuciones

Se aceptan issues y pull requests. Algunas ideas para mejoras futuras:

- [ ] Ajuste gaussiano de picos para manchas solapadas
- [ ] Soporte para placas invertidas (manchas claras sobre fondo oscuro)
- [ ] Procesamiento por lotes de múltiples imágenes
- [ ] Herramienta de curva de calibración para cuantificación absoluta
- [ ] Selección de canal de color (R/G/B) para manchas coloreadas

---

## Licencia

MIT — libre para usar, modificar y compartir.
