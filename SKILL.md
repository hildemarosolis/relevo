---
name: relevo
description: Genera un documento de relevo (handoff) estructurado de 8 secciones antes de cerrar una sesión de Claude Code, para que la siguiente sesión —u otra persona— retome el trabajo sin perder contexto. Lee el estado real del proyecto (git log, git status, archivos tocados) y reconstruye de la conversación lo que git no registra: objetivo de la sesión, decisiones tomadas, cosas que NO funcionaron, trampas detectadas y compromisos adquiridos. Guarda el documento en handoffs/ y entrega el mensaje exacto que abre la siguiente sesión. Úsala SIEMPRE que el usuario diga "cierra la sesión", "guarda estado", "vamos a /clear", "antes de cerrar", "genera handoff", "haz relevo", "lo dejamos aquí", "mañana seguimos", "ya terminamos por hoy", "me voy", "cierra esto", o invoque /relevo. También dispara proactivamente: si el usuario anuncia que va a limpiar contexto o cerrar la terminal y NO ha pedido relevo, Claude debe ofrecerlo antes de que se pierda el contexto — una sesión cerrada sin relevo es contexto quemado. No dispara para preguntas de estado a mitad de sesión ("¿cómo vamos?") ni para resúmenes parciales que no preceden un cierre.
---

# relevo

Skill para que Claude Code cierre cada sesión de trabajo dejando un documento de relevo que la siguiente sesión pueda leer como primera acción y retomar sin pérdida. Pensada para operadores que trabajan en sesiones largas, con varios proyectos abiertos, donde `/clear` sin relevo significa re-explicar media hora de contexto.

## Por qué existe

El contexto de una sesión de Claude Code muere con la sesión. Git registra los commits, pero no registra: por qué se tomó una decisión, qué se intentó y falló, qué quedó a medias, qué prometió el usuario hacer después, ni qué trampa se detectó a mitad de camino. Todo eso vive solo en la conversación — y la conversación desaparece con `/clear`.

El relevo captura eso ANTES de que muera. No es una bitácora (no acumula histórico), no es un changelog (no lista features), no es documentación (no explica el sistema). Es el testigo que una sesión le pasa a la siguiente: estado, dirección y advertencias, en un solo archivo.

## Cuándo se activa

Slash command explícito: `/relevo`.

Frases gatillo del usuario:
- "cierra la sesión", "vamos a cerrar", "antes de cerrar"
- "guarda estado", "genera handoff", "haz relevo"
- "vamos a /clear", "voy a limpiar el contexto"
- "lo dejamos aquí", "mañana seguimos", "ya terminamos por hoy", "me voy"

**Disparo proactivo (obligatorio)**: si el usuario anuncia un `/clear` o un cierre de terminal y no ha pedido relevo, ofrecerlo en una línea antes de que ocurra: *"¿Genero el relevo antes? Si haces /clear sin él, este contexto se pierde."* No insistir más de una vez.

**Disparo preventivo**: si a mitad de una sesión larga Claude detecta que está perdiendo el hilo (contradicciones con lo dicho antes, contexto comprimido), proponer generar el relevo en ese momento. No esperar al final — un relevo a mitad de sesión vale más que uno reconstruido desde la confusión.

**NO se activa** para preguntas de estado a mitad de sesión ("¿cómo vamos?", "¿qué falta?") ni para resúmenes que no preceden un cierre.

## Cómo opera

### Paso 1 — Recolectar evidencia (no improvisar)

Antes de escribir nada, leer el estado real:

- `git log --oneline -15` y `git status` en cada repo tocado durante la sesión.
- Lista de archivos creados/modificados/eliminados en la sesión (la conversación es la fuente; git lo confirma).
- Si existen `CLAUDE.md`, `BITACORA.md`, `TODO.md` o equivalentes en el cwd, revisarlos para no contradecir pendientes ya registrados.

### Paso 2 — Reconstruir lo que git no sabe

De la conversación, extraer:

