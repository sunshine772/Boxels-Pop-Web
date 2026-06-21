---
type: session
area: panel
date: 2026-06-21
slug: genesis-analisis
title: "Análisis inicial y génesis del brain"
tags: [genesis, angular, primeng, cine]
status: active
related: []
sources:
  - repo:package.json
  - repo:src/app/app-routing.module.ts
  - repo:src/environments/environment.development.ts
  - repo:src/app/api/services/api
superseded_by: null
---

# Análisis inicial y génesis del brain

## Contexto

Repo `Boxels-Pop-Web` (paquete `pop`, v0.0.0). SPA Angular 17 sobre el template PrimeNG
Sakai-NG. Es el front del producto POP, un catálogo de cine. Consume la API en
`http://localhost:8000/api/v1` (backend Pop-Server, FastAPI). Auth JWT con
`@auth0/angular-jwt`, `AuthGuard` e interceptor activos. Dominio: pelicula, genero,
etiqueta, comentario, imagen — administrados desde el área `panel`.

## Decisiones

- Esquema de brain adoptado: índice + nodos de sesión + log + sources (Foam/wikilinks).
- Relations del cluster `pop`: [[Boxels-Pop-Server]] (backend que sirve este front).
- Madurez clasificada como `dev` (apiUrl a localhost, sin deploy versionado).

## Output

- `brain/index.md` (panorama, stack, estado, áreas, malla cross-repo).
- `brain/log.md`, `brain/sources.md`, `brain/CLAUDE.md`.
- `brain/sessions/2026-06-21-genesis-analisis.md` (este nodo).
- `.claude/commands/brain.md`, `.vscode/settings.json`.

## Pendientes

- Ingestar sesiones reales de trabajo a medida que avance el desarrollo.
- Definir `apiUrl` de producción en `environment.ts` (hoy vacío).

## Relacionados

- (sin nodos hermanos todavía — primer nodo del wiki)

## Fuentes

- repo:package.json
- repo:src/app/app-routing.module.ts
- repo:src/environments/environment.development.ts
