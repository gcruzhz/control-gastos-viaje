# Plan — Control de Gastos de Viaje 2.0 (nube)

**Fecha:** 2026-11-18.
**Versión objetivo:** 2.0.0 (versionCode 22, versionName «2.0.0»).
**Estado:** plan aprobado por Gerardo; **código todavía no escrito**.
**Repo:** https://github.com/gcruzhz/control-gastos-viaje (ya creado por Gerardo, público, rama `main`).

Este documento es el plan de construcción. No contradice `README.md`,
`PREGUNTAS-DEVS.md` ni `CHANGELOG.md` (1.0–1.20): la 1.20 sigue siendo
la versión actual hasta que la 2.0.0 esté lista, y todo lo que la 1.20
hace hoy debe seguir funcionando en la 2.0.0.

---

## 1. Alcance de la 2.0.0

### 1.1 Qué entra

1. **Proyecto Firebase nuevo y separado**, nombre sugerido
   `control-gastos-viaje`. **NO** se reutiliza `checklist-soporte`:
   son datos fiscales personales y deben vivir aparte.
2. **Cuentas de usuario con registro libre** (self-registration):
   correo + contraseña, y login social con **Google, Meta (Facebook)
   y X**. Firebase Auth con esos 4 proveedores.
3. **Roles:** Gerardo es admin (`gerardo.cruzhz@gmail.com`): ve todos
   los viajes y usuarios, desactiva cuentas y cambia de plan a un
   usuario. Los usuarios normales ven solo sus propios viajes.
4. **Sincronización en la nube:** Firestore con persistencia offline
   (mismo patrón que Checklist cloud): la app funciona sin internet y
   sincroniza al volver la conexión.
5. **Archivos en Firebase Storage:** fotos de tickets, constancia de
   situación fiscal (PDF) y facturas PDF/XML de cada gasto. Esto
   resuelve el tope de ~5 MB de localStorage: los archivos ya no viven
   dentro del JSON.
6. **Migración automática al primer inicio** de la 2.0: lee
   `gastos-viaje-v1`, sube todo a Firestore/Storage, marca la migración
   como hecha y **conserva el localStorage como respaldo**. No se borra
   nada sin confirmación del usuario.
7. **Dos planes: básica y premium** (ver §2). La lógica de planes
   (campo `plan` en el usuario, gating de funciones, placeholder de
   anuncios) se construye en la 2.0.0; AdMob y Play Billing se conectan
   después, cuando existan las cuentas (ver §6).
8. **Misma app, misma firma:** mismo `appId`
   (`mx.gerardo.gastosviaje`), mismo keystore permanente
   (`~/workspace/android/keys/gastos-release.keystore`), versionCode 22.
   La 2.0.0 **instala sobre la 1.20 sin perder datos**.

### 1.2 Qué NO entra en la 2.0.0

- **Viajes compartidos** (invitar a otros usuarios a un viaje): va en
  la **2.1** (decisión en §2.3).
- **Login con Apple:** queda para después. Necesita cuenta de
  desarrollador de Apple ($99/año), que Gerardo no tiene. Se deja el
  gancho preparado (proveedor en el módulo de nube, botón oculto o
  marcado «próximamente»), pero **no se construye** en la 2.0.0.
- **Versión de computadora (Electron/Hosting):** fuera de la 2.0.0;
  después, con el mismo patrón que Checklist (ver §5, 2.2).
- **AdMob y Play Billing reales:** la lógica se construye, el cobro y
  los anuncios reales esperan a las cuentas (ver §6).
- **Edición de gastos existentes:** sigue pendiente como en la 1.20
  (no es parte de la 2.0.0 salvo que Gerardo lo pida).

### 1.3 Funciones de la 1.20 que NO se pueden perder

Checklist de no-regresión (todo esto existe hoy y debe seguir igual
en la 2.0.0):

- Viajes con presupuesto por categoría (Transporte, Hospedaje,
  Alimentación, Combustible, Casetas, Entretenimiento, Otros).
- Gastos diarios con fecha, categoría, concepto, monto, facturable.
- Personas por viaje; «¿Quién lo gastó?» arriba del formulario y solo
  con 2+ personas (con 1 persona se oculta y el gasto queda a su
  nombre).
- Foto de ticket con la **cámara nativa** del teléfono.
- **OCR local** (tesseract.js en el teléfono): pre-llena fecha, total,
  concepto y RFC para revisar antes de guardar. **La foto no se sube a
  ningún lado para leerla.** El OCR **NUNCA** copia el folio del ticket
  al folio de factura (regla de la 1.19).
