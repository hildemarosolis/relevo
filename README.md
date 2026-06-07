# relevo

> Skill para Claude Code que genera un documento de relevo (handoff) de 8 secciones antes de cerrar una sesión, para que la siguiente retome el trabajo sin perder contexto. Diseñada para operadores con sesiones largas y varios proyectos abiertos.

## El problema

El contexto de una sesión de Claude Code muere con la sesión. Git guarda los commits, pero no guarda por qué decidiste algo, qué intentaste y falló, qué quedó a medias, qué prometiste hacer el lunes, ni la trampa que detectaste a mitad de camino. Todo eso vive en la conversación — y la conversación desaparece con `/clear`.

Resultado conocido: abres sesión nueva, pasas media hora re-explicando dónde ibas, y aun así la mitad de las advertencias se pierden.

## Qué hace esta skill

Cuando dices `/relevo` (o "cierra la sesión", "guarda estado", "vamos a /clear", "mañana seguimos"), Claude:

1. **Lee el estado real** — `git log`, `git status`, archivos tocados en la sesión.
2. **Reconstruye lo que git no sabe** — objetivo (planeado y emergente), decisiones, fallos, trampas, compromisos. Si algo no está claro, te pregunta antes de escribir.
3. **Escribe el documento** — plantilla fija de 8 secciones:
   1. Objetivo de la sesión
   2. Estado actual (hecho ✅ / pendiente real ⏳)
   3. Archivos en trabajo / modificados
   4. Cambios hechos (commits / decisiones)
   5. Cosas que NO funcionaron / lecciones
   6. Plan siguiente
   7. Contexto crítico para la próxima sesión (decisiones no-reabrir, setup, trampas, vocabulario)
   8. Compromisos vigentes al cierre
4. **Guarda** en `handoffs/handoff-YYYY-MM-DD.md` y te entrega el mensaje exacto para abrir la sesión nueva, listo para copiar.
5. **Pregunta "¿falta algo?"** antes de dar por cerrado.

Además dispara **proactivamente**: si anuncias un `/clear` sin haber pedido relevo, Claude lo ofrece antes de que el contexto se queme.

## Instalación

### Opción A — Skill global (recomendada)

```bash
git clone https://github.com/hildemarosolis/relevo.git ~/.claude/skills/relevo
```

Reinicia Claude Code. Disponible en todas tus sesiones.

### Opción B — Symlink desde repo local

```bash
git clone https://github.com/hildemarosolis/relevo.git ~/REPOS/relevo
ln -s ~/REPOS/relevo ~/.claude/skills/relevo
```

Ventaja: editas el SKILL.md en el repo y los cambios se reflejan al instante.

## Uso

### Disparo explícito

```
/relevo
```

### Disparo automático (frases gatillo)

- "cierra la sesión", "antes de cerrar", "vamos a cerrar"
- "guarda estado", "genera handoff", "haz relevo"
- "vamos a /clear", "voy a limpiar el contexto"
- "lo dejamos aquí", "mañana seguimos", "ya terminamos por hoy"

### Abrir la sesión siguiente

El relevo te deja el mensaje exacto. Es siempre de la forma:

```
Lee /ruta/al/handoff-YYYY-MM-DD.md y continúa desde la sección "Plan siguiente".
```

## Filosofía

El relevo es un **testigo, no un archivo más**. Se escribe una vez, se lee una vez, y caduca cuando la siguiente sesión lo absorbe.

| No es | Porque |
|---|---|
| Bitácora | La bitácora acumula histórico; el relevo se consume y caduca |
| Changelog | El changelog lista qué cambió; el relevo explica qué sigue y qué vigilar |
| Documentación | La documentación explica el sistema; el relevo explica la sesión |
| Resumen automático de contexto | El resumen comprime; el relevo selecciona y advierte |

La sección más valiosa es la 5 — **lo que NO funcionó** — porque es exactamente lo que git no registra y lo primero que se olvida. Nunca se omite ni se suaviza: un relevo que maquilla el estado sabotea a la sesión siguiente.

## Ejemplo

Ver [`examples/`](examples/) — handoff completo de un proyecto sintético (SaaS de inventario) que muestra las 8 secciones con la textura esperada.

## Roadmap

- [x] v0.1 — versión inicial con plantilla fija de 8 secciones
- [ ] v0.2 — detectar handoffs previos no consumidos ("hay un relevo del martes que nunca se leyó")
- [ ] v0.3 — modo dual-sesión: relevo cruzado entre dos sesiones paralelas del mismo ecosistema
- [ ] v0.4 — métricas de relevo: cuánto contexto sobrevive entre sesiones (compromisos cumplidos vs registrados)

## Contribuir

Si generas un relevo y la sesión siguiente igual se desorienta, abre un issue con qué información faltó. Ese es el único bug que importa en esta skill.

## Licencia

MIT — ver [LICENSE](LICENSE).

## Autor

Hildemaro Solís — tercer repo público. Parte de una serie de skills para builders que ya tienen demasiado en producción.

La serie completa: [detector-humo](https://github.com/hildemarosolis/detector-humo) (filtra ideas externas / reels) → [validador](https://github.com/hildemarosolis/validador) (filtra tus propias ideas) → **relevo** (cierra la sesión sin perder lo decidido).
