# zbx-raid-all

Paquete `.deb` para **monitorizar el estado de pools de almacenamiento** (RAID / ZFS) en servidores Proxmox u otros sistemas Linux mediante `zabbix-agent2`.

El objetivo es exponer en Zabbix:

- Un **descubrimiento unificado de pools** (ZFS, controladoras HP con `ssacli`, controladoras LSI/DELL con `storcli`, etc.).
- Un ítem de **estado por pool**, independiente del hardware.

Ejemplo de ítems en Zabbix:

- `zbx.storage.pool.discovery`
- `zbx.storage.pool.state[{#POOLID}]`
- `zbx.storage.pool.state.code[{#POOLID}]` (ítem dependiente numérico para colores)

---

## Arquitectura

En cada servidor:

- Script coordinador:
  - `/usr/local/sbin/zbx_raid_collect.sh`
- Backends por tipo de almacenamiento:
  - `/usr/lib/zbx-raid/backend_zfs.sh`
  - `/usr/lib/zbx-raid/backend_ssacli.sh`
  - `/usr/lib/zbx-raid/backend_storcli.sh`
- Directorio de datos:
  - `/var/lib/zbx-raid/`
- Cron que ejecuta el recolector:
  - `/etc/cron.d/zbx-raid`
- UserParameters de Zabbix:
  - `/etc/zabbix/zabbix_agent2.d/zbx-raid.conf` :contentReference[oaicite:0]{index=0}  

El diseño sigue el patrón:

> **root** ejecuta los scripts por cron y escribe ficheros →  
> **Zabbix** solo lee esos ficheros (sin sudo, sin timeouts).

---

## Modelo de datos

### Directorio de datos

```text
/var/lib/zbx-raid/
  pools_discovery.json
  pool_zfs_rpool.state
  pool_ssacli_c4_ld1.state
  pool_storcli_c0_vd0.state
  ...

