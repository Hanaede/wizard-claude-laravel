# Wizard: Nuevo Proyecto Laravel 13 + React

Eres el asistente de setup de un nuevo proyecto web. Tu misión es instalar y configurar completamente el stack Laravel 13 + React en la carpeta actual, crear el repositorio en tu GitHub personal (rama principal **main**) y dejarlo listo para trabajar en el servidor remoto vía SSH.

Sigue estas fases en orden. No avances a la siguiente fase sin completar la anterior. Informa al usuario de cada paso que ejecutas.

> **Regla fundamental:** Cualquier dato que el usuario proporcione (dominio, base de datos, cuenta de correo, credenciales SSH) significa que ya existe en el servidor. Nunca pidas al usuario que cree algo que ya ha dado como dato.

---

## COMPROBACIÓN PREVIA — Carpeta del proyecto

Antes de hacer nada, verifica que VS Code está abierto en la carpeta vacía del nuevo proyecto:

```bash
pwd
ls -A
```

Si la carpeta **no está vacía** (contiene archivos o directorios que no sean `.git`), detente y muestra este mensaje:

```
⚠ La carpeta actual no está vacía.

Este wizard debe ejecutarse desde la carpeta vacía del nuevo proyecto,
ya que instalará Laravel y todos los archivos del proyecto directamente aquí.

Pasos para continuar:
1. Crea una carpeta vacía con el nombre de tu proyecto
2. Ábrela en VS Code (File → Open Folder)
3. Abre Claude Code en esa carpeta y vuelve a ejecutar /nuevo-proyecto

No es posible continuar sin una carpeta vacía.
```

**No avances hasta que la carpeta esté vacía.** Si el usuario confirma que ha abierto la carpeta correcta, vuelve a comprobarlo con `ls -A` antes de continuar.

Si la carpeta está vacía, muestra la ruta y pide confirmación:

```
Carpeta del proyecto: [ruta actual]

¿Es esta la carpeta correcta donde instalar el proyecto? (escribe "sí" para continuar)
```

---

## FASE 0 — Verificación de herramientas

Ejecuta estos comandos. Si algo falla, indica al usuario cómo instalarlo y espera confirmación:

```bash
php --version        # debe ser >= 8.2
composer --version
node --version       # debe ser >= 18
npm --version
git --version
gh --version
```

Si `laravel` no está disponible:
```bash
composer global require laravel/installer
export PATH="$PATH:$HOME/AppData/Roaming/Composer/vendor/bin"
```

### Verificación y conexión de GitHub

Comprueba si el usuario ya está autenticado en GitHub:

```bash
gh auth status
```

**Si el comando falla o muestra que no hay sesión activa**, sigue estos pasos de forma guiada, uno a uno, esperando confirmación en cada uno:

**Paso 1 — Iniciar autenticación:**
```bash
gh auth login
```
Cuando se ejecute, selecciona estas opciones:
- `GitHub.com` (no GitHub Enterprise)
- `HTTPS` como protocolo preferido
- `Login with a web browser` para autenticarte

**Paso 2 — Verificar autenticación:**
```bash
gh auth status
```
Debe mostrar el nombre de usuario y el token activo. Si sigue fallando, pide al usuario que revise el navegador y reintente.

**Paso 3 — Confirmar usuario de GitHub:**
```bash
gh api user --jq '.login'
```
Muestra el resultado al usuario y pide confirmación: "¿Es este tu usuario de GitHub correcto? (escribe 'sí' para continuar o 'no' para volver a autenticarte)"

Una vez confirmado, guarda el nombre de usuario de GitHub para usarlo en la creación del repositorio.

---

## FASE 1 — Recopilación de datos

Haz estas preguntas **de una en una**, esperando respuesta antes de pasar a la siguiente.

1. **Nombre del proyecto** (se usará como identificador en todo el stack)

2. **Nombre del repo GitHub** — default: nombre del proyecto en kebab-case

3. ~~Visibilidad~~ — siempre **privado**.

4. **Dominio de producción** — pregunta exactamente así:
   "¿En qué dominio va a vivir este proyecto?
   1) Dominio principal (ej: midominio.com)
   2) Subdominio (ej: app.midominio.com)
   3) Subdirectorio (ej: midominio.com/proyecto)
   Escribe el número y el dominio completo."

   Deriva `APP_URL` = `https://[dominio]`.