- Facturación: estado «Pendiente por facturar» / «Facturado» /
  «No facturable» con la regla
  `facturado = facturable && (folio no vacío || (PDF y XML))`;
  el folio de factura solo lo escribe el usuario a mano.
- Vista de reporte «Por viaje» / «Unificados».
- Datos de facturación copiables de otro viaje (razón social, RFC,
  régimen, CP, uso de CFDI, correo, WhatsApp, correo de envío) y
  constancia en PDF.
- Excel real (.xlsx, 5 hojas: Resumen, Presupuesto, Gastos, Personas,
  Facturación) y PDF elegante (teal `#00879e`, naranja `#e07000`).
- CSV import/export, respaldo manual en Drive (hoja nativa de
  compartir).
- Archivar / restaurar / eliminar viajes (eliminar con confirmación).
- Borradores «Para después» (Crear / Cancelar / Guardar para después).
- Animación de apertura estilo TwoDots y versión visible en el
  encabezado («Versión 2.0.0»).

---

## 2. Planes: básica y premium

### 2.1 Corte propuesto

| Función | Básica (gratis) | Premium |
|---|---|---|
| Nube: sincronización de viajes entre dispositivos | Sí | Sí |
| Viajes activos (no archivados ni «Para después») | Hasta **3** | **Ilimitados** |
| Anuncios | Sí (banner) | **Sin anuncios** |
| Respaldo automático programado (exportación a Drive) | No (manual, como hoy) | Sí |
| Viajes compartidos (invitar a otros usuarios) | No | Sí — **en la 2.1** |
| Soporte / funciones nuevas | Las de la básica | Prioridad |

Notas sobre el corte:

- **La sincronización a la nube es el respaldo en la práctica:**
  Firestore con caché offline ya guarda los datos fuera del teléfono
  y los restaura al reinstalar. El «respaldo automático» premium es
  un extra por encima de eso: exportación programada (Excel/PDF/CSV)
  a Drive sin que el usuario lo pida. La básica conserva el respaldo
  manual por CSV que ya existe en la 1.20.
- **El límite de 3 viajes activos** se cuenta sobre viajes no
  archivados y no «Para después»: los archivados y los borradores no
  cuentan. Al llegar al límite, la app avisa y ofrece archivar un
  viaje o pasar a premium; nunca bloquea ver ni exportar los viajes
  que ya existen.
- **Los anuncios son un placeholder en la 2.0.0:** se reserva el
  espacio del banner y la lógica «si plan == básica, mostrar anuncio»,
  pero sin SDK de AdMob ni IDs reales hasta que exista la cuenta de
  AdMob (ver §6). **No se inventan IDs de AdMob ni claves.**
- **El cobro premium es lógica primero, cobro después:** el campo
  `plan` y el gating se construyen en la 2.0.0; Play Billing se conecta
  cuando exista la cuenta de desarrollador de Google Play. Mientras
  tanto, solo Gerardo (admin) puede cambiar el `plan` de un usuario
  desde la app o desde la consola de Firebase.

### 2.2 Campo `plan` y gating

- `users/{uid}.plan`: `"basica"` | `"premium"` (por defecto
  `"basica"` al registrarse).
- `users/{uid}.planDesde`: fecha del cambio de plan.
- El gating se hace en la app (antes de crear el 4.º viaje activo,
  antes de mostrar funciones premium) **y** en las reglas de Firestore
  (ver §3.3): la app no es la única defensa.
- Cambiar de plan lo hace el admin (Gerardo) o, cuando exista Play
  Billing, la compra en Play actualiza el campo. El usuario nunca se
  cambia el plan a sí mismo desde la app.

### 2.3 Decisión: viajes compartidos van en la 2.1, no en la 2.0.0

**Recomendación: NO incluir viajes compartidos en la 2.0.0.**

Razones:

1. **Cambian el modelo de datos y las reglas.** Un viaje compartido
   necesita `ownerUid` + lista de `memberUids`, y las reglas deben
   permitir leer/escribir a dueño y miembros, pero no a nadie más.
   Eso es un segundo modelo de permisos encima del modelo «cada quien
   ve lo suyo» que necesita la 2.0.0.
