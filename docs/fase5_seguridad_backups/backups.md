# 🔒 Fase 5 – Estrategia de Copias de Seguridad y Recuperación

## 🎯 Objetivo
Diseñar e implementar una estrategia de copias de seguridad **automática, fiable y verificable** para los datos críticos del plano de control. El objetivo no es solo "hacer backups", sino tener la certeza de que puedes **restaurar el servicio** rápidamente en caso de un desastre.

---

## 🤔 ¿De qué hacemos backup y por qué?

La filosofía de la Infraestructura como Código nos dice que no necesitamos hacer un backup completo de cada MiniPC. Su configuración está definida en nuestros playbooks de Ansible. Si un nodo falla, lo reemplazamos y lo reconstruimos desde el código.

Los datos verdaderamente **irremplazables y de estado (stateful)** residen en tu servidor de gestión:

1.  **Base de Datos de NetBox (PostgreSQL):** **CRÍTICO.** Es el cerebro de tu operación. Contiene toda la información sobre tus sitios, clientes, dispositivos, IPs, etc.
2.  **Datos de Grafana:** Contiene todos los dashboards que has creado, las fuentes de datos y las configuraciones de alertas. Perderlos supondría horas de trabajo de reconstrucción.
3.  **Configuración de Prometheus:** Aunque las métricas en sí pueden ser efímeras, la configuración de los `jobs` y las reglas de alerta son importantes.
4.  **Tus Propios Archivos de Infraestructura:** El repositorio completo con tus playbooks de Ansible, y sobre todo, tu **Ansible Vault** encriptado.

--- 

## ✅ La Regla 3-2-1: El Estándar de Oro de los Backups

- **3 Copias:** Mantén tres copias de tus datos importantes (la original + 2 backups).
- **2 Medios Diferentes:** Almacena las copias en al menos dos tipos de almacenamiento distintos (ej: disco local del servidor + almacenamiento en la nube).
- **1 Copia Off-site:** Asegúrate de que al menos una de las copias esté en una ubicación geográfica diferente.

--- 

##  automating-backups-with-a-shell-script

Este script, ejecutado en tu servidor de gestión, orquestará el proceso de backup.

**`/opt/scripts/backup_infra.sh`**
```bash
#!/bin/bash

set -e # El script se detendrá inmediatamente si un comando falla

# --- Variables de Configuración ---
BACKUP_DIR="/mnt/backups/$(date +%Y-%m-%d)" # Crea una carpeta por día
NETBOX_DOCKER_DIR="/opt/netbox-docker"      # Ruta a la instalación de NetBox
GRAFANA_VOLUME_NAME="monitoring_grafana_data" # Nombre del volumen de Docker de Grafana

# --- Rclone Remote (Opcional pero recomendado) ---
# Configura rclone con "rclone config" previamente
RCLONE_REMOTE="b2:mi-bucket-de-backups"

# --- Inicio del Proceso ---
echo "[INFO] Iniciando proceso de backup para $(date)"
mkdir -p $BACKUP_DIR

# --- 1. Backup de la Base de Datos de NetBox ---
echo "[INFO] Realizando dump de la base de datos de NetBox..."
(cd $NETBOX_DOCKER_DIR && sudo docker-compose exec -T postgres sh -c 'pg_dump -U netbox netbox' > $BACKUP_DIR/netbox_db.sql)
echo "[SUCCESS] Dump de la base de datos completado."

# --- 2. Backup de los Datos de Grafana ---
echo "[INFO] Copiando datos del volumen de Grafana..."
# Montamos temporalmente el volumen en un contenedor para copiar los datos de forma segura
sudo docker run --rm -v ${GRAFANA_VOLUME_NAME}:/data -v ${BACKUP_DIR}:/backup alpine tar -czf /backup/grafana_data.tar.gz -C /data .
echo "[SUCCESS] Datos de Grafana comprimidos."

# --- 3. Backup de los Archivos de Configuración ---
# (Prometheus, Ansible, etc. asumimos que están en un repo Git, por lo que el backup es el 'git push')

# --- 4. Comprimir el Backup Diario ---
FINAL_ARCHIVE="/mnt/backups/infra_backup_$(date +%Y-%m-%d).tar.gz"
echo "[INFO] Creando archivo final: ${FINAL_ARCHIVE}"
tar -czf $FINAL_ARCHIVE -C /mnt/backups/ $(date +%Y-%m-%d)

# --- 5. Limpieza del Directorio Diario ---
rm -rf $BACKUP_DIR

# --- 6. Sincronización Off-site con Rclone ---
if command -v rclone &> /dev/null; then
    echo "[INFO] Sincronizando el backup con el almacenamiento remoto (${RCLONE_REMOTE})..."
    rclone copy $FINAL_ARCHIVE ${RCLONE_REMOTE}
    echo "[SUCCESS] Sincronización completada."
else
    echo "[WARNING] rclone no está instalado. Omitiendo backup off-site."
fi

echo "[INFO] Proceso de backup finalizado."
```

### Automatización con Cron

Ejecuta `sudo crontab -e` y añade una línea para ejecutar el script diariamente.

```cron
# Ejecutar el script de backup todos los días a las 3:05 AM
5 3 * * * /opt/scripts/backup_infra.sh >> /var/log/backup_infra.log 2>&1
```

--- 

##  restorability-is-more-important-than-backups

Un backup que no ha sido probado es solo una suposición. Debes **probar periódicamente tu capacidad de restauración**.

- **Plan de Prueba Trimestral:** Cada tres meses, despliega un nuevo VPS temporal y sigue un manual de restauración documentado para reconstruir tu NetBox y Grafana a partir de los backups. Mide el tiempo que tardas (esto es tu **Tiempo de Recuperación Objetivo - RTO**).
- **Documenta el Proceso:** Crea un `RESTAURACION.md` en tu documentación con los pasos exactos para la recuperación. Cuando ocurra un desastre real, no querrás estar adivinando los comandos.