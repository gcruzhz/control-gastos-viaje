# CHANGELOG — Control de Gastos de Viaje

Historial de versiones. Las fechas son aproximadas (día de entrega):
las versiones 1.0 a 1.11 son del 2026-11-12, las 1.12 a 1.15 del
2026-11-17 y las 1.16 a 1.20 del 2026-11-18. La versión de la app la
mandan `versionCode` y `versionName` de `android/app/build.gradle`.

## 1.20 — 2026-11-18 (versionCode 21)

- **Copiar datos de facturación de otro viaje:** en los datos de
  facturación se puede elegir otro viaje ya registrado y copiar su
  razón social, RFC, régimen, CP, uso de CFDI, correo, WhatsApp y
  correo de envío, en vez de teclearlos de nuevo.

## 1.19 — 2026-11-18

- **Corrección del OCR:** la lectura del ticket ya **no** copia el
  folio del ticket al folio de la factura. En 1.18 ese folio marcaba el
  gasto como «Facturado» sin que hubiera factura; ahora el folio de
  factura solo lo escribe el usuario a mano.
- **«¿Quién lo gastó?» pasa al inicio** del formulario de gasto.
- **Concepto del OCR más estricto:** el concepto que se pre-llena
  desde el ticket filtra mejor las líneas que no son el nombre del
  negocio (totales, fechas, montos, «gracias por su compra», etc.).

## 1.18 — 2026-11-18

- **Lectura de tickets con OCR en el teléfono:** tesseract.js 7.0.0
  (copia local en `www/ocr/`, idioma español) lee la foto del ticket y
  pre-llena fecha, total, concepto y RFC para revisarlos antes de
  guardar. Corre en el teléfono, sin mandar la foto a ningún servidor,
  y solo se carga la primera vez que se procesa una foto. Si el OCR
  falla, el formulario queda como estaba y nada bloquea el guardado.

## 1.17 — 2026-11-18

- **Vista «Por viaje» / «Unificados»:** en el reporte se puede ver los
  gastos por viaje o todos los gastos de todos los viajes juntos.

## 1.16 — 2026-11-18

- **Subida de factura PDF + XML por gasto:** cada gasto facturable
  puede llevar su factura en PDF y en XML.
- **Estados de facturación:** «Pendiente por facturar», «Facturado» y
  «No facturable», según la regla: facturado = facturable Y (folio no
  vacío O (PDF Y XML)).

## 1.15 — 2026-11-17

- **Campos de persona ocultos con menos de 2 personas:** «¿Quién lo
  gastó?» y «Filtrar por persona» se ocultan cuando el viaje tiene una
  sola persona registrada (el gasto queda a su nombre) y aparecen a
  partir de dos personas, con «Compartido» o la persona elegida. Si
  queda una sola persona, se vuelven a ocultar.

## 1.14 — 2026-11-17

- **Auto-scroll en el formulario de nuevo viaje:** al abrir el
  formulario, la pantalla baja sola hasta los tres botones (Crear
  viaje, Cancelar, Guardar para después).
- **Versión visible en el encabezado:** la app muestra «Versión 1.14»
  (y siguientes) arriba.

## 1.13 — 2026-11-17

- **«Cancelar» y «Guardar para después»** en el formulario de nuevo
  viaje: «Cancelar» cierra el formulario sin crear nada; «Guardar para
  después» guarda el viaje en la sección **«Para después»** del inicio,
  desde donde se abre después como viaje normal.

## 1.12 — 2026-11-17

- **Foto de ticket con la cámara nativa:** el botón de foto abre
  directo la cámara del teléfono (plugin Camera) con su permiso
  correspondiente. Antes usaba un campo de archivo oculto que nunca
  abría la cámara. La galería no cambió.

## 1.11 — 2026-11-12

- Corrección de visualización con una sola persona en el viaje.
- Animación de encabezado (avión, tren, barco, hotel).

## 1.10 — 2026-11-12

- El selector «¿Quién lo gastó?» queda deshabilitado cuando el viaje
  tiene 0 o 1 persona registrada.

## 1.9 — 2026-11-12

- **Botón «Descargar Excel»** en la pestaña Reporte: un .xlsx real con
  5 hojas — Resumen, Presupuesto, Gastos, Personas y Facturación.
- **PDF elegante rediseñado**, con el mismo desglose, iconos y título,
  en los colores del logo (teal en encabezados y tablas, naranja en
  acentos).
- **Archivar y Eliminar viajes:** cada tarjeta de viaje tiene
  «Archivar» (mueve el viaje a la sección «Archivados», desde donde se
  puede restaurar) y «Eliminar» (pide confirmación porque borra todo
  el viaje).

## 1.8 — 2026-11-12

- **CSV y PDF con plugins nativos:** la exportación y el compartir de
  CSV y PDF usan los plugins Filesystem + Share, con la hoja nativa
  del sistema (que ofrece Guardar en Drive, WhatsApp, correo, etc.).
  Antes la descarga fallaba en el WebView.

## 1.7 — 2026-11-12

- Sección **«Acerca de»** (versión de la app y contacto) y botón
  **«Reportar un problema»** (abre un correo con la versión de la app).

## 1.6 — 2026-11-12

- **Constancia de situación fiscal en PDF** en los datos de
  facturación del viaje (se adjunta el PDF; no se incluye en la
  exportación CSV).

## 1.5 — 2026-11-12

- **CSV import/export de viajes completos:** exporta viaje,
  facturación, personas, presupuesto y gastos; importa el mismo
  formato. Las fotos de tickets, la constancia y las facturas PDF/XML
  no van en el CSV.

## 1.4 — 2026-11-12

- **Envío de datos de facturación por WhatsApp y correo** desde los
  datos de facturación del viaje.
- **Cambio de firma:** la APK release de la 1.4 se firmó con un
  keystore permanente (antes se firmaba con el keystore debug). Eso
  exigió **desinstalar la app una vez** y reinstalar; desde entonces
  las actualizaciones instalan encima sin perder los viajes.

## 1.3 — 2026-11-12

- Correcciones del splash (pantalla de apertura).

## 1.2 — 2026-11-12

- **Animación de apertura estilo TwoDots:** maleta, hotel, auto y
  comida al arrancar la app.

## 1.1 — 2026-11-12

- **Menú de inicio** con los viajes guardados (antes la app abría
  directo al último viaje o al formulario).

## 1.0 — 2026-11-12

- Versión base: crear viajes, presupuesto por categoría, gastos
  diarios con facturable, reporte, respaldo manual a Drive y envío de
  datos por WhatsApp/correo.
