# passportREADME

# Auth Discord Google

### 1. **Instalar las dependencias necesarias**

Comienza instalando las dependencias con los siguientes comandos:

```bash
bash
Copiar código
npm install passport passport-google-oidc express-session

```

- `passport`: Módulo de autenticación.
- `passport-google-oidc`: Estrategia de autenticación para usar OpenID Connect (OIDC) de Google.
- `express-session`: Módulo para manejar sesiones de usuario en Express.

### 2. **Crear una cuenta en Google Developers Console**

1. Ve a la Google Developers Console.
2. Crea un proyecto nuevo o selecciona uno existente.
3. Habilita la API de **Google+** o **People API** para obtener acceso a la autenticación de Google.
4. En **Credenciales** → **Crear credenciales**, selecciona **ID de cliente de OAuth 2.0**.
5. Completa los campos necesarios:
    - Tipo de aplicación: "Aplicación web".
    - URL de redirección: Debe ser algo como `http://localhost:3000/auth/google/callback` para el desarrollo local.
6. Guarda el **Client ID** y **Client Secret** que Google te proporcionará.

### 3. **Configurar Passport y Google OIDC Strategy**

Crea un archivo en tu proyecto, por ejemplo `auth.js`, donde configurarás `passport` con la estrategia de Google.

```
js
Copiar código
const passport = require('passport');
const GoogleOIDCStrategy = require('passport-google-oidc');

passport.use(new GoogleOIDCStrategy({
    clientID: process.env.GOOGLE_CLIENT_ID,    // Tu Google Client ID
    clientSecret: process.env.GOOGLE_CLIENT_SECRET, // Tu Google Client Secret
    callbackURL: 'http://localhost:3000/auth/google/callback', // URL de redirección
    scope: ['profile', 'email'], // Información a solicitar al usuario
  },
  (issuer, profile, done) => {
    // Aquí manejas la creación o búsqueda del usuario en tu base de datos.
    return done(null, profile);
  }
));

// Serialización y deserialización del usuario (para sesiones)
passport.serializeUser((user, done) => {
  done(null, user);
});

passport.deserializeUser((obj, done) => {
  done(null, obj);
});

```

### 4. **Configurar Express y rutas para autenticación**

En tu archivo principal (por ejemplo, `app.js` o `server.js`), configura Express y Passport para manejar las sesiones y las rutas de autenticación.

```
js
Copiar código
const express = require('express');
const session = require('express-session');
const passport = require('passport');
require('./auth');  // Archivo donde configuraste la estrategia de Passport

const app = express();

// Configurar sesiones
app.use(session({ secret: 'your_secret_key', resave: false, saveUninitialized: true }));

// Inicializar Passport
app.use(passport.initialize());
app.use(passport.session());

// Ruta para iniciar el proceso de autenticación con Google
app.get('/auth/google',
  passport.authenticate('google', { scope: ['profile', 'email'] })
);

// Ruta de callback después de que Google haya autenticado al usuario
app.get('/auth/google/callback',
  passport.authenticate('google', { failureRedirect: '/' }),
  (req, res) => {
    // Autenticación exitosa, redirigir a tu página protegida
    res.redirect('/profile');
  }
);

// Ruta protegida (solo accesible si el usuario está autenticado)
app.get('/profile', (req, res) => {
  if (req.isAuthenticated()) {
    res.send(`Hola, ${req.user.displayName}`);
  } else {
    res.redirect('/auth/google');
  }
});

// Ruta para cerrar sesión
app.get('/logout', (req, res) => {
  req.logout((err) => {
    if (err) { return next(err); }
    res.redirect('/');
  });
});

app.listen(3000, () => {
  console.log('Servidor corriendo en http://localhost:3000');
});

```

### 5. **Agregar variables de entorno**

En lugar de poner las claves directamente en el código, utiliza variables de entorno para almacenar el `Client ID` y el `Client Secret`. Crea un archivo `.env` en la raíz de tu proyecto:

```makefile
makefile
Copiar código
GOOGLE_CLIENT_ID=tu-client-id
GOOGLE_CLIENT_SECRET=tu-client-secret

```

Y en tu código, usa `dotenv` para cargar las variables de entorno:

```bash
bash
Copiar código
npm install dotenv

```

Y agrega lo siguiente en el archivo principal de tu aplicación:

```
js
Copiar código
require('dotenv').config();

```

### 6. **Probar la aplicación**

1. Inicia tu aplicación:

```bash
bash
Copiar código
node app.js

```

1. Abre tu navegador y visita `http://localhost:3000/auth/google` para probar la autenticación con Google.

### Resumen de los pasos:

1. Instala las dependencias: `passport`, `passport-google-oidc`, `express-session`.
2. Configura un proyecto en Google Developers Console y obtén las credenciales.
3. Configura la estrategia de Google OIDC en `passport`.
4. Define las rutas para autenticación con Google y manejo de sesiones en Express.
5. Carga las credenciales desde variables de entorno.
6. Ejecutar la aplicación y prueba el flujo de autenticación.