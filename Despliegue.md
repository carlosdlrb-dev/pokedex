# Despliegue de PokeDex en Azure Static Web Apps

Este documento explica paso a paso cómo se publicó la aplicación PokeDex (Angular 14) en **Azure Static Web Apps** (plan **Free**) usando la suscripción **Azure for Students**, con despliegue continuo desde GitHub Actions y encabezados HTTP de seguridad.

## Arquitectura

```
GitHub (repo pokedex, rama main)
        │  git push
        ▼
GitHub Actions ── npm ci + ng build ──► dist/pokedex-angular
        │  Azure/static-web-apps-deploy (token de despliegue)
        ▼
Azure Static Web Apps (CDN global + HTTPS automático)
        │  staticwebapp.config.json → encabezados de seguridad
        ▼
https://<nombre>.azurestaticapps.net
```

## 1. Preparar el código fuente

### 1.1 Clonar y subir el código al repositorio `pokedex`

```bash
git clone https://github.com/keilermora/pokedex-angular.git
cd pokedex-angular
git remote set-url origin https://github.com/<usuario>/pokedex.git
git push -u origin main
```

### 1.2 Ajustes realizados al proyecto para Azure

El proyecto original estaba pensado para GitHub Pages (subruta `/pokedex-angular/`). Se hicieron estos cambios:

| Archivo | Cambio | Motivo |
|---|---|---|
| `src/environments/environment.prod.ts` | `imagesPath: '/assets/images'` | En Azure la app se sirve desde la raíz `/`, no desde `/pokedex-angular/`. |
| `src/index.html` | Se eliminó el script inline "Single Page Apps for GitHub Pages" | Ya no es necesario (Azure maneja las rutas SPA) y un script inline obliga a usar `'unsafe-inline'` en la CSP. |
| `angular.json` | Se quitó `src/404.html` de los assets y se agregó `src/staticwebapp.config.json` | El `404.html` era el truco de GitHub Pages; el archivo de configuración de Azure debe quedar dentro de `dist/`. |
| `angular.json` | `optimization.styles.inlineCritical: false` | Angular inserta `onload="this.media='all'"` en el HTML, que la CSP sin `'unsafe-inline'` en scripts bloquearía. |
| `src/staticwebapp.config.json` | Nuevo | Rutas SPA y encabezados de seguridad (ver sección 4). |
| `.github/workflows/azure-static-web-apps.yml` | Nuevo | Compila y despliega en cada `push` a `main`. |

### 1.3 Probar la compilación en local

```bash
npm ci
npx ng build --configuration production
```

El resultado queda en `dist/pokedex-angular/` (debe contener `index.html` y `staticwebapp.config.json`).

## 2. Crear el recurso Static Web App en Azure

1. Entrar a <https://portal.azure.com> con la cuenta Azure for Students.
2. Buscar **"Static Web Apps"** → **"+ Crear"**.
3. Pestaña **Básico**:
   - **Suscripción:** `Azure for Students`
   - **Grupo de recursos:** `rg-pokedex`
   - **Nombre:** `pokedex` (o similar)
   - **Tipo de plan:** `Free`
   - **Región para Azure Functions API:** `East US 2`
   - **Origen de la implementación:** **`Other`** (el workflow ya está incluido en el repo; así Azure no crea uno propio).
4. **Revisar y crear → Crear**. Esperar a que termine y pulsar **"Ir al recurso"**.
5. En la página **Información general** copiar la **URL** (`https://xxxx.azurestaticapps.net`).

## 3. Conectar GitHub con Azure (token de despliegue)

1. En el recurso Static Web App → **"Administrar token de implementación"** (*Manage deployment token*) → **Copiar**.
2. En GitHub: repositorio `pokedex` → **Settings → Secrets and variables → Actions → New repository secret**:
   - **Name:** `AZURE_STATIC_WEB_APPS_API_TOKEN`
   - **Secret:** el token copiado.
3. Ir a la pestaña **Actions** del repositorio y ejecutar **"Deploy to Azure Static Web Apps" → Run workflow** (o hacer cualquier `git push` a `main`).

El workflow (`.github/workflows/azure-static-web-apps.yml`) hace:

```yaml
- run: npm ci --ignore-scripts                 # instala dependencias (sin husky)
- run: npx ng build --configuration production # compila Angular
- uses: Azure/static-web-apps-deploy@v1        # sube dist/pokedex-angular a Azure
  with:
    azure_static_web_apps_api_token: ${{ secrets.AZURE_STATIC_WEB_APPS_API_TOKEN }}
    action: upload
    app_location: dist/pokedex-angular
    skip_app_build: true
```

Cuando el job termina en verde, la aplicación está publicada en la URL del paso 2.5.

## 4. Encabezados de seguridad (`staticwebapp.config.json`)

Azure Static Web Apps permite definir encabezados HTTP para todas las respuestas con `globalHeaders`:

| Encabezado | Valor | Protege contra |
|---|---|---|
| `Content-Security-Policy` | `default-src 'self'; script-src 'self'; ...` | XSS e inyección de recursos. Solo se permiten la propia app, PokéAPI (`beta.pokeapi.co`, `pokeapi.co`), imágenes de `raw.githubusercontent.com` y `assets.pokemon.com`, y Google Fonts. |
| `Strict-Transport-Security` | `max-age=31536000; includeSubDomains; preload` | Degradación a HTTP / man-in-the-middle. |
| `X-Frame-Options` | `DENY` | Clickjacking. |
| `X-Content-Type-Options` | `nosniff` | MIME sniffing. |
| `Referrer-Policy` | `strict-origin-when-cross-origin` | Fuga de URLs a terceros. |
| `Permissions-Policy` | `camera=(), microphone=(), geolocation=(), ...` | Uso de APIs sensibles del navegador. |

Además, `navigationFallback` reescribe cualquier ruta (ej. `/pokemon/25`) a `/index.html`, para que el enrutador de Angular funcione al recargar la página.

## 5. Verificación

1. Abrir la URL pública y comprobar que carga el listado, el detalle (`/pokemon/25`) y la página `/about`, sin errores en la consola del navegador (F12).
2. Verificar los encabezados desde la terminal:
   ```bash
   curl -sI https://<nombre>.azurestaticapps.net
   ```
3. Escanear en <https://securityheaders.com/> con la URL pública (marcar *Hide results* si no se quiere publicar). Resultado obtenido: **`<colocar nota>`**.
4. (Opcional) Auditorías adicionales:
   - **SSL Labs:** <https://www.ssllabs.com/ssltest/> con el dominio `*.azurestaticapps.net`.
   - **OWASP ZAP:** *Automated Scan* contra la URL pública.

## 6. Actualizaciones futuras

Cada `git push` a la rama `main` vuelve a compilar y publicar automáticamente la aplicación.