4b. **Correo electrónico del proyecto** — pregunta:
   - Dirección de correo que se usará como remitente (ej: `hola@midominio.com`)
   - Contraseña de esa cuenta de correo

   Muestra este aviso y espera confirmación antes de continuar:
   ```
   Necesitas tener creada la cuenta de correo en tu panel de hosting:
   → Cuenta: [correo introducido]

   Si ya la tienes creada, escribe "listo". Si necesitas crearla, hazlo ahora en tu panel de hosting y después escribe "listo".
   ```

   Una vez confirmado, pregunta:
   - Servidor SMTP (ej: `mail.midominio.com`)
   - Puerto SMTP (ej: `465` o `587`)

   Deduce el cifrado automáticamente: puerto 465 → `ssl`, cualquier otro → `tls`.

5. **SSH Servidor remoto** — pide uno a uno:
   - Host SSH (ej: `server123.hosting.net`, `ssh.gb.stackcp.com`)
   - Usuario SSH
   - Puerto SSH (default: `22` o preguntar al hosting)

   Tras recibir los tres datos, detecta el home remoto automáticamente:
   ```bash
   REMOTE_HOME=$(ssh -p [puerto] [usuario]@[host] "echo \$HOME")
   ```
   
   **Nota sobre estructura del hosting:**
   - Algunos hostings (Siteground): Dominio principal → `$REMOTE_HOME/public_html`, otros dominios → `$REMOTE_HOME/www/[dominio-completo]`
   - Otros hostings (20i): carpetas específicas del hosting (ej: `/home/sites/7b/0/0e668576c6/app`)
   
   **Siempre pregunta al usuario la ruta remota exacta después de detectar el home remoto.**
   Muestra la ruta calculada (si aplica) y permite que el usuario corrija si es necesario.

6. **Base de datos** (proporciona los datos de la BD ya creada en tu hosting):
   - Host (default: `localhost`)
   - Nombre
   - Usuario
   - Contraseña — guardarla **siempre entre comillas dobles** en los archivos .env para evitar problemas con caracteres especiales

7. **Correo del administrador** — pregunta:
   - Correo electrónico del usuario administrador del proyecto
   - Contraseña para ese usuario administrador

8. **Versión de Laravel** — consulta Packagist y si hay versión mayor a 13 pregunta si quiere usarla. Default: Laravel 13.

---

## FASE 2 — Instalación de Laravel 13 + React starter kit

Informa: "Instalando Laravel 13 con el starter kit React oficial..."

**ADVERTENCIA sobre rutas con caracteres especiales en Windows:**
Si la ruta local contiene `#`, `%`, `&` u otros caracteres especiales, el build de Vite puede fallar con un error de null bytes en la ruta. El workaround es sustituir el plugin `@tailwindcss/vite` por `@tailwindcss/postcss` (ver más abajo). Aplícalo siempre que el build falle con ese error.

La carpeta actual puede estar vacía pero existir (VS Code la tiene abierta). Instala desde el directorio padre:

```bash
CURRENT_DIR=$(pwd)
FOLDER_NAME=$(basename "$CURRENT_DIR")
PARENT_DIR=$(dirname "$CURRENT_DIR")

cd "$PARENT_DIR"
laravel new "$FOLDER_NAME" --react --no-interaction
cd "$CURRENT_DIR"
```

Si falla con "Application already exists", usa `--force`. Si falla igualmente (archivo bloqueado por VS Code), usa carpeta temporal:
```bash
cd "$PARENT_DIR"
laravel new "__temp_laravel_setup__" --react --no-interaction
cp -r __temp_laravel_setup__/. "$FOLDER_NAME/"
rm -rf __temp_laravel_setup__
cd "$CURRENT_DIR"
```

### Workaround build Windows (aplicar siempre)

Instala `@tailwindcss/postcss` y quita el plugin `@tailwindcss/vite` del `vite.config.ts`:

```bash
npm install -D @tailwindcss/postcss
```

Crea `postcss.config.js`:
```js
export default {
    plugins: {
        '@tailwindcss/postcss': {},
    },
};
```

En `vite.config.ts`, elimina la línea `import tailwindcss from '@tailwindcss/vite';` y elimina `tailwindcss()` de la lista de plugins.

Luego:
```bash
npm install
npm install motion
npm run build   # debe completar sin errores
```

---

## FASE 3 — Configuración de entornos

**No hay servidor local.** El `.env` apunta directamente a producción.

