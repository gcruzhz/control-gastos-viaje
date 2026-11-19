# Preguntas frecuentes para los devs — Control de Gastos de Viaje

Respuestas listas para pasarle a un desarrollador que va a tomar la app.
Versión actual: **1.20** (versionCode 21, 2026-11-18).

## ¿En qué lenguaje está?

**JavaScript en todo.** La app es una sola página web (HTML, CSS y
JavaScript, **sin framework** como React o Vue). Lo nativo de Android es
mínimo: solo lo que genera Capacitor y sus plugins.

## ¿En qué se hizo la interfaz gráfica?

Es **una sola interfaz**: un solo archivo `www/index.html` (unas 2.500
líneas) con CSS propio y toda la lógica en el mismo archivo. No hay
build de JavaScript: se edita el HTML y se sincroniza con Capacitor.

- **Teléfono (APK):** esa página corre dentro de Capacitor, en un
  WebView.
- **Escritorio / iOS:** no existen; solo hay APK de Android.

Los PDF y el Excel se generan con código propio, sin librerías de
diseño. Los colores son teal `#00879e` y naranja `#e07000`; el icono es
una maleta verde con ticket.

## ¿Qué herramientas se usaron?

| Para qué | Herramienta |
|---|---|
| Interfaz y lógica | JavaScript, HTML y CSS, sin framework |
| App de teléfono | Capacitor 8.6.0 |
| Cámara nativa (foto del ticket) | @capacitor/camera 8.2.6 |
| Archivos y hoja de compartir | @capacitor/filesystem 8.1.5, @capacitor/share 8.0.3 |
| Pantalla de inicio | @capacitor/splash-screen 8.0.2 |
| Paquetes | pnpm (npm falla en este sandbox con `EPERM chown`) |
| Compilar la APK | Gradle con Android SDK (targetSdk 36, minSdk 24) |
| Leer los tickets por foto (OCR) | tesseract.js 7.0.0, copia local en `www/ocr/` con el idioma español (`spa.traineddata`) |

## ¿Dónde están los datos?

En el **teléfono del usuario, y solo ahí**. Una sola clave de
localStorage del WebView:

- Clave: `gastos-viaje-v1`
- Contenido: todos los viajes con su presupuesto, gastos, personas y
  datos de facturación (ver `README.md` para el modelo completo).

No hay base de datos, no hay servidor, no hay nube y la app no
sincroniza con nada. Si se desinstala la app, los datos se pierden
salvo que el usuario haya exportado un CSV. La app empieza vacía.

## ¿Cómo se compila?

```bash
source ~/workspace/android/env.sh
cd ~/workspace/android/gastos-app
npx cap sync          # después de editar www/
cd android
./gradlew assembleRelease
```

La APK queda en `android/app/build/outputs/apk/release/`. La versión
la mandan `versionCode` y `versionName` en
`android/app/build.gradle` (hoy 21 / «1.20»); el `version` de
`package.json` (1.0.0) no se usa. El entorno de Android (JDK 21, SDK,
proxy y CA del sandbox) está documentado en `~/AGENTS.md`.

La APK release se firma con un **keystore permanente** fuera del repo
(`~/workspace/android/keys/`, datos en `keys/keystore.properties`).
Con esa firma las actualizaciones instalan encima sin perder los
viajes; cambiar de firma exige desinstalar una vez.

## ¿Cómo se publica?

Hoy se instala directo por APK. La publicación en Google Play está
pendiente: existe documentación de publicación del 2026-11-12 (ficha
técnica, funciones, guía de uso, borrador de política de privacidad y
textos listos para Play Console), y falta: cuenta de desarrollador,
formato AAB, política de privacidad, ficha de tienda, clasificación de
contenido y formulario de seguridad de datos.

## ¿Qué es el OCR?

La lectura de tickets con **tesseract.js 7.0.0**, una copia local en
`www/ocr/` (no es dependencia de npm). Corre **en el teléfono**, sin
mandar la foto a ningún servidor, y solo se carga la primera vez que se
procesa una foto.

Del texto del ticket extrae: fecha, total (la línea que dice TOTAL),
concepto (el nombre del negocio) y RFC. Esos campos se pre-llenan para
revisarlos antes de guardar. El **folio del ticket no se extrae**: el
folio de la factura solo lo escribe el usuario a mano, porque un folio
de ticket no es la factura. Si el OCR falla o no encuentra nada útil,
el formulario queda como estaba y nada bloquea el guardado.

## ¿Dónde están las claves?

**No hay claves en el código.** No hay Firebase, no hay cuentas, no
hay tokens ni servicios externos. Lo único sensible es el keystore de
firma, que vive fuera del repo en `~/workspace/android/keys/` y cuyas
contraseñas solo están en `keys/keystore.properties` (permisos 600):
nunca se imprimen ni se copian.

## ¿Qué me pueden preguntar los devs?

- **Acceso al código:** el proyecto vive en
  `~/workspace/android/gastos-app/`; la publicación a GitHub está
  pendiente (por confirmar si hay repo).
- **La firma:** la APK release usa el keystore permanente; si se pierde
  o se cambia, los usuarios tendrían que desinstalar y reinstalar.
- **El respaldo:** es manual por CSV; el CSV no incluye fotos de
  tickets, constancia ni facturas PDF/XML.
- **La regla de facturación:** facturado = facturable Y (folio no vacío
  O (PDF Y XML)). El folio de factura lo escribe el usuario; el OCR
  nunca lo llena.

## ¿Qué falta por hacer?

- **Nube y cuentas:** guardar los viajes en la nube para usar la app en
  otro teléfono o computadora, como la versión cloud de Checklist de
  Soporte, con creación de cuentas y un usuario administrador.
  Es lo que sigue: hay un plan pendiente de aprobación de Gerardo.
- **Publicación en Google Play:** cuenta de desarrollador, AAB,
  política de privacidad, ficha de tienda, clasificación y formulario
  de seguridad de datos.
- **Respaldo automático:** hoy el respaldo es manual por CSV.
- **Versión de escritorio / iOS:** por confirmar; hoy solo hay APK.

## Cómo se prueba todo

En teléfono Android real, instalando la APK encima de la versión
anterior: los viajes guardados no se deben perder. Pendiente de probar
en 1.20: copiar los datos de facturación de otro viaje. Pendiente de
probar en 1.19: que el OCR ya no copie el folio del ticket y que
«¿Quién lo gastó?» esté al inicio del formulario. Pendiente de probar
en 1.18: la lectura de tickets con OCR en tickets reales.
