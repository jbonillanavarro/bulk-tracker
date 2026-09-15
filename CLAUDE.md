# Bulk Tracker — Contexto del proyecto

## Qué es
App personal de seguimiento para un objetivo de ganancia de peso (bulking): registro de peso corporal, comidas (calorías y macros), y entrenamientos. Un solo usuario, uso personal, sin backend ni base de datos externa.

## Stack técnico — IMPORTANTE, no cambiar sin motivo
- **Un único archivo HTML autocontenido** (`index.html`), sin proceso de build, sin bundler, sin npm/node en producción.
- **React 18 + ReactDOM 18** cargados vía CDN (cdnjs.cloudflare.com) como scripts UMD.
- **Babel Standalone** (cdnjs.cloudflare.com) transpila el JSX en el propio navegador en tiempo de ejecución, dentro de un `<script type="text/babel">`.
- **Sin librería de iconos externa.** Los iconos son SVGs escritos a mano dentro del propio archivo (componente `Icon` + objeto `iconPaths`). **Motivo:** se probó `lucide` (UMD) vía CDN y rompía toda la app en silencio (pantalla en negro) porque esa librería no expone componentes de React del mismo modo que `lucide-react` — cualquier intento de reintroducir una librería de iconos externa debe probarse exhaustivamente antes de sustituir el sistema actual.
- **Persistencia de datos: `localStorage` del navegador**, bajo la clave `bulk-tracker-data` (un único objeto JSON con todos los días, para minimizar operaciones de lectura/escritura). La clave de API opcional se guarda aparte en `bulk-tracker-api-key`.
- **Sin backend.** Todo corre en el cliente.

## Despliegue
- Repositorio en GitHub conectado a Netlify con auto-deploy: cualquier push a `main` redespliega solo.
- Build command y publish directory están vacíos en Netlify (no hay build).
- Archivos del proyecto: `index.html`, `manifest.json` (PWA, permite "Añadir a pantalla de inicio" en móvil), `icon-192.png`, `icon-512.png`.
- El usuario accede principalmente desde el navegador móvil, instalado como PWA (icono en pantalla de inicio, pantalla completa).

## Estructura de datos
```js
// localStorage["bulk-tracker-data"]
{
  "2026-08-05": {
    date: "2026-08-05",
    weight: { value: 51.4, time: "08:31" } | null,
    foods: [
      { id, name, kcal, protein, fat, carbs, time }
    ],
    exercise: { type, notes, time } | null
  },
  ...
}
```

## Objetivos nutricionales (constantes en el código)
- TARGET_KCAL = 2950
- TARGET_PROTEIN = 105 g
- TARGET_FAT = 65 g
- TARGET_CARBS = 475 g

## Funcionalidad actual
- **Pestaña Hoy:** navegación entre días (flechas), registro de peso, añadir comida en modo **Manual** (campos numéricos) o modo **Foto** (sube foto → llama a la API de Claude con visión → rellena los campos automáticamente, editables antes de guardar), registro de entreno (tipo + notas).
- **Pestaña Progreso:** gráfico de línea SVG (hecho a mano, sin librería de gráficos) con la evolución del peso, media de las últimas 7 pesadas, estadísticas.
- **Pestaña Historial:** lista de todos los días registrados, navegable.
- **Ajustes (icono arriba a la derecha):** campo opcional para pegar una clave de API de Anthropic (`sk-ant-...`), guardada solo en local, usada para la función de estimación por foto.

## Estimación de calorías por foto
Llama directamente desde el navegador a `https://api.anthropic.com/v1/messages` (modelo `claude-sonnet-4-6`) con:
- Header `x-api-key` con la clave del usuario.
- Header `anthropic-dangerous-direct-browser-access: true` (obligatorio para llamadas directas desde navegador).
- Contenido: bloque de imagen (base64) + prompt de texto pidiendo un JSON con `nombre`, `kcal`, `proteina_g`, `grasa_g`, `carbohidratos_g`.
- Si no hay clave guardada, se muestra un aviso pidiendo configurarla en Ajustes; el modo Manual sigue funcionando sin clave.

## Sistema de diseño
- Paleta oscura tipo "sala de pesas": fondo grafito (`#0E0F12` / `#15171B`), tarjetas `#1D2025`, acento ámbar `#E2A63B` (calorías/energía), verde azulado `#4F9B8C` (proteína/positivo), rojo apagado `#C1614A`/`#E29282` (errores/déficit).
- Tipografía: `Bebas Neue` (vía Google Fonts) para números grandes y titulares, `Inter` para el resto. Importadas con `@import` en el `<style>`.
- Mobile-first, navegación inferior por pestañas (Hoy / Progreso / Historial), tarjetas redondeadas, sin bordes duros.

## Historial de bugs ya resueltos (no repetir)
1. **Almacenamiento en `window.storage` de artefactos de Claude fallaba en móvil** → se abandonó esa vía por completo y se migró a esta app standalone con `localStorage`.
2. **Un guardado por cada día individual agotaba límites de peticiones** → ahora todo se guarda como un único objeto JSON bajo una sola clave.
3. **Librería `lucide` (no `lucide-react`) rompía el render** → sustituida por SVGs propios embebidos en el archivo.

## Gestión de tareas — TODO.md

Este proyecto mantiene un archivo `TODO.md` en la raíz con la hoja de ruta de funciones pendientes y completadas.

**Instrucción permanente:** cada vez que se complete una tarea (se implemente, se pruebe y se confirme que funciona), marca su casilla como hecha en `TODO.md` (`[x]`) y añade la fecha del día entre paréntesis, sin que haga falta que te lo pida explícitamente. No borres tareas completadas — quedan como historial del proyecto. Si en el proceso surge una tarea nueva no prevista, añádela a la fase que corresponda antes de darla por terminada.

## Cómo trabajar en este proyecto
- Cualquier cambio debe mantenerse dentro de `index.html` como archivo único autocontenido (salvo que se decida explícitamente añadir más archivos).
- Antes de añadir cualquier dependencia externa nueva (CDN de terceros), verificar que expone la API esperada — el bug de `lucide` fue exactamente por asumir una API incorrecta.
- Probar siempre en un navegador móvil real antes de dar por buena una función, ya que ya hubo comportamientos que solo fallaban en móvil.
- El usuario no es programador — las explicaciones de qué hacer (subir a GitHub, revisar Netlify, etc.) deben ser paso a paso y en español.