Actualiza `.env` con los datos recogidos:
```
APP_NAME="[nombre-del-proyecto]"
APP_ENV=production
APP_DEBUG=false
APP_URL=[app-url]

LOG_CHANNEL=stack
LOG_LEVEL=error

DB_CONNECTION=mysql
DB_HOST=[host-db]
DB_PORT=3306
DB_DATABASE=[nombre-db]
DB_USERNAME=[usuario-db]
DB_PASSWORD="[password-db]"

SESSION_DRIVER=database
QUEUE_CONNECTION=database
CACHE_STORE=database

MAIL_MAILER=smtp
MAIL_HOST=[smtp-host]
MAIL_PORT=[smtp-port]
MAIL_USERNAME=[correo-remitente]
MAIL_PASSWORD="[password-correo]"
MAIL_ENCRYPTION=[ssl|tls]
MAIL_FROM_ADDRESS=[correo-remitente]
MAIL_FROM_NAME="${APP_NAME}"
```

El `.env` no se sube a GitHub (ya lo excluye Laravel por defecto).

---

## FASE 4 — Git + GitHub

**IMPORTANTE:** Usa siempre **"main"** como rama principal, no "master".

Recupera el usuario de GitHub autenticado (obtenido en Fase 0) y crea el repositorio en la cuenta personal:

```bash
GITHUB_USER=$(gh api user --jq '.login')

git init -b main
git add .
git commit -m "chore: initial Laravel 13 + React setup"
gh repo create "$GITHUB_USER/[nombre-repo]" --private --source=. --remote=origin --push
```

Informa la URL del repo creado (`https://github.com/[usuario]/[nombre-repo]`) y confirma que está en la rama **main**.

---

## FASE 5 — Generar archivos de soporte

### deploy.sh

`rsync` no está disponible en Git Bash para Windows. El deploy usa `tar + scp`, que sí está disponible:

```bash
#!/bin/bash
set -e

SSH_USER="[usuario-ssh]"
SSH_HOST="[host-ssh]"
SSH_PORT="[puerto-ssh]"
REMOTE_PATH="[ruta-remota]"
TMP_ARCHIVE="/tmp/deploy-[nombre-repo].tar.gz"

echo "Compilando assets..."
npm run build

echo "Empaquetando archivos..."
tar --exclude='.git' \
    --exclude='node_modules' \
    --exclude='.env' \
    --exclude='storage/logs' \
    --exclude='storage/framework/cache' \
    --exclude='storage/framework/sessions' \
    --exclude='storage/framework/views' \
    -czf "$TMP_ARCHIVE" .

echo "Subiendo paquete al servidor..."
scp -P "$SSH_PORT" "$TMP_ARCHIVE" "$SSH_USER@$SSH_HOST:$REMOTE_PATH/deploy.tar.gz"

echo "Extrayendo en el servidor..."
ssh -p "$SSH_PORT" "$SSH_USER@$SSH_HOST" "cd $REMOTE_PATH && tar -xzf deploy.tar.gz && rm deploy.tar.gz"

echo "Ejecutando migraciones..."
ssh -p "$SSH_PORT" "$SSH_USER@$SSH_HOST" "cd $REMOTE_PATH && php artisan migrate --force"

echo "Limpiando caché..."
ssh -p "$SSH_PORT" "$SSH_USER@$SSH_HOST" "cd $REMOTE_PATH && php artisan config:cache && php artisan route:cache && php artisan view:cache"

rm -f "$TMP_ARCHIVE"
echo "Deploy completado en $REMOTE_PATH"
```

```bash
chmod +x deploy.sh
```

### CLAUDE.md

```markdown
# [nombre-del-proyecto]

## Stack
- Laravel 13 + React 19 (Inertia 2 + TypeScript + Tailwind 4 + shadcn/ui)
- Producción: [app-url]
- Hosting: [nombre-hosting]
- Repo: https://github.com/[usuario-github]/[nombre-repo]
- Rama principal: **main**

## Regla principal de vibe coding
Tras CADA cambio de código que se te pida, sin excepción:
1. Ejecuta `./deploy.sh` para subir los cambios al servidor
2. Ejecuta `git add -A && git commit -m "feat/fix/chore: [descripción]" && git push`

No esperes a que el usuario lo pida. Es parte del flujo estándar.

## Añadir componentes de animate-ui
```bash
npx shadcn@latest add "https://animate-ui.com/r/[nombre-componente].json"
```
El componente quedará en `resources/js/components/animate-ui/` listo para importar.
shadcn/ui y motion ya están instalados.

## Nota sobre el build
Usa `@tailwindcss/postcss` vía PostCSS en lugar del plugin `@tailwindcss/vite` (workaround para un bug de rolldown/Vite con rutas Windows que contienen `#`). No revertir.

## Estructura
- `resources/js/pages/` — Páginas Inertia
- `resources/js/components/` — Componentes React
- `resources/js/layouts/` — Layouts
- `app/Http/Controllers/` — Controladores Laravel
- `routes/web.php` — Rutas
- `database/migrations/` — Migraciones

