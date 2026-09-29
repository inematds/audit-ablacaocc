# audit-ablacaocc — auditoría de ablación para Claude Code

**🇧🇷 [Português](README.md) · 🇺🇸 [English](README.en.md) · 🇪🇸 [Español](README.es.md)**

Skill de **diagnóstico** (solo lectura) que audita tu `CLAUDE.md`, tus skills y tus hooks, clasifica cada instrucción y propone una versión mínima — sin modificar ningún archivo.

> Cada línea de tu configuración es **culpable de complejidad hasta que se demuestre su utilidad**.

## 📖 Guía de uso

Guía completa (landing + paso a paso): **https://inematds.github.io/audit-ablacaocc/guia/es/**

- Skill: [`SKILL.md`](SKILL.md)
- Base conceptual: [`doc/`](doc/) — resumen de la charla de Boris Cherny (Y Combinator, un día después del lanzamiento de Opus 5)

---

## Índice

1. [La visión de Boris: por qué borrar tu propia configuración](#1-a-visão-do-boris-por-que-apagar-a-própria-config)
2. [Por qué esta limpieza es útil en la práctica](#2-por-que-essa-faxina-é-útil-na-prática)
3. [Qué hace la skill (y qué nunca hace)](#3-o-que-a-skill-faz-e-o-que-ela-nunca-faz)
4. [Instalación](#4-instalação)
5. [Cómo usarla](#5-como-usar)
6. [Consejo de uso: invoca skills todo el tiempo](#6-dica-de-uso-chame-skills-o-tempo-todo)
7. [Ciclo recomendado](#7-ciclo-recomendado)
8. [Guía práctica: publicar en Git y en el portal](#8-guia-prático-publicar-no-git-e-no-portal)

---

## 1. La visión de Boris: por qué borrar tu propia configuración

Boris Cherny creó Claude Code en Anthropic. Su tesis, resumida:

**La configuración envejece.** Cada instrucción que escribes corrige una debilidad de un modelo específico. Cuando el siguiente modelo deja de tener esa debilidad, la instrucción se convierte en peso muerto y sigue leyéndose cada vez que se usa, ocupando contexto y limitando al modelo.

Eso fue literalmente lo que ocurrió en Claude Code: buena parte del prompt de sistema existía solo para corregir comportamientos que el modelo debería poder ejecutar por sí solo. Con Opus 5, el equipo **eliminó más del 80% del prompt de sistema**. Lo que quedó es casi todo seguridad, permisos, análisis estático e interfaz.

**Ablación** es el método. Es un término de investigación: eliminas algo para medir su impacto. Aplicado a prompts, significa borrar todo y volver a incorporar cada línea, una por una, para descubrir qué hace realmente. El consejo de Boris para quienes solo *usan* Claude Code:

> Cada ~6 meses, y sobre todo cuando se lance un nuevo modelo importante: borra tu `CLAUDE.md`, borra tus skills, borra tus hooks y mira qué hace el modelo sin ellos.

La reconstrucción tiene cuatro etapas, y la cuarta es la que importa:

1. Borra la configuración.
2. Usa la herramienta en **trabajo real**, no en una prueba hipotética.
3. Observa dónde funciona bien y dónde tropieza.
4. Solo vuelve a agregar una instrucción **después de ver que el modelo falla repetidamente en lo mismo**.

Hay dos razones para seguir esta disciplina: eres muy malo para predecir qué instrucciones necesita el modelo, y cada línea que mantienes consume contexto en cada ejecución.

Los otros pilares de esta visión:

- **Especificar demasiado es el error más común** — y es más frecuente entre quienes tienen años de experiencia en ingeniería. Indicar paso a paso «haz A, luego B y después C» funcionaba con modelos antiguos; hoy impide que el modelo encuentre un camino mejor. Escribe **tarea + límites de seguridad + criterios de salida** y vuelve después a comprobar el resultado.
- **La verificación es la palanca en la que casi todo el mundo se equivoca.** Un prompt corto con una forma real de que el modelo compruebe su propio trabajo es mejor que un prompt enorme sin verificación. El ejemplo clásico: reescribir la app de Electron en Swift, ejecutar la original en una VM, tomar una captura de pantalla, comparar píxel por píxel y no parar hasta terminar. Producir → observar → comparar → condición de parada.
- **Apunta por encima del límite.** Dale una tarea un poco más difícil de lo que crees que el modelo puede manejar y vuelve a probar tus fracasos anteriores con cada nueva versión: la causa del fallo puede haber desaparecido.
- **Trabajar con modelos se ha vuelto una ciencia empírica**, no teórica. Se parece más a conocer a una criatura viva que a configurar un sistema. El nivel de instrucciones adecuado es el mismo que usarías para orientar a un colega.
- **No hay trucos secretos.** El ciclo es: tarea difícil → medios para verificar → observar dónde se atasca → corregir *eso* (con un mejor prompt, una skill o un MCP) → repetir.

Para hacer pruebas por tu cuenta, Boris mencionó dos opciones: la flag de prompt de sistema al iniciar y `CLAUDE_CODE_SIMPLE=1` (elimina todos los prompts de sistema, incluidos los de las herramientas; es la línea base de ablación que se usa internamente). Un detalle contraintuitivo: sin los prompts, el modelo se vuelve *ligeramente más inteligente*; se mantienen porque hacen que el producto se comporte como la gente espera.

## 2. Por qué esta limpieza es útil en la práctica

Quienes usan Claude Code desde hace tiempo acumulan la misma capa de sedimentos:

| Síntoma | Lo que suele ser |
|---|---|
| Un `CLAUDE.md` de 300 líneas con excepciones desactualizadas | correcciones para modelos que ya no existen |
| La misma regla en `CLAUDE.md` y dentro de 3 skills | redundancia que genera conflictos cuando cambia una de las copias |
| Una skill que enseña «cómo pensar» en 400 líneas | razonamiento genérico que el modelo ya hace por sí solo |
| Un paso a paso rígido de 12 etapas | microgestión que bloquea una mejor ruta |
| Dos reglas que se contradicen | el modelo elige una y tú no sabes cuál |
| Nada que diga *cómo comprobar* que quedó bien | el vacío más costoso de todos |

El costo es real y acumulativo: contexto consumido en cada llamada, menos autonomía, comportamiento inconsistente y reglas zombi que nadie se atreve a borrar porque ya nadie sabe qué sostienen.

Lo que ganas con la limpieza:

- **Recuperas contexto** para el trabajo real.
- **Menos contradicciones** — una regla que solo existe en un lugar no entra en conflicto con ninguna otra.
- **Más autonomía** — los criterios de salida permiten que el modelo encuentre una ruta mejor que la tuya.
- **Una configuración auditable** — sabrás *por qué* está cada línea, porque cada una habrá superado una pregunta.
- **La verificación entra en el cálculo** — la auditoría muestra dónde falta una forma objetiva de demostrar que el resultado es correcto.

El objetivo **no** es maximizar la reducción. Es maximizar:

**calidad + autonomía + verificabilidad ÷ complejidad**

Por eso la skill preserva explícitamente lo que el modelo no puede inferir: identidad del proyecto, rutas y fuentes de verdad, branding, seguridad, compliance, integraciones, contratos de interfaz y convenciones internas.

## 3. Qué hace la skill (y qué nunca hace)

**Hace lo siguiente:**

- lee `CLAUDE.md` (del proyecto y global), skills, hooks y `settings.json`;
- clasifica cada instrucción — `CONTEXTO`, `GUARDRAIL`, `CRITERIO DE CALIDAD`, `VERIFICACIÓN`, `INTEGRACIÓN/HERRAMIENTA`, `PROCEDIMIENTO REPETIBLE`, `MICROGESTIÓN`, `REDUNDANCIA`, `LEGADO/OBSOLETA`, `AMBIGUA/NO COMPROBADA`;
- decide `KEEP` / `SIMPLIFY` / `MOVE` / `MERGE` / `TEST` / `REMOVE` (en caso de duda, `TEST`);
- audita las skills una por una — `KEEP` / `SIMPLIFY` / `MERGE` / `SPLIT` / `LOAD-ON-DEMAND` / `CONVERT-TO-CONTEXT` / `DELETE-CANDIDATE`;
- entrega un informe de 10 secciones: resumen ejecutivo, métricas, problemas por archivo, candidatas a eliminación, redundancias y conflictos, skills, propuesta de `CLAUDE.md` mínimo, skills propuestas, plan de prueba de ablación (versiones A/B/C) y las 10 principales propuestas de cambio según impacto ÷ riesgo.

**Nunca hace lo siguiente:** editar, mover, borrar, sobrescribir la configuración ni hacer commits. Es un diagnóstico. Aplicarlo es decisión tuya y requiere un pedido aparte — es intencional: una auditoría que ya modifica cosas es una auditoría en la que no confías.

Diferencia con la skill de al lado: `memory-audit` se ocupa de la **memoria** guardada de la sesión. Esta se ocupa de la **configuración del agente**.

## 4. Instalación

```bash
git clone https://github.com/inematds/audit-ablacaocc.git
mkdir -p ~/.claude/skills/audit-ablacao
cp audit-ablacaocc/SKILL.md ~/.claude/skills/audit-ablacao/SKILL.md
```

O, para instalarla solo en el proyecto actual:

```bash
mkdir -p .claude/skills/audit-ablacao
cp /caminho/audit-ablacaocc/SKILL.md .claude/skills/audit-ablacao/SKILL.md
```

Reinicia la sesión de Claude Code para que cargue la skill.

## 5. Cómo usarla

Invocación directa:

```
/audit-ablacao
```

O mediante un activador, en lenguaje natural:

- «haz una auditoría de ablación de mi CLAUDE.md global»
- «audita las skills de este proyecto, mi prompt está inflado»
- «¿qué puedo eliminar de mi configuración sin romper nada?»

Alcances que vale la pena revisar:

| Alcance | Cuándo |
|---|---|
| Global (`~/.claude/`) | cada ~6 meses o cuando se lance un nuevo modelo importante |
| Un proyecto | cuando su `CLAUDE.md` supera ~150 líneas |
| Un conjunto de skills | cuando 3+ skills compiten por el mismo activador |

Después del informe, el flujo recomendado es elegir las **10 principales según impacto ÷ riesgo**, aplicarlas en otra sesión y ejecutar el **plan de ablación** (versiones A / B / C) en tareas reales del proyecto antes de aprobar los cambios.

## 6. Consejo de uso: invoca skills todo el tiempo

Esta es una práctica que adopté y que cambia el resultado más que cualquier ajuste de prompt: **usa las skills al máximo, invocándolas directamente o dentro de los proyectos.**

¿Por qué?

- **Una skill es un procedimiento, no una instrucción global.** Las reglas que viven en `CLAUDE.md` se leen en *todas* las ejecuciones, incluidas el 90% que no tiene nada que ver con ellas. La misma regla dentro de una skill solo consume contexto cuando la tarea corresponde. Mover un procedimiento de `CLAUDE.md` a una skill es la forma más sencilla de reducir la configuración sin perder nada: es exactamente lo que proponen `MOVE` y `LOAD-ON-DEMAND` en el informe.
- **Invocarla directamente (`/nombre-de-la-skill`) elimina la lotería del activador.** Dejas de depender de que la descripción coincida con tu frase. Cuando varias skills compiten por el mismo tema, la invocación explícita desempata y elimina la necesidad de escribir una regla de enrutamiento en `CLAUDE.md`.
- **Una skill dentro del proyecto (`.claude/skills/`) queda versionada junto con el código.** El procedimiento acompaña al repo, se incluye en el PR, se puede revisar y no se filtra a otros proyectos.
- **Una skill es la unidad adecuada para corregir problemas.** En el ciclo de Boris, cuando el modelo tropieza tienes tres soluciones: un mejor prompt (la instrucción no era clara), una **skill** (falta un procedimiento repetible) o un MCP (falta contexto al que no puede acceder). Elegir la solución adecuada evita el reflejo de agregar otra regla a `CLAUDE.md`.
- **Es fácil auditar y retirar una skill.** Tiene límites, nombre y alcance propios. Puedes desactivar una skill y medir el resultado; desactivar «ese párrafo en medio de `CLAUDE.md`» es mucho más difícil.

En la práctica, `CLAUDE.md` se queda solo con lo que es verdad **siempre** (identidad, guardrails, fuentes de verdad, seguridad). Todo lo demás — procedimientos, formatos, recetas e integraciones — se convierte en una skill que invocas directamente cuando sabes lo que quieres.

## 7. Ciclo recomendado

1. **Ejecuta** `/audit-ablacao` en el alcance elegido.
2. **Recorta** las 10 principales propuestas según impacto ÷ riesgo (en una sesión aparte; la skill no las aplica).
3. **Úsala en trabajo real** durante unos días. No en pruebas hipotéticas.
4. **Anota los fallos** y vuelve a agregar instrucciones solo cuando se repita el mismo fallo, y en la forma más breve posible.
5. **Repite** cada ~6 meses o con cada lanzamiento de un nuevo modelo importante.

## 8. Guía práctica: publicar en Git y en el portal

Esta es la receta que siguió este mismo repositorio. Sirve para cualquier skill o proyecto de INEMA.

### 8.1 Estructura del repo

```
audit-ablacaocc/
├── SKILL.md        # la skill (la raíz permite instalarla con un cp)
├── README.md       # este archivo
├── capa/capa.png   # portada 1280x720 (skill capa-inema)
├── guia/index.html # landing + guía self-contained (skill projetos-landing-guia)
├── doc/            # material de referencia
└── .github/workflows/pages.yml   # deploy de GitHub Pages mediante Actions
```

Regla para evitar rehacer el trabajo: **la guía va en `guia/`, nunca en la raíz**; la raíz es para el código o la skill. Y es **una página dentro del mismo repo**, nunca un repositorio separado. El nombre del repo remoto es el mismo que el de la carpeta local.

### 8.2 Git: de cero al push

```bash
cd ~/projetos/audit-ablacaocc

git init -b main
git config user.name  "inematds"
git config user.email "inematds@gmail.com"   # el autor corresponde a la cuenta de destino
git add -A
git commit -m "feat: skill audit-ablacao + guia"

gh repo create inematds/audit-ablacaocc --public --source=. --remote=origin --push
```

Una comprobación que ahorra tiempo: revisa `git config user.email` **antes** de hacer el commit. Un autor que no corresponde a la cuenta de destino es la razón nº 1 por la que se bloquean los deploys.

Si el push falla por `.github/workflows/*.yml` (falta el alcance `workflow` en el token HTTPS), envía ese commit por SSH:

```bash
git push git@github.com:inematds/audit-ablacaocc.git main
```

### 8.3 GitHub Pages

Publica **mediante GitHub Actions**, no con el build legacy («deploy from a branch»): la cola legacy se bloquea, con builds atascados en `building` durante horas y el deploy devuelve `Deployment failed, try again later`.

Crea `.github/workflows/pages.yml` para servir desde la raíz (la guía sigue en `/guia/`):

```yaml
name: Deploy static site to Pages
on:
  push: { branches: [main, master] }
  workflow_dispatch:
permissions: { contents: read, pages: write, id-token: write }
concurrency: { group: pages, cancel-in-progress: false }
jobs:
  deploy:
    environment: { name: github-pages, url: "${{ steps.deployment.outputs.page_url }}" }
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/configure-pages@v5
      - uses: actions/upload-pages-artifact@v3
        with: { path: '.' }
      - id: deployment
        uses: actions/deploy-pages@v4
```

Después, configura Pages para usar Actions y sigue la ejecución:

```bash
gh api -X POST repos/inematds/audit-ablacaocc/pages -f build_type=workflow \
  || gh api -X PUT repos/inematds/audit-ablacaocc/pages -f build_type=workflow
gh run watch <run-id> --repo inematds/audit-ablacaocc --exit-status
```

Confirma que está publicado antes de decir que lo publicaste:

```bash
curl -sI https://inematds.github.io/audit-ablacaocc/guia/es/ | head -1   # espera 200
```

La primera ejecución tarda ~1 min. Como la guía está en `guia/`, la URL pública termina en `/guia/`.

### 8.4 Portal (inema.club)

Cuando la URL de Pages responda, publica en las 3 superficies de INEMA (portal, inemabuscas y catálogo PRO):

```
/atualiza-portal https://inematds.github.io/audit-ablacaocc/guia/es/
```

La skill se encarga de la tarjeta, la portada y del commit + push en los tres repositorios.

### 8.5 Definición de «publicado»

Publicar = **commit + push a `origin`**. El deploy es automático mediante el webhook git → Vercel (y GitHub Pages, si corresponde) y está fuera de nuestro control: no abras el dashboard, no revises el estado ni vuelvas a activar el deploy. Se da por terminado cuando el push entra en `origin` y, en el caso de Pages, cuando el archivo responde.

---

## Créditos

Método y visión: Boris Cherny (Anthropic), charla en Y Combinator sobre la reducción del 80% del prompt de sistema de Claude Code. Resúmenes y material de referencia en [`doc/`](doc/). Empaquetado como skill por INEMA.
