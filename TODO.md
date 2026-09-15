# Bulk Tracker — Hoja de ruta

Roadmap de funciones pendientes y completadas. Al terminar una tarea, márcala con `[x]` y añade la fecha (ej. `(hecho 2026-08-10)`). No borres tareas completadas — sirven de historial del proyecto.

## Fase 1 — Fundamentos

- [ ] Exportar/importar copia de seguridad en JSON (botón "Descargar copia de seguridad" y "Restaurar desde archivo"), para no depender solo del localStorage del navegador
- [ ] Separar la pestaña Progreso en dos gráficas independientes: una de peso y otra de calorías diarias (actualmente solo existe la de peso)
- [ ] Mejorar el modo Foto de comida: permitir añadir como contexto opcional el nombre del alimento y/o las calorías ya conocidas (no solo el peso en gramos), y usarlos en el prompt de estimación

## Fase 2 — Inteligencia sobre los datos

- [ ] Proyección de tendencia de peso a futuro, calculada de forma transparente a partir de la tasa de cambio real de las últimas semanas (no un modelo de caja negra ni una predicción fija a 1 año). Mostrar el cálculo/supuestos usados, no solo el número final. Debe recalcularse automáticamente según se añaden más datos.
- [ ] Marcar visualmente en el historial los días que se desvían del plan (déficit calórico significativo, sin entreno, sueño insuficiente) y mostrar una correlación simple con la evolución del peso de los días siguientes (no causalidad estricta, solo una observación descriptiva)

## Fase 3 — Asistente activo

- [ ] Sugerencias de comida según la hora del día y las calorías/macros que ya lleva la persona ese día. Deben proponer alimentos concretos (ej. "2 yogures griegos + un puñado de almendras"), nunca sugerencias genéricas tipo "come algo proteico"
- [ ] Aviso/recordatorio visual dentro de la app sobre horas de sueño (no notificación push real, ya que eso requeriría permisos adicionales del navegador/móvil que esta app no gestiona)

## Principios que deben respetarse en cualquier tarea nueva

- Toda recomendación numérica (calorías, proteína, frecuencia de entreno, sueño) debe basarse en consensos científicos existentes (ISSN position stands, guías de sueño, etc.), no en cifras inventadas
- Nunca perder datos ya registrados: cualquier cambio de estructura de datos debe incluir migración de los datos existentes en localStorage, no un reseteo
- Seguir las restricciones técnicas y de diseño ya documentadas en CLAUDE.md (archivo único sin build, sin librerías de iconos externas, localStorage como única persistencia, paleta y tipografía ya definidas)
- Probar en navegador móvil real antes de dar una función por terminada
