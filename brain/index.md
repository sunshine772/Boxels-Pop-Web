---
node_type: brain-index
repo: sunshine772/Boxels-Pop-Web
domain: pop
relations:
  - sunshine772/Boxels-Pop-Server
updated: 2026-06-21
---

# Boxels-Pop-Web — Brain

## Panorama del repo

Front-office/back-office web de **POP**, un catálogo de cine (películas). Es una SPA
**Angular 17** construida sobre el template **PrimeNG Sakai-NG** (layout, menú, dashboard).
El paquete se llama `pop` (version 0.0.0). Consume una API REST en `apiUrl`
(`http://localhost:8000/api/v1`, ver `repo:src/environments/environment.development.ts`),
que corresponde al backend [[Boxels-Pop-Server]] (FastAPI en :8000). Autenticación por
**JWT** con `@auth0/angular-jwt` + `AuthGuard` + `auth.interceptor`.

El dominio es gestión de un catálogo de películas: **pelicula**, **genero**, **etiqueta**,
**comentario** e **imagen**, administrados desde el área `panel`. Branding de cine
(componente `popcorn-background`, `particles.js`, editor `quill`, `chart.js`,
`@fullcalendar`).

## Stack & arquitectura

- **Lenguaje/Framework:** Angular 17 (TypeScript 5.2), PrimeNG 17.2 + PrimeFlex (Sakai-NG).
- **Entrypoints:** `repo:src/main.ts`, `repo:src/app/app-routing.module.ts` (rutas raíz protegidas por `AuthGuard`: `''`→dashboard, `panel`→PanelModule; `auth`→login/access/error).
- **Componentes de panel:** `repo:src/app/api/components/panel/` — pelicula, genero, etiqueta, comentario (CRUD).
- **Servicios API:** `repo:src/app/api/services/api/` — pelicula, genero, etiqueta, comentario, imagen; auth en `services/auth/` (auth.service, storage.service).
- **Modelos:** `repo:src/app/api/models/` — pelicula, genero, etiqueta, comentario, imagen, token.
- **Auth:** `@auth0/angular-jwt`, `repo:src/app/api/guards/auth.guard.ts`, `repo:src/app/api/interceptors/auth/auth.interceptor.ts`.
- **Datos:** ninguno propio; estado en backend vía REST. Token en localStorage (storage.service).
- **Deploy/infra:** sin Docker/CI en el repo; `ng build`. `environment.ts` (prod) sin `apiUrl` definido (pendiente apuntar a backend real).

## Estado actual

Madurez **dev**. App scaffoldeada sobre Sakai-NG con el CRUD de catálogo de cine cableado
(pelicula/genero/etiqueta/comentario/imagen) y auth JWT funcional (guard + interceptor
activos). `apiUrl` apunta a `localhost:8000` (desarrollo); `environment.ts` de producción
no define `apiUrl` todavía. Sin pipeline de deploy versionado.

## Áreas

- `auth` — login/access/error, guard e interceptor JWT.
- `panel` — CRUD de pelicula, genero, etiqueta, comentario.
- `services/api` — clientes REST por entidad contra Pop-Server.
- `layout` — shell Sakai-NG (menú, topbar, dashboard).

## Nodos

- [[2026-06-21-genesis-analisis]] — análisis inicial del repo y creación del brain

## Malla cross-repo

> Resolver vía API de GitHub con el token local: GET /repos/<owner>/<repo>/contents/brain/index.md

- [sunshine772/Boxels-Pop-Server](https://github.com/sunshine772/Boxels-Pop-Server/blob/HEAD/brain/index.md) — backend REST (:8000/api/v1) que sirve el catálogo de cine que este front administra