2. **Cambian la interfaz.** Hay que añadir invitar/quitar
   participantes, ver quién es dueño y quién invitado, y resolver
   conflictos (dos personas editando el mismo gasto sin conexión).
3. **Son función premium.** El gating de premium debe existir y estar
   probado antes de colgarle una función encima.
4. **La 2.0.0 ya es grande:** cuentas + 3 logins sociales + migración
   + Storage + planes. Meter compartidos en el mismo salto retrasa
   todo y arriesga la migración, que es lo que no puede fallar.

En la 2.0.0 el modelo ya deja el gancho: `trips/{id}` lleva
`ownerUid` y un campo `memberUids` (vacío hasta la 2.1), y las reglas
ya contemplan «dueño o miembro» aunque todavía no haya miembros. La
2.1 solo tiene que llenar ese campo y añadir la interfaz de invitación.

---

## 3. Modelo de datos en Firestore

### 3.1 Patrón reutilizado de Checklist cloud

Se copia el patrón que ya funciona en
`~/workspace/android/checklist-cloud/`:

- `www/cloud/firebase.js` (ESM) compilado con esbuild a
  `www/cloud/firebase.bundle.js` (`pnpm build:cloud`), que expone
  `window.Cloud`. **Después de editar `www/cloud/firebase.js` hay que
  reconstruir el bundle y correr `npx cap sync`** (lección de la
  2.5.4 de Checklist: el bundle viejo se queda en la app instalada).
- `www/firebase-config.js` con la configuración del proyecto (el
  `.example.js` sí se versiona; el real no, está en `.gitignore`).
  La `apiKey` web es pública por diseño; la seguridad la dan las
  reglas.
- `initializeFirestore` con `persistentLocalCache`: caché offline
  persistente; la app trabaja sin internet y sincroniza al volver.
- `firestore.rules` en la raíz del repo como fuente de verdad de la
  seguridad; se publica desde la consola o con
  `firebase deploy --only firestore:rules`.
- Login con persistencia local (recordar sesión), como en Checklist.
- Dependencia `firebase` (misma versión mayor que Checklist,
  `^11.10.0`) instalada con **pnpm** (npm falla en este sandbox con
  `EPERM chown`; ver `~/AGENTS.md`).

Lo nuevo respecto a Checklist: **Firebase Storage** (Checklist no lo
usa) y **login social** (Checklist solo usa correo/contraseña).

### 3.2 Colecciones

```
users/{uid}
  email            string   correo de Auth (minúsculas)
  nombre           string   nombre para mostrar
  rol              string   "admin" | "usuario"   (Gerardo: "admin")
  plan             string   "basica" | "premium"  (defecto "basica")
  planDesde        timestamp
  activo           boolean  false = cuenta desactivada por el admin
  proveedores      array    ["password", "google", "facebook", "x"]
  creadoEn         timestamp
  ultimoAcceso     timestamp

trips/{tripId}
  ownerUid         string   uid del dueño (el que lo creó)
  memberUids       array    uids invitados — VACÍO en 2.0.0, se usa en 2.1
  nombre, destino  string
  inicio, fin      string   fecha ISO aaaa-mm-dd
  moneda           string   "MXN" | "USD" | "EUR"
  archived         boolean
  paraDespues      boolean
  billing          map      { razon, rfc, regimen, cp, uso, correo,
                              whatsapp, correoEnvio,
                              constanciaPath }   // ruta en Storage, no base64
  creadoEn         timestamp
  actualizadoEn    timestamp

trips/{tripId}/people/{personId}
  nombre, rol, notas   string

trips/{tripId}/budget/{budgetId}
  categoria        string   Transporte | Hospedaje | Alimentación |
                              Combustible | Casetas | Entretenimiento | Otros
  concepto         string
  monto            number

trips/{tripId}/expenses/{expenseId}
  fecha            string   fecha ISO aaaa-mm-dd
  categoria        string   (mismas 7)
  concepto         string
  monto            number
  personaId        string   id de people (o null si 1 persona)
  facturable       boolean
  folio            string   folio de factura, solo a mano (nunca del OCR)
  nota             string
  fotoPath         string   ruta en Storage de la foto del ticket (o null)
  facturaPdfPath   string   ruta en Storage del PDF de la factura (o null)
  facturaXmlPath   string   ruta en Storage del XML de la factura (o null)
  creadoEn         timestamp

config/admin
  initialized      boolean  true cuando ya existe el admin
  adminUid         string   uid de Gerardo

migrations/{uid}
  hecha            boolean
  fecha            timestamp
  viajesMigrados   number
  gastosMigrados   number
  archivosMigrados number
  errores          array    (qué falló, si algo falló)
```

