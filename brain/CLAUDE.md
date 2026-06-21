# brain — Schema operativo

Este directorio es el "brain" del repo: un wiki markdown interconectado mantenido por Claude.
Si estás leyendo este archivo, probablemente fuiste invocado por el comando `/brain`.
Seguí estas reglas al pie de la letra.

---

## Qué es cada archivo

- `index.md` — catálogo denso de nodos + panorama del repo + frontmatter de malla cross-repo. Punto de entrada para cualquier query o ingest.
- `log.md` — bitácora append-only de **operaciones sobre el wiki** (ingest, lint, rename, merge). No es el log de trabajo — eso vive en los nodos.
- `sources.md` — registro de fuentes (código del repo y URLs). Directorio, no cache de contenido.
- `sessions/` — nodos de sesión. Planos, un archivo por sesión.
- `CLAUDE.md` — este archivo.

---

## Tipos de nodo

Al arranque sólo existe **session**. Conceptos, ADRs, entidades y personas emergerán
orgánicamente vía `lint` cuando el patrón se repita en 3+ nodos.

**NO** crear nodos de otros tipos preventivamente.

---

## Frontmatter obligatorio en cada nodo session

```yaml
---
type: session
area: <ruta-del-subsistema, ej: api | db | deploy | infra>
date: YYYY-MM-DD
slug: <slug-del-nombre-de-archivo-sin-fecha-ni-.md>
title: "<título humano>"
tags: [<libres, kebab-case>]
status: active                  # active | superseded | archived
related:
  - <slug-sin-.md>
sources:
  - repo:<path-relativo>
superseded_by: null             # slug del nodo que reemplaza a éste, o null
---
```

Reglas:
- `slug`: igual al nombre de archivo sin fecha ni `.md`. Kebab-case, ≤6 palabras. Describe el resultado, no el proceso.
- `area`: ruta relativa inequívoca. Single-value. Si una sesión toca dos áreas, crear dos nodos cross-linked, o uno con área primaria y cross-ref.
- `related`: slugs sin `.md`. Máx. 5. El LLM los detecta por tags solapados, misma área, proximidad cronológica.
- `sources`: prefijos válidos: `repo:<path-relativo>`, `url:<url-completa>`.

---

## Convención de enlaces — Wikilinks Foam

**Regla crítica**: todos los enlaces internos del wiki usan `[[slug]]`, compatible con Foam/Markdown Notes en VSCode.

- **Sintaxis**: `[[slug]]` — solo el nombre del archivo sin `.md`, sin path, sin alias. Ej: `[[2026-06-21-genesis-analisis]]`.
- **Resolución**: Foam resuelve el slug al archivo por nombre. Los slugs son únicos por construcción (incluyen fecha).
- **Dónde se usan**:
  - `index.md`: cada entry del catálogo
  - Nodos: sección `## Relacionados`
  - Nodos: sección `## Fuentes` cuando apunta a otro nodo del wiki o a un anchor interno (`[[sources#id]]`)
- **Dónde NO se usan**:
  - URLs externas: usar markdown link `[texto](url)`
  - Referencias a OTROS repos: usar coordenadas `owner/repo` (ver Malla cross-repo) — los wikilinks no cruzan repos
  - Archivos fuera del vault `brain/`: backticks literales

---

## Cuerpo obligatorio de cada nodo session

Secciones en este orden exacto:

```
# <Título de la sesión>

## Contexto

## Decisiones

## Output

## Pendientes

## Relacionados
- [[slug]] — razón en una línea

## Fuentes
- [[sources#id]]
- Origen histórico (no modificar): `ruta/original.md`
```

Reglas del cuerpo:
- `Relacionados`: **sólo wikilinks** a otros nodos del wiki. Cada bullet con razón en una línea. Sin razón → candidato orphan-link en lint.
- `Fuentes`: referencias externas. Para archivos del repo: `repo:<path>` o `[[sources#id]]`. Para URLs web: `[texto](url)`.

---

## Malla cross-repo (relations)

Este brain es independiente y vive dentro de su repo. La conexión con los brains de
otros repos NO se guarda en un hub central: se declara por referencia y se resuelve en
vivo con el token local + la API de GitHub.