- **Objetivo inicial** de la sesión y **objetivo emergente** si el flujo derivó a otra cosa (casi siempre deriva).
- **Decisiones tomadas** — incluidas las que NO deben reabrirse.
- **Lo que NO funcionó** — intentos fallidos, críticas recibidas, hipótesis descartadas. Esta sección es la más valiosa y la primera que se pierde.
- **Trampas detectadas** — patrones del usuario o del proyecto que la siguiente sesión debe vigilar.
- **Compromisos** — qué prometió el usuario hacer y cuándo.

Si algo de esto no está claro, preguntar al usuario ANTES de escribir. Un relevo con huecos inventados es peor que uno con huecos marcados.

### Paso 3 — Escribir el documento

Plantilla fija de 8 secciones. No omitir ninguna; si una sección queda vacía, escribir explícitamente "Nada que registrar" — el vacío declarado también es información.

```markdown
# HANDOFF — Sesión YYYY-MM-DD

**Estado al cierre:** [una línea: limpio / a medias / interrumpido + lo esencial]

---

## 1. Objetivo de la sesión
[Objetivo planeado. Si hubo objetivo emergente no planeado, marcarlo como tal.]

## 2. Estado actual
### Lo que quedó hecho ✅
[Tabla: Item | Ubicación | Estado]
### Lo que NO se tocó (pendiente real) ⏳
[Tabla: Item | Por qué importa | Bloqueado por]

## 3. Archivos en trabajo / modificados
[Bloque de código: Created / Modified / Untouched relevantes]

## 4. Cambios hechos (commits / decisiones)
[Commits con hash + decisiones de diseño tomadas en la sesión]

## 5. Cosas que NO funcionaron / lecciones
[Intentos fallidos, críticas recibidas y qué se aceptó de ellas,
patrones detectados. Sin maquillar.]

## 6. Plan siguiente
### Acción inmediata
[Pasos numerados, ejecutables, empezando por cómo abrir la sesión nueva]
### Después
[Lo que sigue cuando lo inmediato esté cerrado]

## 7. Contexto crítico que la próxima sesión necesita saber
### Decisiones ya tomadas (no reabrir)
### Setup técnico activo
### Trampas conocidas
### Vocabulario interno (si el proyecto usa términos propios)

## 8. Compromisos vigentes al cierre
[Lista numerada: quién se comprometió a qué y para cuándo.
Si hay condición de incumplimiento, nombrar la consecuencia.]

---

**Generado al cierre de sesión YYYY-MM-DD por Claude Code, validado por [usuario].**
**Siguiente lectura prevista: [cuándo y en qué contexto].**
```

### Paso 4 — Guardar

- Ruta por defecto: `handoffs/handoff-YYYY-MM-DD.md` en la raíz del proyecto activo (crear `handoffs/` si no existe).
- Si el usuario tiene una carpeta central de handoffs configurada (en su CLAUDE.md o memoria), usar esa.
- Si ya existe un handoff del mismo día, sufijar: `handoff-YYYY-MM-DD-2.md`.

### Paso 5 — Validar y entregar el testigo

En el chat, tras guardar (máximo ~200 palabras):

1. Ruta del archivo guardado.
2. Las 2-3 cosas más importantes que registra.
3. El mensaje exacto para abrir la sesión nueva, listo para copiar:

```
Lee [ruta absoluta del handoff] y continúa desde la sección "Plan siguiente".
```

4. Preguntar: *"¿Falta algo antes de cerrar?"* — y solo después de la confirmación dar por terminado el relevo.

## Reglas de tono

- Español neutro, directo, sin adulación.
- El documento dice la verdad: si la sesión fue improductiva, el relevo lo registra. Un relevo que maquilla el estado sabotea a la sesión siguiente.
- La sección 5 (lo que NO funcionó) nunca se omite ni se suaviza.
- El documento no tiene tope rígido de palabras — es un artefacto, no una respuesta de chat — pero apunta a 1-2 páginas. La respuesta en chat tras guardarlo sí es breve.

## Qué NO es

| No es | Porque |
|---|---|
| Bitácora | La bitácora acumula histórico; el relevo se consume una vez y caduca |
| Changelog | El changelog lista qué cambió; el relevo explica qué sigue y qué vigilar |
| Documentación | La documentación explica el sistema; el relevo explica la sesión |
| Resumen de contexto automático | El resumen comprime; el relevo selecciona y advierte |