Decisiones de diseño:

- **Subcolecciones dentro de `trips/`** (people, budget, expenses)
  en vez de colecciones planas: cada viaje es una unidad que se
  consulta, se escucha y se borra junta; las reglas heredan el permiso
  del viaje padre. La vista «Unificados» usa una
  `collectionGroup("expenses")` con filtro por `ownerUid` (por eso el
  campo `ownerUid` se duplica también en cada gasto — ver nota abajo).
- **Los archivos viven en Storage, no en Firestore.** En Firestore
  solo va la **ruta** (`fotoPath`, `facturaPdfPath`, …); el contenido
  base64 de la 1.20 se convierte a archivo al migrar y se sube. Esto
  elimina el tope de ~5 MB.
- **El estado de facturación no se guarda:** se calcula igual que
  hoy (`facturable && (folio || (pdf && xml))`). Guardarlo duplicaría
  la regla y se desincronizaría.
- **IDs:** se conservan los `id` de la 1.20 (viaje, persona,
  presupuesto, gasto) como IDs de documento al migrar, para que los
  enlaces internos (`personaId`) sigan funcionando sin reescribirlos.
- **Campo duplicado `ownerUid` en subcolecciones:** las reglas de una
  subcolección no pueden leer el documento padre sin un `get()` extra;
  duplicar `ownerUid` en cada gasto/persona/presupuesto permite reglas
  simples y consultas `collectionGroup` filtradas por dueño. Se escribe
  una sola vez (al crear) y nunca cambia de dueño el viaje en 2.0.0.

### 3.3 Reglas de seguridad (líneas generales)

Archivo `firestore.rules` en la raíz del repo (igual que Checklist).
Resumen; el detalle exacto se escribe al construir:

- **Sin sesión, nada:** toda lectura/escritura exige `request.auth !=
  null`, salvo nada público. No hay lectura pública de ningún dato
  personal ni fiscal.
- **`users/{uid}`:** cada usuario lee y actualiza solo su propio
  documento (nombre, proveedores); **solo el admin** cambia `rol`,
  `plan` y `activo`, y solo el admin lista todos los usuarios. Un
  usuario desactivado (`activo == false`) no puede leer ni escribir
  nada más.
- **`trips/{tripId}` y sus subcolecciones:** leer/escribir solo el
  dueño (`ownerUid == request.auth.uid`) o un miembro
  (`request.auth.uid in memberUids`, vacío en 2.0.0). El admin lee
  todos los viajes; el admin puede borrar cualquier viaje. Nadie puede
  cambiar el `ownerUid` de un viaje que no creó.
- **`config/admin`:** lectura para usuarios activos; creación una sola
  vez (bootstrap del primer admin); nunca se actualiza ni se borra.
- **`migrations/{uid}`:** solo el propio usuario (escritura) y el
  admin (lectura).
- **Límite de 3 viajes activos (básica):** la app lo controla antes de
  crear; las reglas no pueden contar documentos sin leerlos todos, así
  que la regla no lo impone — se documenta como control de app, no de
  reglas. (Si en el futuro se necesita imponerlo en servidor, hace
  falta Cloud Functions, que la 2.0.0 no usa, igual que Checklist.)

**Reglas de Storage** (archivo `storage.rules`, nuevo; Checklist no
tiene):

```
users/{uid}/trips/{tripId}/tickets/{expenseId}.jpg
users/{uid}/trips/{tripId}/facturas/{expenseId}.pdf | .xml
users/{uid}/trips/{tripId}/constancia.pdf
```

- Solo el dueño del viaje (o un miembro en 2.1) puede subir, leer y
  borrar sus archivos; el admin puede leerlos todos.
- Solo se aceptan los tipos esperados por carpeta (imagen en
  `tickets/`, PDF/XML en `facturas/`, PDF en `constancia.pdf`) y con
  un tamaño máximo razonable por archivo (p. ej. 10 MB por foto,
  10 MB por PDF/XML; a fijar al construir).

### 3.4 Qué pasa con localStorage

- **Después de migrar, Firestore es la fuente de verdad** (con su
  caché offline persistente). El `save()` actual se sustituye por
  escrituras a Firestore; la caché offline hace el papel que hoy hace
  localStorage.
