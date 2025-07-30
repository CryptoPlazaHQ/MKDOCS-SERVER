# 📊 Fase 4 – Despliegue de Exporters en Nodos Windows

## 🎯 Objetivo
Instalar y configurar el **`windows_exporter`** en todos los MiniPCs de forma automatizada usando Ansible. Este componente es el puente que permite a Prometheus recolectar las métricas internas del sistema operativo Windows.

---

## 🤔 ¿Qué es un Exporter y cómo funciona?

Prometheus funciona con un modelo **pull** (jalar), no **push** (empujar). Esto significa que Prometheus decide cuándo recolectar las métricas; los nodos no se las envían activamente.

Un **exporter** es un pequeño servidor web que se ejecuta en el nodo a monitorear. Su única función es:
1.  Recolectar métricas del sistema operativo local (uso de CPU, memoria, I/O de disco, etc.).
2.  Traducirlas al formato de texto simple que Prometheus entiende.
3.  Exponerlas en una URL, típicamente en el endpoint `/metrics`.

Cuando Prometheus "raspa" (scrapes) un nodo, simplemente hace una petición HTTP a `http://<IP_DEL_NODO>:9182/metrics` y almacena los datos que recibe.

---

## ✅ Playbook para Desplegar `windows_exporter`

Usaremos Ansible para una instalación desatendida, idempotente y consistente en toda la flota de nodos.

**`install_windows_exporter.yml`**
```yaml
---
- name: Desplegar y Configurar Windows Exporter
  hosts: Nodo_Cliente
  gather_facts: no
  vars:
    exporter_version: "0.25.0"
    exporter_url: "https://github.com/prometheus-community/windows_exporter/releases/download/v{{ exporter_version }}/windows_exporter-{{ exporter_version }}-amd64.msi"
    exporter_dest_path: "C:\\Windows\\Temp\\windows_exporter.msi"

  tasks:
    - name: 1. Descargar el instalador de Windows Exporter
      ansible.windows.win_get_url:
        url: "{{ exporter_url }}"
        dest: "{{ exporter_dest_path }}"

    - name: 2. Instalar el paquete MSI
      ansible.windows.win_package:
        path: "{{ exporter_dest_path }}"
        state: present

    - name: 3. Asegurar que el servicio este iniciado y en modo automatico
      ansible.windows.win_service:
        name: windows_exporter
        state: started
        start_mode: auto

    - name: 4. Abrir el puerto del exporter en el Firewall de Windows
      community.windows.win_firewall_rule:
        name: "Windows Exporter Metrics (Prometheus)"
        localport: 9182
        action: allow
        direction: in
        protocol: tcp
        state: present
        enabled: yes
        description: "Permite a Prometheus recolectar métricas."
```

**Mejoras en este Playbook:**
- **Variables:** Se define la versión y la URL en la sección `vars` para facilitar futuras actualizaciones. Para actualizar el exporter en toda la flota, solo necesitas cambiar el número de versión en un lugar.
- **Nombres Descriptivos:** La regla de firewall tiene un nombre y una descripción claros, lo que facilita la auditoría.

---

## ✅ Configurar el Descubrimiento de Servicios en Prometheus

Ahora, le diremos a Prometheus que consulte a NetBox para encontrar sus objetivos (targets).

Edita tu archivo `/opt/monitoring/prometheus/prometheus.yml` en el servidor de monitoreo y añade un nuevo `job_name`:

```yaml
# ... (sección global y job 'prometheus' existentes)

  - job_name: 'windows_nodes'
    # Usar el plugin de descubrimiento de servicios de NetBox
    netbox_sd_configs:
      - server: 'http://<IP_DE_TU_SERVIDOR_NETBOX>'
        authorization:
          credentials: !secret netbox_token # Lee el token de un archivo secreto
        port: 9182
        refresh_interval: 1m # Consultar a NetBox cada minuto por si hay nuevos nodos

    # Filtrar y re-etiquetar los objetivos descubiertos
    relabel_configs:
      # Mantener solo los dispositivos que tienen el rol "Nodo Cliente"
      - source_labels: [__meta_netbox_device_role]
        regex: 'Nodo Cliente'
        action: keep

      # Mantener solo los dispositivos que están en estado "Activo"
      - source_labels: [__meta_netbox_status]
        regex: 'active'
        action: keep
```

**Mejoras en esta Configuración:**
- **Gestión de Secretos:** En lugar de poner el token en texto plano, `!secret netbox_token` le dice a Prometheus que lo lea de un archivo externo (ver abajo).
- **Filtrado Adicional:** Hemos añadido una segunda regla de `relabel_configs` para asegurarnos de que solo se monitorean los nodos cuyo estado en NetBox es "Activo".

**Crear el archivo de secretos de Prometheus:**
En tu servidor de monitoreo, crea el archivo `/opt/monitoring/prometheus/secrets.yml`:
```yaml
netbox_token: 'TU_TOKEN_DE_API_DE_NETBOX_AQUI'
```
Luego, en tu `docker-compose.yml`, necesitas mapear este archivo al contenedor de Prometheus. Añade esta línea bajo `prometheus.volumes`:
`- ./prometheus/secrets.yml:/etc/prometheus/secrets.yml`

Finalmente, **reinicia Prometheus** para que todos los cambios surtan efecto:
`cd /opt/monitoring && sudo docker-compose restart prometheus`

---

## ✅ Checklist de Validación

- [ ] El playbook de Ansible se ejecuta sin errores en todos los nodos.
- [ ] En un nodo cualquiera, puedes abrir un navegador y ver las métricas en `http://localhost:9182/metrics`.
- [ ] En la interfaz de Prometheus (`:9090`), la sección **Status > Targets** muestra todos tus nodos `windows_nodes` con el estado `UP`.
- [ ] Si cambias un nodo a estado `offline` en NetBox, después de un minuto desaparecerá de la lista de objetivos de Prometheus.