## Despliegue SSH
- Script: `./deploy.sh`
- SSH: [usuario-ssh]@[host-ssh]:[puerto-ssh]
- Ruta remota: [ruta-remota]
```

---

## FASE 6 — Primera subida a producción

```bash
# Crear directorio y subir .env
ssh -p [puerto] [usuario]@[host] "mkdir -p [ruta-remota]"
scp -P [puerto] .env [usuario]@[host]:[ruta-remota]/.env

# Primer deploy completo
./deploy.sh

# Inicialización del servidor (solo primera vez)
# IMPORTANTE: crear los directorios de storage antes de ejecutar artisan,
# o view:cache fallará con "View path not found"
ssh -p [puerto] [usuario]@[host] "
  cd [ruta-remota] &&
  mkdir -p storage/framework/views storage/framework/cache storage/framework/sessions storage/logs &&
  chmod -R 775 storage bootstrap/cache &&
  php artisan key:generate &&
  php artisan storage:link &&
  php artisan config:cache &&
  php artisan route:cache &&
  php artisan view:cache
"

# Symlink public_html → public
# Algunos hostings usan public_html como document root, pero Laravel usa public/.
# La solución es eliminar la carpeta public_html (si el hosting la crea por defecto)
# y crear un symlink que apunte a public/.
ssh -p [puerto] [usuario]@[host] "
  rm -rf [ruta-remota]/public_html &&
  ln -s [ruta-remota]/public [ruta-remota]/public_html
"

# Crear usuario administrador con los datos recogidos en Fase 1
ssh -p [puerto] [usuario]@[host] "
  cd [ruta-remota] &&
  php artisan tinker --execute=\"
    use App\\\Models\\\User;
    if (!User::where('email', '[correo-admin]')->exists()) {
      User::create([
        'name' => 'Administrador',
        'email' => '[correo-admin]',
        'password' => bcrypt('[password-admin]'),
        'email_verified_at' => now(),
      ]);
      echo 'Usuario admin creado.';
    } else {
      echo 'El usuario ya existia.';
    }
  \"
"

# Enviar correo de prueba para verificar configuración SMTP
# NOTA: usar php con script temporal en lugar de tinker directo via SSH,
# ya que los escapes de shell anidados (SSH → bash → tinker → PHP) corrompen
# caracteres especiales en strings. Es más fiable y predecible.
ssh -p [puerto] [usuario]@[host] "
  cat > /tmp/test_mail_[nombre-repo].php << 'PHPEOF'
<?php
require '[ruta-remota]/vendor/autoload.php';
\$app = require_once '[ruta-remota]/bootstrap/app.php';
\$kernel = \$app->make(Illuminate\Contracts\Http\Kernel::class);
\$app->make('config');
\$mailer = \$app->make('mailer');
\$mailer->raw(
    'Correo de prueba del proyecto [nombre-del-proyecto]. Si recibes este mensaje, el SMTP está correctamente configurado.',
    function(\$m) { \$m->to('[correo-admin]')->subject('Prueba SMTP - [nombre-del-proyecto]'); }
);
echo 'Correo enviado.';
PHPEOF
  php /tmp/test_mail_[nombre-repo].php
  rm /tmp/test_mail_[nombre-repo].php
"
```

---

## FASE 7 — Verificación final

### Verificación automática

Ejecuta estos checks desde local:

```bash
# 1. Verificar que la web responde con HTTP 200
HTTP_STATUS=$(curl -o /dev/null -s -w "%{http_code}" --max-time 15 "[app-url]")
if [ "$HTTP_STATUS" = "200" ]; then
  echo "✓ Web accesible: [app-url] → HTTP $HTTP_STATUS"
else
  echo "✗ La web devolvió HTTP $HTTP_STATUS (esperado 200)"
fi

# 2. Verificar que la página de login carga correctamente
LOGIN_STATUS=$(curl -o /dev/null -s -w "%{http_code}" --max-time 15 "[app-url]/login")
if [ "$LOGIN_STATUS" = "200" ]; then
  echo "✓ Página de login accesible: [app-url]/login → HTTP $LOGIN_STATUS"
else
  echo "✗ La página de login devolvió HTTP $LOGIN_STATUS (esperado 200)"
fi
```

Si algún check falla, diagnostica antes de continuar:
- HTTP 500 → revisar logs: `ssh -p [puerto] [usuario]@[host] "tail -50 [ruta-remota]/storage/logs/laravel.log"`
- HTTP 404 → verificar que el symlink `public_html → public/` esté creado correctamente
- Sin respuesta / timeout → verificar que el dominio apunta al servidor correcto

### Resumen final

Informa al usuario:

```
✓ PROYECTO CONFIGURADO CORRECTAMENTE
=====================================
Proyecto:   [nombre-del-proyecto]
URL:        [app-url]
Repo:       https://github.com/[usuario-github]/[nombre-repo]
Admin:      [correo-admin]
Deploy:     ./deploy.sh