- `gastos-viaje-v1` **no se borra**: queda como respaldo local. Se
  añade una segunda clave, `gastos-viaje-migrado-v2`, con
  `{ hecha: true, fecha }` para no volver a migrar.
- Si Firestore no está disponible (sin proyecto configurado, sin
  sesión), la app sigue funcionando en modo local con localStorage,
  como la 1.20 — el mismo patrón «sin config = modo LOCAL» de
  Checklist. Esto permite probar la 2.0.0 sin nube y no deja tirado a
  nadie si la nube falla.

---

## 4. Plan de migración (1.20 → 2.0.0)

### 4.1 Cuándo y cómo se dispara

- Al primer inicio de la 2.0.0 con sesión iniciada (o justo después
  del primer login/registro), si `gastos-viaje-migrado-v2` no existe
  y `gastos-viaje-v1` tiene viajes, la app muestra una pantalla de
  migración: «Vamos a subir tus viajes a la nube» con progreso
  (viaje X de N, archivos subidos) y un botón «Migrar ahora».
- La migración **no se hace en silencio ni sin avisar**: el usuario
  ve qué se va a subir y puede posponerla («Más tarde»). Si la
  pospone, la app sigue en modo local hasta que acepte.
- Si no hay viajes locales, no hay migración: se marca como hecha
  directamente.

### 4.2 Pasos

1. **Leer y validar** `gastos-viaje-v1` con `try/catch`. Si el JSON
   está corrupto: **no se resetea a vacío en silencio** (ese es el
   bug de la 1.20). Se muestra el error, se conserva el texto crudo
   y se ofrece importar un CSV de respaldo como alternativa.
2. **Crear los documentos** en Firestore conservando los `id`
   originales: `trips/{id}` + subcolecciones `people`, `budget`,
   `expenses`. El `billing.constancia` base64 y los `foto` /
   `facturaPdf` / `facturaXml` base64 **no** van a Firestore.
3. **Subir los archivos a Storage** convirtiendo cada base64 a Blob:
   foto del ticket → `tickets/{expenseId}.jpg`; factura PDF/XML →
   `facturas/{expenseId}.pdf|xml`; constancia → `constancia.pdf`.
   Cada subida actualiza el documento con su `*Path`. Si un archivo
   falla, se registra en `migrations/{uid}.errores` y se reintenta en
   el siguiente inicio; el viaje igual queda migrado.
4. **Escribir `migrations/{uid}`** con contadores y fecha, y la clave
   local `gastos-viaje-migrado-v2`.
5. **Verificar**: leer de Firestore los viajes migrados y compararlos
   con el local (número de viajes, gastos y archivos). Si algo no
   cuadra, se avisa y no se marca la migración como hecha.
6. **Conservar el respaldo**: `gastos-viaje-v1` queda intacto en el
   teléfono. Solo se ofrece borrarlo (con confirmación explícita)
   cuando la migración esté verificada y el usuario lo pida.

### 4.3 Casos límite

- **La 1.20 sigue funcionando offline con sus datos hasta migrar:**
  quien no actualice, o actualice y posponga la migración, no pierde
  nada. La actualización instala encima (misma firma) y los datos
  locales siguen ahí.
- **Reinstalación limpia (sin localStorage):** no hay nada que migrar;
  al iniciar sesión, los viajes bajan de Firestore.
- **Dos teléfonos con datos distintos:** cada teléfono migra sus
  propios viajes locales a la misma cuenta; los `id` locales son
  únicos por teléfono, así que no se pisan. Si el mismo viaje existe
  en los dos teléfonos con distinto `id`, quedará duplicado: se avisa
  en la pantalla de migración y el usuario podrá borrar el duplicado.
- **Sin internet durante la migración:** se sube lo que se puede, se
  guarda el progreso en `migrations/{uid}` y se continúa al volver la
  conexión. La caché offline de Firestore no sube archivos a Storage;
  las subidas pendientes se reintentan al recuperar conexión.
- **Cuota de localStorage al migrar:** la migración solo lee el
  localStorage, no escribe viajes nuevos en él; no hay riesgo de
  superar los ~5 MB durante la migración.

---

## 5. Secuencia de construcción por versiones

### 5.1 Versión 2.0.0 — orden de trabajo

