# MusicApp – Gestión de Canciones (Kodular + Firebase)

Aplicación móvil Android desarrollada en **Kodular** para el **Ejercicio N° 20 – Canciones**.
Permite registrar usuarios, iniciar sesión (con contraseña o huella digital), recuperar la clave por correo, administrar un catálogo de canciones (CRUD) y generar reportes parametrizados. Los datos se guardan en **Firebase** (Authentication + Realtime Database).

| Dato | Valor |
|---|---|
| Estudiante | Oscar Mercado |
| Email institucional | omercador@unicartagena.edu.co |
| Ejercicio | 20 – Canciones |
| Actividad | Unidad 1 – Aplicación móvil en Kodular con sensores |
| Video explicativo | _Pendiente_ |

---

## 1. Funcionalidades

| Requerimiento | Cómo se cumple |
|---|---|
| Base de datos en internet | Firebase Realtime Database (NoSQL, plan gratuito Spark) |
| CRUD tabla **Canciones** | Pantalla *Canciones*: crear, listar, editar y eliminar |
| CRUD tabla **Usuario** | *Registro* (crear) y *Mi perfil* (ver, editar, eliminar cuenta) |
| 2+ reportes por tabla | Canciones: por ritmo, por rango de fechas y por duración. Usuarios: por username exacto y por nombre que empieza por… |
| Login | Firebase Authentication (correo y contraseña) |
| Recordatorio de clave por email | Botón "¿Olvidaste la contraseña?": Firebase envía un enlace de restablecimiento al correo |
| Sensor 1 | **Huella digital / biometría** (`KodularBiometricPrompt`): entrar sin escribir la contraseña |
| Sensor 2 | **Acelerómetro**: agitar el celular en *Canciones* recarga la lista |

### Modelo de datos

**Usuario** → `usuarios/<uid>`

| Campo | Tipo | Nota |
|---|---|---|
| username | texto | |
| nombre | texto | |
| email | texto | |
| password | – | No se guarda en la base de datos: la administra Firebase Authentication, cifrada |

**Canciones** → `canciones/<id>`

| Campo | Tipo | Nota |
|---|---|---|
| id | texto | Clave generada por Firebase |
| nombre | texto | |
| ritmo | texto | Rock, Pop, Salsa, Vallenato… |
| duracion | número | En segundos |
| album | texto | |
| posicionEnAlbum | número | |
| banda | texto | |
| interprete | texto | |
| autor | texto | |
| fechaLanzamiento | texto | Formato `AAAA-MM-DD` |

---

## 2. Contenido del repositorio

```
├── README.md
├── AppCancionesFirebase.aia     ← código fuente (proyecto Kodular)
├── apk/
│   └── AppCancionesFirebase.apk ← instalable para Android
└── firebase/
    └── database.rules.json      ← reglas de seguridad de la base de datos
```

---

## 3. Instalar y probar la app (APK)

**Requisitos:** celular Android 6.0 o superior con conexión a internet. Para entrar con huella, el celular debe tener una huella o un bloqueo de pantalla configurado.

1. Descarga `apk/AppCancionesFirebase.apk` desde este repositorio y pásalo al celular.
2. Ábrelo. Si Android lo pide, permite **Instalar apps de origen desconocido** para el navegador o el administrador de archivos.
3. Instala y abre **MusicApp**.
4. Toca **Registrarse**, completa nombre, username, correo y contraseña (mínimo 6 caracteres) y toca **Crear cuenta**.
5. Inicia sesión con tu correo y contraseña. Desde el menú puedes:
   - **Canciones**: agregar canciones (elige la fecha con el botón de fecha), tocar una de la lista para editarla o eliminarla y **agitar el celular** para actualizar la lista.
   - **Mi perfil**: editar tu username y nombre, o eliminar tu cuenta.
   - **Reportes de canciones / usuarios**: elegir el parámetro y ver los resultados.
6. Cierra sesión y vuelve a abrir la app: aparece **Entrar con huella digital**.
7. Para probar la recuperación de clave: en la pantalla de inicio escribe tu correo y toca **¿Olvidaste la contraseña?**. Revisa tu bandeja de entrada (y la carpeta de spam).

---

## 4. Abrir el código fuente en Kodular

1. Clona el repositorio:
   ```bash
   git clone https://github.com/Chromnos05/Desarrollo_de_apps_unidad_1_Oscar_Mercado.git
   ```
2. Entra a <https://creator.kodular.io> con tu cuenta.
3. **Import project** → selecciona `AppCancionesFirebase.aia`.
4. Para probar sin compilar: **Test → Connect to Companion** y escanea el código QR con la app *Kodular Companion*.
   > La huella digital no funciona en el emulador; usa un celular real.
5. Para generar el APK: **Export → Android App (.apk)**.

La app ya trae configurado el proyecto de Firebase del autor, así que funciona sin cambios.

---

## 5. (Opcional) Usar tu propio proyecto de Firebase

1. En <https://console.firebase.google.com> crea un proyecto.
2. **Authentication → Comenzar → Método de acceso → Correo electrónico/contraseña → Habilitar**.
   En **Plantillas** cambia el idioma a español para el correo de restablecimiento de contraseña.
3. **Realtime Database → Crear base de datos** (modo bloqueado). En la pestaña **Reglas**, pega el contenido de `firebase/database.rules.json`:
   ```json
   {
     "rules": {
       "usuarios": {
         ".read": "auth != null",
         ".indexOn": ["username", "nombre", "email"],
         "$uid": { ".write": "auth.uid === $uid" }
       },
       "canciones": {
         ".read": "auth != null",
         ".write": "auth != null",
         ".indexOn": ["ritmo", "banda", "interprete", "album", "autor", "duracion", "fechaLanzamiento"]
       }
     }
   }
   ```
4. En **Configuración del proyecto → General**, copia la **Clave de API web**. La URL de la base de datos aparece en la parte superior de Realtime Database.
5. En Kodular, abre la vista **Blocks** y reemplaza las variables globales:
   - `API_KEY` en las pantallas *Screen1*, *Registro* y *Perfil*.
   - `DB_URL` en *Registro*, *Canciones*, *Perfil*, *ReporteCanciones* y *ReporteUsuarios* (termina en `/`).

---

## 6. Cómo funciona por dentro

- La app usa el componente **Web** de Kodular para llamar a las **API REST** de Firebase:
  - `accounts:signUp`, `accounts:signInWithPassword`, `accounts:sendOobCode` (correo de recuperación) y `accounts:delete` de Identity Toolkit.
  - `securetoken.googleapis.com/v1/token` para renovar la sesión al entrar con huella.
  - `https://<proyecto>.firebaseio.com/<nodo>.json?auth=<idToken>` con `GET`, `POST`, `PUT` y `DELETE` para el CRUD.
- Los **reportes** usan consultas de Firebase en la URL (`orderBy` + `equalTo`, o `startAt`/`endAt`), apoyadas en los índices `.indexOn` de las reglas.
- La sesión (token, uid y correo) se guarda en el dispositivo con **TinyDB**. El token dura 1 hora: si vence, la app vuelve a la pantalla de inicio.
- Las reglas de seguridad solo permiten leer y escribir a usuarios autenticados, y cada usuario solo puede modificar su propio registro en `usuarios`.

## 7. Tecnologías

Kodular Creator · Firebase Authentication · Firebase Realtime Database · API REST (JSON) · TinyDB · Sensores: biometría y acelerómetro.
