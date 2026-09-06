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

La documentación heredada de la plantilla (`plan-flujo-movil*.md`, `fase0-resultado.md`, `fase3-fly.md`, `plantilla.md`, `sesiones/2026090[56]-*`) se movió a `docs/plantilla/`, de forma que `docs/planificacion/` queda para la planificación de este proyecto. `ARRANQUE.md` se conserva como registro de cómo se creó el repo, con un aviso al principio.

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

Todas corregidas en el commit siguiente al de la inicialización. La corrección de fondo es
**`tools/inicializar.sh`**: un script que hace las sustituciones en todos los sitios, aparta la
documentación heredada, regenera el `README` y **falla con código 1 si queda cualquier residuo** de la
plantilla. Se probó desatendido sobre el árbol del commit inicial (`2c3f410`) con parámetros ficticios:
sin residuos, `cargo build --release` en verde con el paquete renombrado, y los dos workflows y los dos
TOML parseando.

Estos arreglos están en este repo y portados a la plantilla real en el PR [npiobject/DesdeMovil#2](https://github.com/npiobject/DesdeMovil/pull/2), pendiente de revisión.

1. **[CORREGIDO] La lista de ficheros a sustituir estaba incompleta.** El paso 1 del bloque a pegar nombra `app/fly.toml`, `.github/workflows/deploy.yml`, `CLAUDE.md`, `tools/aterrizar.ps1` y `tools/estado.ps1`. Faltan `app/Cargo.toml` y `app/Dockerfile`, que **sí** llevan marcador `PLANTILLA:`, y faltan tres ficheros que llevan el nombre del proyecto y **no** llevan marcador ninguno: `README.md`, `docs/index.html` y `app/src/main.rs`. Un agente que se limite a la lista deja el nombre viejo en el mock público y en la respuesta de `GET /`. → El script cubre los doce sitios y `ARRANQUE.md` los tabula, marcando cuáles no llevan marcador.
2. **[CORREGIDO] `grep "PLANTILLA:"` no es un checklist fiable.** El propio `ARRANQUE.md` y `docs/planificacion/plantilla.md` contienen la cadena, así que el grep mezcla marcadores accionables con prosa que los menciona. Y `plantilla.md` habla de "los diez sitios" mientras su tabla lista ocho ficheros; ni el número ni la tabla cuadran con lo que devuelve el grep. → El checklist deja de ser un grep manual: lo hace el script al final, con exclusiones explícitas, y aborta si encuentra algo.
3. **[CORREGIDO] El prefijo del número de build no estaba parametrizado.** `CLAUDE.md` exige `<meta name="build" content="DM-B3-AAAAMMDD-NNN">`, donde `DM` viene de *DesdeMovil* y `B3` de una fase de aquel proyecto. Nada decía qué prefijo usar en un proyecto nuevo; aquí se usó `PP-B1`. **[SUPUESTO]** que el prefijo es libre por proyecto; plan B si debe ser estable entre proyectos: volver a un prefijo fijo y renumerar. → El script lo deriva de las iniciales del nombre (`MiProyecto` → `MP-B1`) y admite `--prefijo`.
4. **[CORREGIDO] No decía qué hacer con `docs/planificacion/` heredado.** El repo hijo nace con la planificación completa de `DesdeMovil` (cuatro documentos y tres resúmenes de sesión) mezclada con la carpeta donde va la del proyecto nuevo. → El script mueve ese material a `docs/plantilla/` y deja `docs/planificacion/` vacío, y `CLAUDE.md` lo dice.
5. **[CORREGIDO] El mensaje de error de Pages que documentaba no es el que sale.** `ARRANQUE.md` anuncia `Get Pages site failed… Not Found`. El fallo real del commit inicial de este repo (run [34023384474](https://github.com/npiobject/PruebaPlantilla/actions/runs/34023384474)) fue:
   `Create Pages site failed. Error: Resource not accessible by integration`.
   El `Get Pages site failed… Not Found` aparece solo como *warning* previo. Es decir: `enablement: true` en `configure-pages` **no** sustituye al paso manual, porque el `GITHUB_TOKEN` no tiene permiso para crear el sitio. → `ARRANQUE.md` ya cita el error real y explica por qué `enablement: true` no basta.
6. **[CORREGIDO] El primer commit de la plantilla deja dos runs en rojo antes de empezar.** *Use this template* dispara `pages.yml` y `deploy.yml` de inmediato, y ambos fallan si los pasos manuales aún no están hechos (aquí: Pages sin activar y `FLY_API_TOKEN` ausente). No es un problema real. → `ARRANQUE.md` lo avisa ahora en un recuadro.
7. **[CORREGIDO] No advertía de que el nombre de app de Fly es único en todo Fly.io.** `flyctl apps create … || true` tolera que ya exista, pero si el nombre está tomado por otra cuenta el fallo aparece más tarde, en el `deploy`, y con un mensaje que no apunta a la causa. → Añadida esa advertencia al paso 3.
8. **[CORREGIDO] Choca con la rama de trabajo de las sesiones de Code.** La sesión llega con una rama `claude/…` designada y con la regla de no empujar a otra, mientras que `ARRANQUE.md` y `CLAUDE.md` exigen push a `main` — que además es la única rama que dispara los dos workflows. Sin la instrucción explícita "haz push a main" en el prompt, la inicialización no se puede verificar. → El bloque a pegar dice ahora "haz commit y push **a main**" y explica por qué.
9. **[CORREGIDO] Nada decía qué hacer con `ARRANQUE.md` en el repo hijo.** Una vez inicializado el proyecto, ese fichero solo describe cómo se creó. → Se conserva, enlazado desde el `README` y con un aviso al principio del propio `ARRANQUE.md` para quien lo abra en un repo ya inicializado.
10. **[CORREGIDO] El id de Drive solo se usa en `CLAUDE.md`.** El paso 1 pedía "actualiza también el id de Drive" sin decir dónde. → Lo pone el script y `ARRANQUE.md` lo dice explícitamente: es solo `CLAUDE.md`.

## Limitaciones conocidas

- **[SUPUESTO] Los scripts de PowerShell siguen sin ejecutarse.** El sandbox no tiene `pwsh`; el cambio ha sido solo de los valores por defecto de `param()`. Plan B si algo falla: pasar `-Proyecto` y `-Root` explícitos.
- Las tres URLs de "Estado final" no se han abierto desde la sesión: el sandbox no alcanza `*.github.io` ni `*.fly.dev`. La evidencia es el `deploy-pages` del run de Pages y el `curl` del paso *Verificar /salud* del run de Fly, ambos ejecutados en el runner.
- `actions/checkout` subido a `v5` en los dos workflows, lo que quita ese aviso de deprecación de Node 20. `actions/configure-pages@v5` sigue avisando: no hay versión posterior.
- **Pendiente:** revisar y mezclar el PR [npiobject/DesdeMovil#2](https://github.com/npiobject/DesdeMovil/pull/2), que lleva `tools/inicializar.sh`, el `ARRANQUE.md` reescrito y `checkout@v5` a la plantilla real. Allí el script se probó de nuevo sobre el árbol de la propia plantilla (`--nombre CasaVerde`): sin residuos y `cargo build --release` en verde.
