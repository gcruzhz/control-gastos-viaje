# Control de Gastos de Viaje

**Versión actual: 1.20** (versionCode 21) — 2026-11-18.
App Android (Capacitor) para registrar el presupuesto y los gastos de un
viaje de trabajo, día a día, y saber qué gastos se pueden facturar.

## Qué es

Gerardo viaja por trabajo y necesita llevar la cuenta de lo que gasta:
primero arma el presupuesto del viaje por categoría, luego registra cada
gasto conforme ocurre, con su foto de ticket, y al final valida cuáles
gastos se pueden facturar (con su folio o con su factura PDF + XML) para
comprobarlos. Todo queda en el teléfono, sin cuentas ni nube.

## Para quién

Para una persona que viaja sola o con compañeros de trabajo y tiene que
comprobar gastos: técnicos de soporte, personal de campo, cualquiera que
viaje por trabajo y facture viáticos.

## Funciones principales (hasta 1.20)

- **Menú de inicio:** lista de viajes guardados, sección «Para después»
  (viajes planeados sin abrir) y sección «Archivados».
- **Viajes:** nombre, destino, fechas de inicio y fin, moneda (MXN, USD,
  EUR). Cada viaje se puede archivar, restaurar o eliminar (con
  confirmación, porque borra todo el viaje).
- **Presupuesto:** por categoría (Transporte, Hospedaje, Alimentación,
  Combustible, Casetas, Entretenimiento, Otros), con concepto y monto.
- **Gastos diarios:** fecha, categoría, concepto, monto, quién lo gastó
  (si hay 2 o más personas en el viaje) y si es facturable.
- **Personas:** nombre, rol y notas por cada persona del viaje. Con una
  sola persona, los campos de persona se ocultan y el gasto queda a su
  nombre.
- **Facturación por gasto:** cada gasto facturable tiene un estado —
  «Pendiente por facturar», «Facturado» o «No facturable» — según la regla
  de la sección de datos. Se puede subir la factura en PDF y XML de cada
  gasto.
- **Datos de facturación del viaje:** razón social, RFC, régimen fiscal,
  código postal, uso de CFDI, correo, WhatsApp y correo de envío; se
  puede adjuntar la constancia de situación fiscal en PDF y copiar los
  datos de facturación de otro viaje ya registrado.
- **Lectura de tickets con OCR:** la foto del ticket se lee en el propio
  teléfono y pre-llena fecha, total, concepto y RFC para revisarlos antes
  de guardar. El folio del ticket **no** se copia al folio de factura.
- **Foto de ticket con la cámara nativa** del teléfono (plugin Camera).
- **Reporte:** resumen por viaje (presupuestado, gastado, restante, %
  usado), vista «Por viaje» o «Unificados» (todos los gastos juntos), y
  compartir por WhatsApp o correo.
- **Exportar:** Excel real (.xlsx) con 5 hojas — Resumen, Presupuesto,
  Gastos, Personas y Facturación — y PDF elegante con el mismo desglose.
  También CSV para importar/exportar viajes completos.
- **Respaldo:** exportación manual a CSV; la hoja nativa de compartir
  ofrece guardar en Drive. No hay respaldo automático.
- **Acerca de y Reportar un problema:** la versión visible en el
  encabezado y un botón que abre un correo a gerardo.cruzhz@linux.com
  con la versión de la app.
- **Animación de apertura** estilo TwoDots (maleta, hotel, auto, comida)
  y animación de encabezado (avión, tren, barco, hotel).

## Stack técnico

| Para qué | Herramienta |
|---|---|
| Interfaz y lógica | JavaScript, HTML y CSS, **sin framework** |
| App de teléfono | Capacitor 8.6.0 (WebView) |
| Cámara nativa | @capacitor/camera 8.2.6 |
| Archivos y compartir | @capacitor/filesystem 8.1.5, @capacitor/share 8.0.3 |
| Pantalla de inicio | @capacitor/splash-screen 8.0.2 |
| Lectura de tickets (OCR) | tesseract.js 7.0.0, copia local en `www/ocr/` (idioma español, `spa.traineddata`) |
| Paquetes | pnpm (npm falla en este sandbox con `EPERM chown`; ver `~/workspace/android/` y `AGENTS.md`) |
| Compilar la APK | Gradle con Android SDK (targetSdk 36, minSdk 24) |
| Colores | teal `#00879e` y naranja `#e07000` |
| Icono | maleta verde con ticket |

No hay claves, tokens ni servicios externos en el código: no hay
Firebase, no hay cuentas, no hay backend. La app no sincroniza con nada.

## Estructura del proyecto

```
gastos-app/
├── www/
│   ├── index.html        # Toda la app: interfaz, estilos y lógica (2.503 líneas)
│   ├── app-icon.png      # Icono (maleta verde con ticket)
│   ├── manifest.json
│   └── ocr/              # tesseract.js 7.0.0 local: tesseract.min.js,
│       ├── lang/         #   worker.min.js, tesseract-core.wasm(.js),
│       │   └── spa.traineddata   #   idioma español
├── android/              # Proyecto Android generado por Capacitor
│   └── app/build.gradle  # applicationId, versionCode, versionName, firma
├── capacitor.config.json # appId mx.gerardo.gastosviaje, webDir www
├── package.json          # Dependencias de Capacitor (versión npm: 1.0.0,
│                         #   la versión real de la app vive en build.gradle)
└── docs/                 # Esta documentación
    ├── README.md
    ├── PREGUNTAS-DEVS.md
    └── CHANGELOG.md
```