| Fase | Qué se hace | Entregable verificable |
|---|---|---|
| **A. Base nube** | Copiar el patrón de Checklist: `www/cloud/firebase.js` + esbuild (`pnpm build:cloud`) + `firebase-config.js` / `.example.js` + dependencias `firebase` con pnpm. | Bundle reconstruido; app arranca en modo LOCAL sin config. |
| **B. Auth** | Registro libre con correo/contraseña; login con Google, Facebook y X; «Olvidé mi contraseña» (correo de Firebase); recordar sesión; pantalla de login (la app no muestra nada hasta confirmar sesión, como Checklist 2.1.2). | Entrar con los 4 métodos en un teléfono real. |
| **C. Datos en la nube** | Colecciones del §3, `firestore.rules`, `storage.rules`; sustituir `save()` por escrituras a Firestore con caché offline; `localStorage` queda solo como respaldo/migración. | Crear un viaje sin internet y verlo sincronizado al volver. |
| **D. Archivos** | Subida de foto de ticket, constancia y facturas PDF/XML a Storage al guardar; descarga bajo demanda al abrir el gasto (con caché local). | Foto y factura visibles en otro teléfono con la misma cuenta. |
| **E. Migración** | Pantalla, pasos y casos límite del §4; marca `gastos-viaje-migrado-v2`; no borrar sin confirmación. | Instalar la 2.0.0 sobre una 1.20 con viajes reales y comprobar que todo llega a la nube. |
| **F. Planes** | Campo `plan` en `users/`; gating del límite de 3 viajes activos; placeholder de banner de anuncios; admin puede cambiar el plan. | Usuario básica no puede crear el 4.º viaje activo; admin sí puede cambiarle el plan. |
| **G. Admin** | Gerardo como admin: lista de usuarios, desactivar cuentas, cambiar plan, ver todos los viajes. | Probar con la cuenta de Gerardo y una cuenta normal. |
| **H. Cierre** | versionCode **22**, versionName **«2.0.0»**, texto «Versión 2.0.0» en el encabezado; `npx cap sync`; `./gradlew assembleRelease` con el keystore permanente; probar instalación sobre la 1.20 sin perder datos; subir a GitHub (`control-gastos-viaje`) y actualizar `README.md`, `CHANGELOG.md` y `PREGUNTAS-DEVS.md`. | APK 2.0.0 instalada encima de la 1.20 con los viajes intactos. |

**Reglas de construcción (de `~/AGENTS.md` y de Checklist):**

- Después de editar `www/cloud/firebase.js`: `pnpm build:cloud` y
  `npx cap sync` antes de compilar; verificar el md5 del bundle.
- `npx cap sync` siempre tras editar `www/` (nunca `pnpm dlx cap
  sync`: resuelve un paquete ajeno).
- La APK release se firma con el keystore permanente; **nunca** se
  imprimen ni se copian sus contraseñas.
- La API key web de Firebase sí puede ir en el repo (es pública por
  diseño); lo que nunca se versiona: keystore, contraseñas y claves
  de servicios.
- Cada fase se prueba en teléfono real instalando encima de la
  versión anterior, como en todas las 1.x.

### 5.2 Versión 2.1 — lo que queda fuera de la 2.0.0

1. **Viajes compartidos:** llenar `memberUids`, interfaz de invitar /
   quitar participantes, reglas «dueño o miembro» ya preparadas en
   la 2.0.0. Función premium.
2. **Login con Apple:** cuando Gerardo tenga la cuenta de
   desarrollador de Apple; el gancho ya queda preparado en la 2.0.0.
3. **Versión de computadora:** Electron / Firebase Hosting con la
   misma `www/` y el mismo bundle, como Checklist. Reutiliza todo el
   backend de la 2.0.0 sin cambios.
4. **AdMob real y Play Billing real:** conectar el SDK y el cobro
   cuando existan las cuentas (ver §6). El gating y el placeholder ya
   existen desde la 2.0.0.
5. **Respaldo automático programado a Drive** (premium) y, si hace
   falta, edición de gastos existentes.

---

## 6. Pasos externos que necesita Gerardo

### 6.1 Ya hecho

| Paso | Estado |
|---|---|
| Repo GitHub `gcruzhz/control-gastos-viaje` (público, rama `main`) | ✅ Creado por Gerardo el 2026-11-19 |

### 6.2 Lo que necesita su acción (no se puede hacer sin él)

