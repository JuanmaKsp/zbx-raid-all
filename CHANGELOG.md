```markdown
# Changelog

Todas las versiones de `zbx-raid-all`.  
Formato: `Versión — Fecha — Resumen`.

---

## 1.0-1 — 2025-12-XX

### Añadido

- Versión inicial del paquete `zbx-raid-all`.
- Arquitectura de recolección unificada de pools:
  - Script coordinador `/usr/local/sbin/zbx_raid_collect.sh`.
  - Directorio de datos `/var/lib/zbx-raid/`.
  - Cron `/etc/cron.d/zbx-raid`.
  - UserParameters Zabbix en `/etc/zabbix/zabbix_agent2.d/zbx-raid.conf`.
- Modelo de datos:
  - Fichero `pools_discovery.json` con LLD unificada de pools.
  - Ficheros `pool_<POOLID>.state` con el estado de cada pool.
- Backend ZFS:
  - Script `/usr/lib/zbx-raid/backend_zfs.sh`.
  - Detección automática de zpools mediante `zpool list`.
  - Normalización de estados: `ONLINE` → `OK`, `DEGRADED` → `DEGRADED`,
    `FAULTED/OFFLINE` → `FAILED`.
- Integración básica con Zabbix:
  - `zbx.storage.pool.discovery` para LLD.
  - `zbx.storage.pool.state[{#POOLID}]` para estado de cada pool.

### Pendiente / futuro

- Backend ssacli:
  - Script `/usr/lib/zbx-raid/backend_ssacli.sh` para controladoras HP.
  - Parseo de configuración de `ssacli ctrl all show config ...`.
- Backend storcli:
  - Script `/usr/lib/zbx-raid/backend_storcli.sh` para controladoras LSI/DELL.
  - Parseo de `storcli64 /cALL /vall show all`.
- Template Zabbix oficial en este repositorio con:
  - Protótipo de item `Pool {#POOLID} status`.
  - Protótipos de trigger para estados `DEGRADED` y `FAILED`.