- Las relaciones viven en el frontmatter de `index.md`, campo `relations:`, como
  coordenadas `owner/repo` (ej: `Boxeles/Boxels-Pip-App`).
- Para seguir una relación: `GET /repos/<owner>/<repo>/contents/brain/index.md` con el
  token del credential store local. Trae el brain del repo hermano sin clonarlo.
- El conjunto completo de brains se descubre por API: `GET /orgs/Boxeles/repos` +
  `GET /user/repos?affiliation=owner` (filtrando owner=sunshine772), revisando qué
  repos tienen carpeta `brain/`.
- Bidireccionalidad: si este repo referencia a otro, el otro referencia a este. Son
  simétricas por dominio.
- Dentro del repo, los enlaces son wikilinks `[[slug]]`. Entre repos, son referencias
  `owner/repo`.

---

## Reglas de ingest

Cuando se invoca `/brain ingest`:

1. Leer `brain/CLAUDE.md` y `brain/index.md`.
2. Revisar la conversación actual. Identificar: área(s) tocadas, verbo de acción principal, output concreto (archivos/URLs), decisiones con rationale, pendientes explícitos.
3. Generar el slug: `YYYY-MM-DD-<kebab-case>`. Fecha = hoy. El slug describe el RESULTADO, máx. 6 palabras.
4. ¿Existe nodo del mismo día con tema muy similar? **Sí** → UPDATE bajo `### Actualización [HH:MM]`, no reescribir. **No** → CREATE.
5. CREATE: frontmatter completo. `related` busca en `index.md` nodos con tags solapados, misma área o proximidad cronológica. Máx. 5.
6. Cuerpo con las 6 secciones. Cross-refs con `[[slug]]` y razón en una línea.
7. **Bidireccionalidad obligatoria**: para cada nodo en `related`, abrirlo y agregar bullet recíproco en su `## Relacionados`.
8. Actualizar `brain/index.md`: insertar `[[slug]] — one-liner` al tope de la sección del área.
9. Append a `brain/log.md`: `## [YYYY-MM-DD HH:MM] ingest | <slug>`.
10. Reportar: path del nodo, cross-refs agregadas, decisiones ambiguas.

---

## Reglas de query

Cuando se invoca `/brain query <pregunta>`:

1. Leer `brain/CLAUDE.md` y `brain/index.md` completos.
2. Seleccionar 1-5 nodos candidatos de los one-liners del índice.
3. Leer esos nodos completos.
4. Para cada `source` con prefijo `repo:<path>`, leer el archivo real del repo antes de afirmar su contenido. `sources.md` es un directorio, no un cache autoritativo. Si la pregunta cruza a un repo hermano (`relations`), resolverlo por API con el token local.
5. Sintetizar en 2-5 párrafos con citations en wikilinks `[[slug]]`. **NO** markdown links para nodos del wiki.
6. Si detectás un gap, NO lo arregles — reportalo como "Sugerencia para `/brain lint`".
7. Si no hay info suficiente, decirlo. **No inventar**.

---

## Reglas de lint

Cuando se invoca `/brain lint`:

1. Leer `brain/CLAUDE.md`, `brain/index.md` y **todos** los archivos en `brain/sessions/`.
2. Revisar: orphan nodes, broken wikilinks, stale claims, missing cross-refs, conceptos emergentes (3+ nodos), contradicciones sin `superseded_by`, frontmatter inválido, índice desincronizado, markdown links mal usados.
3. **Malla cross-repo**: validar que cada `owner/repo` de `relations` exista y resuelva por API, y que la relación sea recíproca en el repo hermano.
4. Devolver reporte markdown estructurado por categoría con sugerencia accionable por ítem.
5. **NO modificar archivos.** Solo append de una entrada corta a `log.md` con conteos.

---

## Fuente de verdad — código del repo

El código del propio repo es la fuente de verdad. Referenciá archivos con `repo:<path>`.
Leé siempre el archivo real antes de afirmar su contenido; `sources.md` sólo lista qué
existe y dónde.

---

## Idioma

Todo el contenido del wiki se escribe en español. Los identificadores de código
(variables, funciones, paths, comandos) van en inglés, tal cual aparecen en el repo.