La versión de la app la mandan `versionCode` y `versionName` de
`android/app/build.gradle` (hoy 21 / «1.20»), y el encabezado de la app
la muestra. El `version` de `package.json` (1.0.0) no se usa para la
versión de la app.

## Estructura de datos

Todos los datos viven en **localStorage** del WebView, en una sola clave:

- Clave: `gastos-viaje-v1`
- Contenido: un objeto con la lista de viajes (y el estado de la vista).
- La app empieza vacía: sin viajes de ejemplo.

### Viaje

```js
viaje = {
  id,                    // identificador único
  nombre,                // ej. "Viaje a Guadalajara"
  destino,
  inicio,                // fecha ISO aaaa-mm-dd
  fin,                   // fecha ISO aaaa-mm-dd
  moneda,                // "MXN" | "USD" | "EUR" (por defecto MXN)
  archived,              // true = en la sección Archivados
  paraDespues,           // true = en la sección "Para después" (borrador)
  billing: {
    razon, rfc, regimen, cp, uso,      // razón social, RFC, régimen fiscal,
    correo, whatsapp, correoEnvio,     //   CP, uso de CFDI, correos y WhatsApp
    constancia                         // PDF de la constancia de situación fiscal
  },
  people:   [ { id, nombre, rol, notas } ],
  budget:   [ { id, categoria, concepto, monto } ],
  expenses: [ {
    id, fecha, categoria, concepto, monto,
    personaId,          // quién lo gastó (id de people)
    facturable,         // true/false
    folio,              // folio/UUID de la factura (lo escribe el usuario a mano)
    nota,
    foto,               // foto del ticket (data URL)
    facturaPdf,         // { nombre, data } de la factura en PDF
    facturaXml          // { nombre, data } de la factura en XML
  } ]
}
```

Categorías de presupuesto y gastos:

```
Transporte, Hospedaje, Alimentación, Combustible, Casetas,
Entretenimiento, Otros
```

### Regla de facturación

Un gasto está **facturado** cuando es facturable **y** (tiene folio **o**
tiene factura PDF **y** XML):

```js
esFacturado = facturable && (folio no vacío || (facturaPdf && facturaXml))
```

Estados que muestra la app:

| Estado | Condición |
|---|---|
| No facturable | `facturable` es falso |
| Facturado | `esFacturado` es verdadero |
| Pendiente por facturar | facturable, pero sin folio ni PDF+XML |

El folio del ticket que lee el OCR **no** se copia al folio de factura:
el folio de factura solo lo escribe el usuario a mano. (En la 1.18 el
OCR sí lo copiaba y eso marcaba gastos como «Facturado» sin factura;
se corrigió en la 1.19.)

## Cómo compilar la APK

El entorno de Android (JDK 21, SDK, proxy y CA) está en
`~/workspace/android/`; las notas de ese entorno viven en
`~/AGENTS.md`. Resumen:

```bash
source ~/workspace/android/env.sh
cd ~/workspace/android/gastos-app

# Después de editar www/ (la app es web, no hay build de JS):
npx cap sync

# Compilar la APK release (firmada):
cd android
./gradlew assembleRelease
# APK en: android/app/build/outputs/apk/release/app-release.apk
```

- La versión se sube editando `versionCode` y `versionName` en
  `android/app/build.gradle` (y el texto «Versión X.Y» del encabezado
  en `www/index.html`).
- La APK release se firma con un **keystore permanente** guardado fuera
  del repo, en `~/workspace/android/keys/` (sus datos se leen de
  `keys/keystore.properties`). **Nunca se usó ni se usa el keystore
  debug para las releases**: cambiar de firma exige desinstalar la app
  instalada una vez; con la firma permanente las actualizaciones
  instalan encima sin perder los viajes.
- No se imprimen ni se copian contraseñas del keystore en ningún lado:
  solo existen en `keys/keystore.properties` (permisos 600) y en
  `build.gradle` se leen por nombre de propiedad.

## Cómo se publica (por confirmar)

La publicación en Google Play está pendiente: existe documentación de
publicación entregada el 2026-11-12 (ficha técnica, funciones, guía de
uso, borrador de política de privacidad y textos listos para Play
Console), y lo que falta para publicar es: cuenta de desarrollador,
formato AAB, política de privacidad, ficha de tienda, clasificación de
contenido y formulario de seguridad de datos. La app hoy se instala
directamente por APK.

## Limitaciones conocidas

- **Sin nube ni cuentas:** todo vive en el teléfono. Si se desinstala
  la app o se borran los datos del WebView, los viajes se pierden salvo
  que haya un respaldo CSV exportado.
- **localStorage:** es el único almacén. No hay base de datos ni
  sincronización entre dispositivos.
- **CSV sin fotos:** la exportación/importación CSV incluye viaje,
  facturación, personas, presupuesto y gastos, pero **no** incluye las
  fotos de tickets, el PDF de la constancia ni las facturas PDF/XML
  (son muy pesadas para el CSV). El Excel y el PDF tampoco las
  incrustan.
- **Respaldo manual:** no hay respaldo automático; el usuario exporta
  el CSV cuando quiere y lo guarda donde quiera (Drive, WhatsApp,
  correo) desde la hoja de compartir.
- **OCR en el teléfono:** la lectura de tickets corre local con
  tesseract.js; si la foto está borrosa o el ticket no tiene el formato
  esperado, el OCR puede no encontrar fecha o total y el formulario se
  llena a mano. Si el OCR falla, no bloquea nada.
- **Moneda por viaje:** cada viaje tiene una sola moneda; no hay
  conversión entre monedas.
- **Solo Android:** no hay versión de escritorio ni de iOS.
