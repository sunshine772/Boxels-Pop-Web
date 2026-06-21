---
description: Opera el wiki interno de Boxels-Pop-Web. Subcomandos: ingest (default), query, lint.
argument-hint: "[ingest | query <pregunta> | lint]"
---

Sos el operador del wiki de Boxels-Pop-Web. El wiki es un knowledge base markdown vivo
en `brain/`. **Antes de hacer nada, leé `brain/CLAUDE.md` completo** — ese archivo es tu
manual operativo con todas las reglas de frontmatter, wikilinks, malla cross-repo y operaciones.

---

## Detectar sub-operación

Parseá `$ARGUMENTS`:
- Vacío o `ingest` → modo **INGEST**.
- Empieza con `query` → modo **QUERY** con el resto como pregunta.
- `lint` → modo **LINT**.
- Cualquier otra cosa → pedir aclaración y abortar.

---

## Modo INGEST

Objetivo: crear (o actualizar) un nodo de sesión en `brain/sessions/` que capture lo que
pasó en ESTA conversación.

1. Leé `brain/CLAUDE.md` y `brain/index.md`.
2. Revisá la conversación actual. Identificá área(s), verbo de acción principal, output concreto, decisiones con rationale, pendientes.
3. Generá el slug: `YYYY-MM-DD-<kebab-case-corto>`. Fecha = hoy. Describe el RESULTADO, máx. 6 palabras.
4. ¿Existe ya un nodo con esa fecha y tema similar? **Sí** → UPDATE bajo `### Actualización [HH:MM]`. **No** → CREATE.
5. CREATE: frontmatter completo según `brain/CLAUDE.md`. `related` (máx. 5) por tags/área/cronología. `sources` con prefijo `repo:` o `url:`.
6. Cuerpo con las 6 secciones. En `Relacionados` usá `[[slug]]` con razón en una línea.
7. **Bidireccionalidad obligatoria**: para cada nodo en `related`, agregá bullet recíproco en su `## Relacionados`.
8. Actualizá `brain/index.md`: insertá `[[slug]] — one-liner` al tope de la sección del área.
9. Append a `brain/log.md`:
   ```
   ## [YYYY-MM-DD HH:MM] ingest | <slug>
   - Área: <area>
   - Cross-refs: <slugs o "ninguna">
   ```
10. Reportá: path del nodo, cross-refs agregadas, decisiones ambiguas.

---

## Modo QUERY

1. Leé `brain/CLAUDE.md` y `brain/index.md` completos.
2. Seleccioná 1-5 nodos candidatos. Si ninguno es relevante, decilo.
3. Leé esos nodos completos.
4. Para cada `repo:<path>` citado, leé el archivo real. Si la pregunta cruza a un repo hermano (`relations` del index), resolvelo por API con el token local del credential store.
5. Sintetizá en 2-5 párrafos. Citations con wikilinks `[[slug]]`. **NO** markdown links para nodos del wiki.
6. Gap detectado → reportar como "Sugerencia para `/brain lint`", no arreglar.
7. Sin info suficiente → decirlo. **No inventes**.

---

## Modo LINT

1. Leé `brain/CLAUDE.md`, `brain/index.md` y **todos** los archivos en `brain/sessions/`.
2. Revisá cada categoría según las reglas de lint de `brain/CLAUDE.md`, incluida la **malla cross-repo** (cada `owner/repo` de `relations` existe, resuelve por API y es recíproco).
3. Devolvé un reporte markdown estructurado (Resumen / Críticos / Medios / Oportunidades).
4. **NO modificar archivos.**
5. Append a `brain/log.md`:
   ```
   ## [YYYY-MM-DD HH:MM] lint | report
   - Críticos: N | Medios: N | Oportunidades: N
   ```
