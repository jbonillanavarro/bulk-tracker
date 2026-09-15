# Bulk Tracker — Hoja de ruta

Roadmap de funciones pendientes y completadas. Al terminar una tarea, márcala con `[x]` y añade la fecha (ej. `(hecho 2026-08-10)`). No borres tareas completadas — sirven de historial del proyecto.

## Fase 1 — Fundamentos

- [x] Exportar/importar copia de seguridad en JSON (botón "Descargar copia de seguridad" y "Restaurar desde archivo"), para no depender solo del localStorage del navegador (hecho 2026-09-15)
  Añadido en el modal de Ajustes: "Descargar copia de seguridad" genera un `.json` con todos los días (`bulk-tracker-backup-<fecha>.json`, sin la clave de API); "Restaurar desde archivo" valida la estructura, pide confirmación (reemplaza todos los datos actuales) y guarda en localStorage. Iconos `Download`/`Upload` añadidos al wrapper de lucide ya existente (verificado que existen en el bundle CDN). Pendiente: confirmar visualmente en un navegador móvil real, ya que este entorno no tiene un navegador disponible para probarlo de forma automática.
- [x] Separar la pestaña Progreso en dos gráficas independientes: una de peso y otra de calorías diarias (actualmente solo existe la de peso) (hecho 2026-09-15)
  Añadido `KcalBarChart`: gráfica de barras (SVG a mano, mismo estilo que la de peso) con las kcal totales de los últimos 14 días registrados, línea discontinua en el objetivo (TARGET_KCAL) y barras en verde azulado si ese día se alcanzó/superó el objetivo o en ámbar si se quedó corto. Se muestra debajo de la gráfica de peso existente, solo cuando hay al menos un día con comidas registradas.
- [x] Mejorar el modo Foto de comida: permitir añadir como contexto opcional el nombre del alimento y/o las calorías ya conocidas (no solo el peso en gramos), y usarlos en el prompt de estimación (hecho 2026-09-15)
  Añadidos dos campos opcionales antes de "Estimar": "Nombre del plato" y "Kcal ya conocidas" (además del peso en gramos que ya existía). Si se rellenan, se incluyen como contexto adicional en el prompt enviado a la API de Claude — el nombre como descripción del plato, y las kcal conocidas como referencia principal para que el modelo ajuste los macros de forma coherente con ellas. Sin ninguno de los dos campos, el comportamiento es idéntico al anterior.

- [ ] Convertir los objetivos nutricionales (TARGET_KCAL, TARGET_PROTEIN, TARGET_FAT, TARGET_CARBS) de constantes fijas en el código a valores editables desde Ajustes y persistidos en localStorage. Añadir lógica de ajuste: si tras 2 semanas cumpliendo el objetivo calórico no hay subida real de peso, sugerir subir 150-300 kcal (principalmente vía carbohidratos); si se sube más rápido de lo esperado (>0,7-0,8 kg/semana sostenido), sugerir bajar ligeramente

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