Web:        [app-url] → HTTP [status]
Login:      [app-url]/login → HTTP [status]

Accede a [app-url]/login con [correo-admin] para verificar
que el login funciona correctamente.

Con cada cambio que pidas, subiré los archivos al servidor
y haré commit+push automáticamente.
```

---

## Aprendizajes documentados (lecciones del proceso)

Esta sección recoge errores reales encontrados durante el desarrollo de este wizard. Léela antes de hacer cambios al wizard para no repetir los mismos problemas.

### 1. Datos proporcionados por el usuario = ya existen en el servidor
Cuando el usuario da un dominio, base de datos, o cuenta SSH, significa que ya están creados. Nunca mostrar mensajes como "necesitas crear la base de datos" ni pedir que se cree algo que el usuario ya proporcionó como dato.

### 2. Ruta remota en Siteground
Siteground organiza los dominios así:
- Dominio principal del hosting → `~/public_html`
- Cualquier otro dominio/subdominio → `~/www/[nombre-completo-del-dominio]`
  Ejemplo: `stack-test.midominio.com` → `~/www/stack-test.midominio.com`

Obtener `$HOME` automáticamente vía SSH antes de calcular la ruta, nunca asumirla.

### 3. tar+scp en lugar de rsync
`rsync` no está disponible en Git Bash para Windows (ni siquiera con Chocolatey si no hay permisos de administrador). Usar siempre `tar --exclude... -czf` + `scp` + `ssh tar -xzf`. Es más lento pero funciona en todos los entornos Windows.

### 4. Workaround PostCSS para rutas Windows con caracteres especiales
El plugin `@tailwindcss/vite` falla con un error de "null bytes en la ruta" cuando el path del proyecto contiene `#`, `%`, `&` u otros caracteres especiales (bug de rolldown 1.0+). La solución es:
- Instalar `@tailwindcss/postcss`
- Crear `postcss.config.js` con ese plugin
- Eliminar `tailwindcss()` de `vite.config.ts`

Aplicar siempre en proyectos Windows. No intentar renombrar la carpeta del proyecto como alternativa — es un workaround estable.

### 5. Directorios de storage en el servidor (primera vez)
En un servidor vacío, `storage/framework/views`, `storage/framework/cache`, `storage/framework/sessions` y `storage/logs` no existen hasta que los crea Laravel. Si se ejecuta `php artisan view:cache` antes de crearlos manualmente, falla con "View path not found". Siempre ejecutar `mkdir -p` de estos directorios antes de los comandos artisan en el primer deploy.

### 6. public_html en hostings compartidos
Algunos hostings (como Siteground) crean automáticamente una carpeta `public_html` como document root. Laravel usa `public/`. La solución es:
1. `rm -rf public_html` (elimina la carpeta vacía del hosting)
2. `ln -s [ruta-remota]/public [ruta-remota]/public_html` (symlink)

Esto permite que el hosting apunte a `public_html` y Laravel sirva desde `public/` sin configuración adicional en el panel.

### 7. Escapes de shell en comandos SSH anidados
Ejecutar `tinker` directamente vía SSH con código PHP complejo (especialmente con comillas, backslashes o caracteres especiales en strings) es propenso a corrupción de escapes. Para operaciones como enviar correos de prueba o ejecutar código PHP con strings complejos, es más fiable:
1. Escribir un script PHP en `/tmp/` en el servidor con heredoc (`<< 'PHPEOF'`)
2. Ejecutarlo con `php /tmp/script.php`
3. Eliminarlo tras su ejecución

### 8. Contraseñas en .env siempre entre comillas dobles
Las contraseñas en archivos `.env` deben guardarse siempre entre comillas dobles (`"contraseña"`). Caracteres como `&`, `<`, `>`, `$`, `;` son problemáticos sin comillas. Nunca usar `sed` para escribir contraseñas en `.env` — usar el editor de texto directamente (herramienta Edit) para evitar que `sed` interprete `&` como referencia al match.

### 9. Autenticación de GitHub con gh CLI
Si `gh auth status` falla, el flujo correcto es `gh auth login` seleccionando GitHub.com + HTTPS + Login with a web browser. Tras autenticarse, verificar siempre con `gh api user --jq '.login'` antes de crear el repositorio, para asegurarse de que el repo se crea en la cuenta correcta.
