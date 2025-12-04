
---

## CHANGELOG.md final

```markdown
# Changelog

Todas las versiones de `zbx-raid-all`.  
Formato: `Versión — Fecha — Resumen`.

---

## 1.0-1 — 2025-12-04

### Añadido

- Versión inicial del paquete `zbx-raid-all`.
- Arquitectura de recolección unificada de pools:
  - Script coordinador `/usr/local/sbin/zbx_raid_collect.sh`.
  - Directorio de datos `/var/lib/zbx-raid/`.
  - Cron `/etc/cron.d/zbx-raid`.
  - UserParameters Zabbix en `/etc/zabbix/zabbix_agent2.d/zbx-raid.conf`.
- Modelo de datos:
  - Fichero `pools_discovery.json` con LLD unificada de pools.
  - Ficheros `pool_<POOLID>.state` que contienen **solo el estado normalizado** del pool (`OK`, `DEGRADED`, `OFFLINE`, `FAILED`, etc.).
- Backend ZFS:
  - Script `/usr/lib/zbx-raid/backend_zfs.sh`.
  - Detección automática de zpools mediante `zpool list`.
  - Normalización de estados:
    - `ONLINE` → `OK`
    - `DEGRADED` → `DEGRADED`
    - `FAULTED` / `UNAVAIL` / `REMOVED` → `FAILED`
    - `OFFLINE` → `OFFLINE`.
- Backend HP Smart Array (ssacli):
  - Script `/usr/lib/zbx-raid/backend_ssacli.sh`.
  - Parseo de `ssacli ctrl all show config detail`.
  - Generación de identificadores `ssacli_c<slot>_ld<LD>` (ej. `ssacli_c4_ld1`).
  - Normalización de `Status:` del logical drive:
    - `OK` → `OK`
    - Estados de degradado / recuperación / rebuild → `DEGRADED`
    - `Offline` → `OFFLINE`
    - Estados con `Fail` → `FAILED`.
- Backend LSI/DELL (storcli64):
  - Script `/usr/lib/zbx-raid/backend_storcli.sh`.
  - Parseo de `storcli64 /cALL /vall show all`.
  - Generación de identificadores `storcli_c<ctrl>_vd<VD>` (ej. `storcli_c0_vd0`).
  - Normalización de `State` del virtual drive:
    - `Optl` / `Optimal` → `OK`
    - `Pdgd`, `Dgrd`, `Rec`, `Rbld`, `Rebuild`, `Rebuilding` → `DEGRADED`
    - `OfLn`, `Offln`, `Offline` → `OFFLINE`
    - Estados con `Fail` → `FAILED`.

### Integración con Zabbix

- Template recomendado `Template RAID Pools by Agent`:
  - Regla de descubrimiento `zbx.storage.pool.discovery`.
  - Protótipo de item `Pool {#POOLID} status` (texto) con `zbx.storage.pool.state[{#POOLID}]`.
- Ítem dependiente numérico:
  - `zbx.storage.pool.state.code[{#POOLID}]` (dependent item).
  - Preprocesado JavaScript para mapear:
    - `OK` → `0`
    - `DEGRADED` → `1`
    - `OFFLINE` → `2`
    - `FAILED` → `3`
    - otros → `4`.
  - Value mapping sugerido `Storage pool state code` para mostrar texto desde el valor numérico.
- Triggers sugeridos:
  - Pool `{#POOLID}` degradado → estado `DEGRADED` (WARNING).
  - Pool `{#POOLID}` fallado → estado `FAILED` (DISASTER).
  - Pool `{#POOLID}` no OK → estado distinto de `OK`.

### Visualización

- Integración pensada para Zabbix 7.0:
  - Uso del item numérico `zbx.storage.pool.state.code[{#POOLID}]` en widgets tipo **Item value / Hex**.
  - Rangos de color recomendados:
    - `0` → verde (OK)
    - `1` → amarillo/naranja (DEGRADED)
    - `2` → gris (OFFLINE)
    - `3` → rojo (FAILED)
    - `>=4` → color especial (UNKNOWN).
