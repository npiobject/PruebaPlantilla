# PruebaPlantilla — inicialización desde la plantilla

**Fecha:** 2026-09-06
**SHA de la inicialización:** `7ee507a3cf2fd4ff009a9abd41ec79430d751a60`
**Repo:** `npiobject/PruebaPlantilla`, rama `main`

Primera ejecución real de `ARRANQUE.md` sobre un repo creado con *Use this template*. El objetivo declarado de la prueba no era solo dejar los dos despliegues vivos, sino **medir la fricción de la plantilla**; esa lista está en la última sección.

## Parámetros aplicados

| Parámetro | Valor |
|---|---|
| Nombre del proyecto | `PruebaPlantilla` |
| Owner de GitHub | `npiobject` |
| App de Fly.io | `pruebaplantilla-npi` (región `cdg`) |
| Carpeta de Drive | `Mi unidad/PruebaPlantilla`, id `1EUL0hBVLSHsUWz9auVSXF7J-Z90hNqXl` |

## Qué se sustituyó

| Fichero | Cambio |
|---|---|
| `app/fly.toml` | `app = "pruebaplantilla-npi"`; comentarios `PLANTILLA:` resueltos |
| `.github/workflows/deploy.yml` | `FLY_APP: pruebaplantilla-npi`; comentarios `PLANTILLA:` resueltos |
| `.github/workflows/pages.yml` | Eliminado el comentario `PLANTILLA:`; se conserva la nota de *Settings → Pages* |
| `app/Cargo.toml` + `app/Cargo.lock` | Paquete renombrado a `pruebaplantilla-backend` |
| `app/Dockerfile` | Ruta del binario y usuario del runtime (`pruebaplantilla`) |
| `app/src/main.rs` | Cadenas de la raíz y del log: `PruebaPlantilla backend` |
| `docs/index.html` | Título, `h1` y `meta name="build"` → `PP-B1-20260906-001` |
| `CLAUDE.md` | Reescrito: nombre, repo, id de Drive, dos URLs vivas, nombre del paquete, prefijo de build y nota sobre la documentación heredada |
| `README.md` | Reescrito como README del proyecto, con las dos URLs vivas |
| `tools/aterrizar.ps1`, `tools/estado.ps1` | `-Proyecto` (y `-Owner`) por defecto |

No se tocó `docs/planificacion/` heredado (`plan-flujo-movil*.md`, `fase0-resultado.md`, `fase3-fly.md`, `plantilla.md`, `sesiones/2026090*`): es historial de la plantilla `DesdeMovil`, no de este proyecto. Se conserva como referencia y así queda anotado en `CLAUDE.md`. `ARRANQUE.md` también se conserva, sin tocar, como documentación de cómo se arranca un proyecto desde esta plantilla.

## Verificación

Comprobaciones en la sesión antes del push: `cargo build --release` en verde con el paquete renombrado, los dos workflows parseados con `yaml.safe_load`, y `app/fly.toml` y `app/Cargo.toml` con `tomllib`.

Runs del SHA `7ee507a`, ambos en `success`:

| Run | Workflow | Resultado |
|---|---|---|
| [34024493221](https://github.com/npiobject/PruebaPlantilla/actions/runs/34024493221) | Deploy docs to GitHub Pages | success |
| [34024493213](https://github.com/npiobject/PruebaPlantilla/actions/runs/34024493213) | Desplegar backend en Fly.io | success (1 min 9 s) |

Paso *Verificar /salud* del despliegue, al primer intento:

```
Verificando https://pruebaplantilla-npi.fly.dev/salud (se espera build=7ee507a3cf2fd4ff009a9abd41ec79430d751a60)
Intento 1/10: {"build":"7ee507a3cf2fd4ff009a9abd41ec79430d751a60","ok":true}
OK: /salud responde con el build 7ee507a3cf2fd4ff009a9abd41ec79430d751a60
```

## Estado final

| Qué | URL |
|---|---|
| Mock (Pages) | https://npiobject.github.io/PruebaPlantilla/ |
| Backend (Fly) | https://pruebaplantilla-npi.fly.dev/ |
| Salud | https://pruebaplantilla-npi.fly.dev/salud → `{"build":"7ee507a3cf2fd4ff009a9abd41ec79430d751a60","ok":true}` |

## Fricciones detectadas en `ARRANQUE.md`

Ordenadas por coste de corregirlas.

1. **La lista de ficheros a sustituir está incompleta.** El paso 1 del bloque a pegar nombra `app/fly.toml`, `.github/workflows/deploy.yml`, `CLAUDE.md`, `tools/aterrizar.ps1` y `tools/estado.ps1`. Faltan `app/Cargo.toml` y `app/Dockerfile`, que **sí** llevan marcador `PLANTILLA:`, y faltan tres ficheros que llevan el nombre del proyecto y **no** llevan marcador ninguno: `README.md`, `docs/index.html` y `app/src/main.rs`. Un agente que se limite a la lista deja el nombre viejo en el mock público y en la respuesta de `GET /`.
2. **`grep "PLANTILLA:"` no es un checklist fiable.** El propio `ARRANQUE.md` y `docs/planificacion/plantilla.md` contienen la cadena, así que el grep mezcla marcadores accionables con prosa que los menciona. Y `plantilla.md` habla de "los diez sitios" mientras su tabla lista ocho ficheros; ni el número ni la tabla cuadran con lo que devuelve el grep.
3. **El prefijo del número de build no está parametrizado.** `CLAUDE.md` exige `<meta name="build" content="DM-B3-AAAAMMDD-NNN">`, donde `DM` viene de *DesdeMovil* y `B3` de una fase de aquel proyecto. Nada dice qué prefijo usar en un proyecto nuevo. Decidido aquí: `PP-B1`, y `CLAUDE.md` lo documenta ya con el prefijo propio. **[SUPUESTO]** que el prefijo es libre por proyecto; plan B si resulta que debe ser estable entre proyectos: volver a `DM-B3` y renumerar.
4. **No dice qué hacer con `docs/planificacion/` heredado.** El repo hijo nace con la planificación completa de `DesdeMovil` (cuatro documentos y tres resúmenes de sesión) mezclada con la carpeta donde va la del proyecto nuevo. Decidido aquí: conservar y anotarlo en `CLAUDE.md`. Alternativa razonable: que la plantilla mueva ese material a `docs/plantilla/` para que `docs/planificacion/` nazca vacío.
5. **El mensaje de error de Pages que documenta no es el que sale.** `ARRANQUE.md` anuncia `Get Pages site failed… Not Found`. El fallo real del commit inicial de este repo (run [34023384474](https://github.com/npiobject/PruebaPlantilla/actions/runs/34023384474)) fue:
   `Create Pages site failed. Error: Resource not accessible by integration`.
   El `Get Pages site failed… Not Found` aparece solo como *warning* previo. Es decir: `enablement: true` en `configure-pages` **no** sustituye al paso manual, porque el `GITHUB_TOKEN` no tiene permiso para crear el sitio. Conviene decirlo así de explícito.
6. **El primer commit de la plantilla deja dos runs en rojo antes de empezar.** *Use this template* dispara `pages.yml` y `deploy.yml` de inmediato, y ambos fallan si los pasos manuales aún no están hechos (aquí: Pages sin activar y `FLY_API_TOKEN` ausente). No es un problema real, pero `ARRANQUE.md` no lo avisa y el historial de Actions arranca en rojo.
7. **No advierte de que el nombre de app de Fly es único en todo Fly.io.** `flyctl apps create … || true` tolera que ya exista, pero si el nombre está tomado por otra cuenta el fallo aparece más tarde, en el `deploy`, y con un mensaje que no apunta a la causa. Merece una línea en el paso 3.
8. **Choca con la rama de trabajo de las sesiones de Code.** La sesión llega con una rama `claude/…` designada y con la regla de no empujar a otra, mientras que `ARRANQUE.md` y `CLAUDE.md` exigen push a `main` — que además es la única rama que dispara los dos workflows. Sin la instrucción explícita "haz push a main" en el prompt, la inicialización no se puede verificar. Conviene que el bloque a pegar de `ARRANQUE.md` lo diga textualmente.
9. **Nada dice qué hacer con `ARRANQUE.md` en el repo hijo.** Una vez inicializado el proyecto, ese fichero solo describe cómo se creó. Conservado y enlazado desde el `README`; sería igual de defendible borrarlo.
10. **El id de Drive solo se usa en `CLAUDE.md`.** El paso 1 pide "actualiza también el id de Drive", pero no dice dónde: es solo `CLAUDE.md`, y no lleva marcador `PLANTILLA:`. Una línea evitaría la búsqueda.

## Limitaciones conocidas

- **[SUPUESTO] Los scripts de PowerShell siguen sin ejecutarse.** El sandbox no tiene `pwsh`; el cambio ha sido solo de los valores por defecto de `param()`. Plan B si algo falla: pasar `-Proyecto` y `-Root` explícitos.
- Las tres URLs de "Estado final" no se han abierto desde la sesión: el sandbox no alcanza `*.github.io` ni `*.fly.dev`. La evidencia es el `deploy-pages` del run de Pages y el `curl` del paso *Verificar /salud* del run de Fly, ambos ejecutados en el runner.
- `actions/checkout@v4` y `actions/configure-pages@v5` siguen avisando de la deprecación de Node 20. No rompe nada hoy.
