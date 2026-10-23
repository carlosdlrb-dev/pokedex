# PokeDex — Despliegue en Azure for Students

Este repositorio contiene la aplicación **PokeDex** (Angular 14) desarrollada por PumasLab para **Pueblo Paleta Inc.**, publicada en la nube pública **Microsoft Azure** usando la suscripción gratuita **Azure for Students** y el servicio **Azure Static Web Apps**.

- **URL pública:** `https://<nombre-asignado>.azurestaticapps.net` *(reemplazar por la URL real)*
- **Guía de despliegue:** ver [Despliegue.md](Despliegue.md)

---

## Tutorial: creación de la cuenta Azure for Students

### Requisitos previos

- Un **correo institucional** de la universidad (ej. `usuario@universidad.edu.co`). Azure for Students lo usa para verificar que eres estudiante.
- Un número de **teléfono móvil** para la verificación por SMS.
- **No** se necesita tarjeta de crédito.

### Paso 1. Ingresar al portal de Azure for Students

1. Abrir en el navegador: <https://azure.microsoft.com/es-es/free/students/>
2. Hacer clic en **"Empezar gratis"** / **"Start free"**.

### Paso 2. Iniciar sesión con una cuenta Microsoft

1. Escribir el **correo institucional**.
2. Si el correo ya está asociado a Microsoft 365 de la universidad, se redirige al inicio de sesión de la institución; ingresar la contraseña institucional.
3. Si no existe una cuenta Microsoft con ese correo, elegir **"Crear una"**, definir una contraseña y confirmar el código enviado al correo.

### Paso 3. Verificar la identidad de estudiante

1. Completar el formulario con: país, nombre, apellidos, fecha de nacimiento y nombre de la institución educativa.
2. Ingresar el número de teléfono móvil y pulsar **"Enviarme un mensaje de texto"**.
3. Escribir el código recibido por SMS y pulsar **"Comprobar código"**.
4. Si se solicita, confirmar el correo académico abriendo el enlace que Microsoft envía a la bandeja institucional.

### Paso 4. Aceptar el contrato

1. Leer y marcar la casilla de aceptación del **Contrato de suscripción de Microsoft Azure for Students**.
2. Pulsar **"Registrarse"**.
3. Al finalizar, Azure muestra el mensaje de bienvenida y redirige al **Portal de Azure**.

### Paso 5. Comprobar la suscripción

1. Ingresar a <https://portal.azure.com>.
2. En el buscador superior escribir **"Suscripciones"**.
3. Debe aparecer la suscripción **"Azure for Students"** en estado **Activo**, con el crédito disponible (USD 100 durante 12 meses, renovable mientras se siga siendo estudiante).

### Paso 6. Configuración inicial recomendada

1. **Grupo de recursos:** en el portal buscar **"Grupos de recursos" → "+ Crear"**.
   - Suscripción: `Azure for Students`
   - Nombre: `rg-pokedex`
   - Región: `East US 2` (o la más cercana disponible)
   - Pulsar **"Revisar y crear" → "Crear"**.
2. **Alerta de presupuesto (opcional pero recomendado):** buscar **"Cost Management" → "Presupuestos" → "+ Agregar"**, definir un presupuesto bajo (ej. USD 5) y una alerta por correo para no consumir el crédito por error.
3. **Cuenta de GitHub:** tener una cuenta en <https://github.com> con el repositorio `pokedex` creado, pues Azure Static Web Apps despliega directamente desde GitHub.

Con esto la cuenta queda lista para el despliegue descrito en [Despliegue.md](Despliegue.md).

---

## Sobre la aplicación

La aplicación muestra el listado y el detalle de los Pokémon de las primeras 3 generaciones, consumiendo la API [PokéAPI](https://pokeapi.co/).

### Ejecutar en local

Requiere [Node.js](https://nodejs.org) 18 LTS.

```bash
npm install
npm start
```

La app queda disponible en `http://localhost:4200`.

## Créditos

Código fuente original: [keilermora/pokedex-angular](https://github.com/keilermora/pokedex-angular).