| # | Paso | Qué tiene que hacer Gerardo | Por qué solo él puede |
|---|---|---|---|
| 1 | **Proyecto Firebase** `control-gastos-viaje` | En console.firebase.google.com: crear el proyecto (plan Spark, gratuito), registrar una app web y una app Android (`mx.gerardo.gastosviaje`), activar Authentication (correo/contraseña, Google, Facebook, X), crear la base Firestore en modo producción y el bucket de Storage. | Requiere entrar con su cuenta de Google y aceptar las condiciones. |
| 2 | **SHA-1 del keystore en Firebase** | En el registro de la app Android del proyecto, pegar la huella SHA-1 del keystore release. La huella se puede calcular en el sandbox sin mostrar la contraseña; pegarla en la consola la hace él (o da acceso). | Sin el SHA-1, el login con Google en Android falla. |
| 3 | **App en Meta Developers (Facebook Login)** | En developers.facebook.com: crear una app, activar el producto Facebook Login, anotar el ID y el secreto de la app y pegarlos en Firebase Auth → Facebook. | Requiere su cuenta de Facebook y aceptar las condiciones de Meta. |
| 4 | **App en el portal de desarrolladores de X (OAuth 2.0)** | En developer.x.com: crear un proyecto/app, activar OAuth 2.0, anotar Client ID y Client Secret y pegarlos en Firebase Auth → X (Twitter). | Requiere su cuenta de X; X puede pedir revisión de la app. |
| 5 | **Cuenta de desarrollador de Google Play** ($25, pago único) | Crear la cuenta y pagar. Necesaria para publicar la app y, después, para Play Billing (cobro premium). | Requiere su pago y sus datos fiscales. |
| 6 | **Cuenta de AdMob** | Crear la cuenta en admob.google.com cuando quiera anuncios reales. | Requiere su cuenta y sus datos de pago. |
| 7 | **Cuenta de desarrollador de Apple** ($99/año) | Solo cuando quiera el login con Apple (2.1 o después). | Requiere su pago y su ID de Apple. |

### 6.3 Lo que se puede hacer sin él (en el sandbox)

- Todo el código de la 2.0.0 (fases A–H del §5.1): módulo de nube,
  login, Firestore, Storage, migración, planes, admin.
- `firestore.rules` y `storage.rules` listos para pegar/publicar.
- `firebase-config.example.js` y la estructura de `firebase-config.js`.
- El build de la APK (con el keystore permanente) y la subida del
  código a GitHub.
- Probar la 2.0.0 en modo LOCAL y la migración con datos de prueba
  sin nube.
- Calcular la huella SHA-1 del keystore (sin imprimir contraseñas)
  para que él solo tenga que pegarla.

### 6.4 Dependencias entre pasos externos y fases

- **Fase A–B (base nube y Auth):** se puede escribir todo el código
  sin el proyecto Firebase, usando la config de ejemplo; pero **no se
  puede probar el login real** hasta el paso 1 (y el login con Google
  hasta el paso 2, con Facebook hasta el paso 3, con X hasta el paso 4).
- **Fase C–E (datos, archivos, migración):** se prueban contra el
  proyecto del paso 1. Sin proyecto no hay prueba real en la nube.
- **Fase F (planes):** no depende de ningún paso externo; el gating
  funciona sin AdMob ni Play.
- **Publicación en Play y cobro premium:** dependen de los pasos 5 y 6;
  no bloquean la 2.0.0 como APK directa.

---

## 7. Riesgos y decisiones abiertas

- **El login social es lo más frágil:** cada proveedor (Google,
  Facebook, X) tiene su propio alta y sus propios errores (SHA-1,
  redirecciones, revisión de app en X). Si uno se atrasa, la 2.0.0
  puede salir con correo + los proveedores que ya funcionen, y el
  resto en una 2.0.x. El registro con correo siempre funciona.
- **Migración con muchos archivos:** un viaje con muchas fotos y
  facturas puede tardar en subirse; la pantalla de migración debe
  mostrar progreso y permitir continuar en segundo plano, no bloquear
  la app.
- **Costo de Firebase:** plan Spark (gratuito) cubre el uso esperado
  de una app personal; si los archivos en Storage crecen mucho, se
  revisa el plan Blaze (pago por uso) con Gerardo antes de cambiar.
- **Decisión abierta (no bloquea la 2.0.0):** si los viajes
  compartidos de la 2.1 serán solo premium o también con un límite en
  básica (p. ej. 1 viaje compartido). Se decide con Gerardo antes de
  la 2.1.
