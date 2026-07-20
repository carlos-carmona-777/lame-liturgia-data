# lame-liturgia-data

Datos públicos del **Calendario Litúrgico** para la app *La Misa Explicada* (iOS
y Android). El app consulta el `manifest.json` de este repo y descarga los
**calendarios** conforme avanza su ventana de años (estrategia "rolling").

## Qué hay aquí (y qué NO)

**Sí:** los **calendarios** por país (`calendar-<código>.json`) — solo fechas,
nombres de celebración, ciclos y *llaves*. No contienen texto bíblico, así que
son de distribución libre.

**No:** el **texto litúrgico** (lecturas, Liturgia de las Horas). Ese texto (CEE)
tiene derechos y **viaja empacado en el bundle del app**, no por descarga
abierta. Este repo nunca lo publica.

## Archivos

| Archivo | Qué es |
|---|---|
| `manifest.json` | Índice: versión, cobertura, y por calendario su `file`, `bytes`, `sha256`, `coverage`. El app lo lee primero. |
| `calendar-mx.json` | Calendario de México (default). |
| `calendar-general.json` | Calendario General/Romano (default y fallback). |

`baseUrl` en el manifest apunta al Release donde viven los archivos; el app
arma cada URL como `baseUrl + file` y verifica el `sha256` tras descargar.

## Cómo lo consume el app (resumen)

1. Lee `manifest.json` (con throttle, ej. semanal).
2. Compara `coverage` contra la fecha de hoy. Si su ventana local baja de ~3
   años de futuro, descarga el/los `calendar-*.json` que tenga instalados,
   verifica `sha256`, y hace swap atómico en su carpeta escribible.
3. El texto ya está en el bundle; solo se actualiza al actualizar el app
   (compara `text.bundleCoverageTarget`).

## Cómo publicar una versión nueva

1. Regenerar el dataset (en `migracion-breviario/`):
   ```
   node generate.mjs --countries=mx,general --startYear=2026 --endYear=2040
   BASE_URL="https://github.com/<owner>/lame-liturgia-data/releases/download/<tag>/" \
       python3 build-hosting-manifest.py
   ```
   Esto reescribe `calendar-*.json` y `manifest.json` en esta carpeta.
2. Crear un **Release** en GitHub (tag ej. `2026-2040-r1`) y subir como assets:
   `manifest.json`, `calendar-mx.json`, `calendar-general.json`.
3. Asegurar que `baseUrl` del manifest coincida con la URL de descarga del tag.

> Recomendado: un **GitHub Action** que corra el pipeline en agenda (anual) y
> publique el Release automáticamente, para que los fixes del calendario sigan
> fluyendo sin regenerar 100 años a mano.

## Regenerar / mantener

El calendario se calcula con **romcal** + **breviarium** en el repo del pipeline
(`migracion-breviario/`). Los fixes (traslados de solemnidades, etc.) viven ahí;
al regenerar, se aplican a **todos** los años. Ver `migracion-breviario/COMOFUNCIONA.md`.
