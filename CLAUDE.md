# Bulk Tracker — Contexto del proyecto

## Qué es
App personal de seguimiento para un objetivo de ganancia de peso (bulking): registro de peso corporal, comidas (calorías y macros), y entrenamientos. Un solo usuario, uso personal, sin backend ni base de datos externa.

## Stack técnico — IMPORTANTE, no cambiar sin motivo
- **Un único archivo HTML autocontenido** (`index.html`), sin proceso de build, sin bundler, sin npm/node en producción.
- **React 18 + ReactDOM 18** cargados vía CDN (cdnjs.cloudflare.com) como scripts UMD.
- **Babel Standalone** (cdnjs.cloudflare.com) transpila el JSX en el propio navegador en tiempo de ejecución, dentro de un `<script type="text/babel">`.
- **Iconos: `lucide` (UMD) vía CDN** (`unpkg.com/lucide@0.383.0`), envuelto en un helper (`createLucideIcon` + `renderLucideNode`) que convierte cada icono vainilla-JS de `lucide` en un componente React utilizable como JSX (`<Plus size={16} />`), ya que `lucide` no expone componentes de React del mismo modo que `lucide-react`. Antes de usar un icono nuevo del catálogo, confirmar que existe en ese bundle (se ha verificado así antes de añadir cada icono nuevo).
- **Persistencia de datos: `localStorage` del navegador**, en varias claves: `bulk-tracker-data` (datos de cada día, un único objeto JSON), `bulk-tracker-targets` (objetivos nutricionales editables), `bulk-tracker-api-key` (clave de Claude), `bulk-tracker-gemini-api-key` (clave de Gemini), `bulk-tracker-api-provider` (proveedor de IA elegido, "claude" o "gemini").
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
    exercise: { type, notes, time } | null,
    sleepHours: 7.5 | null   // opcional; días guardados antes de esta función simplemente no tienen la clave
  },
  ...
}

// localStorage["bulk-tracker-targets"]
{ kcal: 2950, protein: 105, fat: 65, carbs: 475 }  // valores por defecto si no se ha guardado nada
```

## Objetivos nutricionales — editables desde Ajustes
Ya no son constantes de código: se editan en Ajustes ("Objetivos nutricionales") y se guardan en `bulk-tracker-targets`. Valores por defecto (`DEFAULT_TARGETS` en el código) si el usuario no los ha tocado:
- kcal = 2950
- protein = 105 g
- fat = 65 g
- carbs = 475 g

Además hay una sugerencia automática (no intrusiva, nunca se aplica sin confirmación) en la pestaña Progreso, basada en los últimos 14 días:
- Si el peso sube ≥0,8 kg/semana sostenido, sugiere bajar ~200 kcal (~50g carbohidratos menos).
- Si se cumple el objetivo calórico (≥70% de los días dentro de ±10%) pero el peso apenas sube (≤0,15 kg/semana), sugiere subir ~200 kcal (~50g carbohidratos más).
- La proteína no se toca al aplicar una sugerencia; los ajustes de calorías se hacen sobre todo con carbohidratos.

## Funcionalidad actual
- **Pestaña Hoy:** navegación entre días (flechas), registro de peso, registro de horas de sueño (con recordatorio si falta el dato de ayer o si la media semanal es baja), sugerencias de comida concretas por franja horaria y macros restantes (catálogo local + botón opcional "Otra (IA)"), añadir comida en modo **Manual** (campos numéricos) o modo **Foto** (sube foto → la IA configurada rellena los campos automáticamente, editables antes de guardar), registro de entreno (tipo + notas).
- **Pestaña Progreso:** gráfico de peso (SVG a mano, con proyección a 4 semanas discontinua) y gráfico de barras de calorías diarias, ambos sin librería externa; medias y estadísticas.
- **Pestaña Historial:** lista de todos los días registrados, navegable, marcando visualmente los días desviados del plan (déficit calórico significativo, sin entreno o poco sueño) y con una observación descriptiva de correlación con el peso del día siguiente.
- **Ajustes (icono arriba a la derecha):** objetivos nutricionales, selector de proveedor de IA (Claude / Gemini) con su clave respectiva, y exportar/restaurar copia de seguridad en JSON.

## Estimación de calorías por foto y sugerencias de comida por IA
Dos proveedores posibles, elegidos en Ajustes (`bulk-tracker-api-provider`):
- **Claude:** `https://api.anthropic.com/v1/messages` (modelo `claude-sonnet-4-6`), con `x-api-key` y el header `anthropic-dangerous-direct-browser-access: true` (obligatorio para llamadas directas desde navegador).
- **Gemini:** `https://generativelanguage.googleapis.com/v1beta/models/gemini-3.6-flash:generateContent?key=...` (clave como query param), gratis de forma indefinida pero con el aviso de que Google puede usar las imágenes enviadas para entrenar sus modelos en el tier gratuito — se muestra ese aviso en Ajustes. **Ojo:** Google va retirando modelos de Gemini con cierta frecuencia (ya pasó una vez con `gemini-2.5-flash`, que dejó de estar disponible para claves nuevas) — si un usuario reporta "modelo no disponible" o similar, comprobar el nombre de modelo vigente antes de asumir que es otro tipo de fallo.

Ambos reciben un prompt pidiendo JSON con `nombre`, `kcal`, `proteina_g`, `grasa_g`, `carbohidratos_g` (o un array de 3 para las sugerencias de comida). Si no hay clave del proveedor activo, se muestra un aviso pidiendo configurarla en Ajustes; el modo Manual y el catálogo local de sugerencias siguen funcionando sin clave.

## Sistema de diseño
- Paleta oscura tipo "sala de pesas": fondo grafito (`#0E0F12` / `#15171B`), tarjetas `#1D2025`, acento ámbar `#E2A63B` (calorías/energía), verde azulado `#4F9B8C` (proteína/positivo), rojo apagado `#C1614A`/`#E29282` (errores/déficit).
- Tipografía: `Bebas Neue` (vía Google Fonts) para números grandes y titulares, `Inter` para el resto. Importadas con `@import` en el `<style>`.
- Mobile-first, navegación inferior por pestañas (Hoy / Progreso / Historial), tarjetas redondeadas, sin bordes duros.

## Gestión de tareas — TODO.md

Este proyecto mantiene un archivo `TODO.md` en la raíz con la hoja de ruta de funciones pendientes y completadas.

**Instrucción permanente:** cada vez que se complete una tarea (se implemente, se pruebe y se confirme que funciona), marca su casilla como hecha en `TODO.md` (`[x]`) y añade la fecha del día entre paréntesis, sin que haga falta que te lo pida explícitamente. No borres tareas completadas — quedan como historial del proyecto. Si en el proceso surge una tarea nueva no prevista, añádela a la fase que corresponda antes de darla por terminada.

## Cómo trabajar en este proyecto
- Cualquier cambio debe mantenerse dentro de `index.html` como archivo único autocontenido (salvo que se decida explícitamente añadir más archivos).
- Antes de añadir cualquier dependencia externa nueva (CDN de terceros) o de usar un endpoint de API externo por primera vez, verificar contra la documentación real (o el propio bundle/respuesta) que expone la forma esperada — asumir una API incorrecta ya causó fallos en el pasado.
- Probar siempre en un navegador móvil real antes de dar por buena una función, ya que ya hubo comportamientos que solo fallaban en móvil.
- El usuario no es programador — las explicaciones de qué hacer (subir a GitHub, revisar Netlify, etc.) deben ser paso a paso y en español.