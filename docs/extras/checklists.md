# 🧾 Checklists y Manuales de Operaciones Estándar (SOPs)

## 🎯 Objetivo
Centralizar los checklists operativos más importantes en un solo lugar. Estos checklists son la base para la consistencia, la calidad y la delegación de tareas. Deben ser tratados como documentos vivos y mejorados continuamente.

---

## ✅ SOP-01: Onboarding de un Nuevo Nodo

**Propósito:** Añadir un único nodo nuevo a la infraestructura gestionada.

**Responsables:** Operador Local, Gestor Remoto.

### Fase I: Aprovisionamiento Físico (Operador Local)

- [ ] **1. Conexión Física:** El MiniPC está conectado al switch y a la corriente.
- [ ] **2. Configuración de Red:** Se ha asignado una IP pública fija, máscara, gateway y DNS al sistema operativo.
- [ ] **3. Habilitación de Acceso:** El script `enable-ssh.ps1` ha sido ejecutado como Administrador.
- [ ] **4. Validación Local:**
    - [ ] `ipconfig` muestra la IP correcta.
    - [ ] `ping <gateway>` es exitoso.
    - [ ] `ping 8.8.8.8` es exitoso.
    - [ ] `Get-Service sshd` muestra el servicio como `Running` y `Automatic`.
- [ ] **5. Notificación:** Se ha comunicado al Gestor Remoto que el nodo está listo, indicando la IP asignada.

### Fase II: Integración Lógica (Gestor Remoto)

- [ ] **1. Validación Remota:**
    - [ ] `ping <nueva_ip>` es exitoso.
    - [ ] `nc -zv <nueva_ip> 22` confirma que el puerto SSH está abierto.
    - [ ] La conexión `ssh admin@<nueva_ip>` es exitosa.
- [ ] **2. Registro en NetBox:**
    - [ ] El nuevo objeto de dispositivo ha sido creado.
    - [ ] La dirección IP ha sido asignada al dispositivo.
    - [ ] El dispositivo ha sido asignado al **Tenant** correcto.
    - [ ] El dispositivo ha sido etiquetado con el **Tag** correcto.
- [ ] **3. Ejecución de Automatización:**
    - [ ] Se ha verificado que el nuevo host aparece en el inventario dinámico de Ansible.
    - [ ] Se han ejecutado los playbooks de configuración base (instalación, monitoreo, hardening) **limitando la ejecución** al nuevo host (`--limit <hostname>`).
- [ ] **4. Verificación de Monitoreo:**
    - [ ] El nuevo host aparece como `UP` en la página de Targets de Prometheus.
    - [ ] Las métricas del nuevo host son visibles en los dashboards de Grafana.

--- 

## ✅ SOP-02: Descomisionamiento de un Nodo

**Propósito:** Retirar un nodo de la infraestructura de forma segura y limpia.

**Responsable:** Gestor Remoto.

- [ ] **1. Exclusión de la Automatización:**
    - [ ] En NetBox, cambiar el **Status** del dispositivo de `Active` a `Decommissioning` o `Offline`. Esto lo excluirá automáticamente de la mayoría de los playbooks y del monitoreo.
- [ ] **2. Limpieza de Servicios (Opcional):**
    - [ ] Ejecutar un playbook de "offboarding" que desinstale el software de monitoreo, elimine las reglas de firewall y deshabilite el acceso SSH.
- [ ] **3. Eliminación de Registros:**
    - [ ] Eliminar el objeto de dispositivo de NetBox.
    - [ ] Liberar la dirección IP en el IPAM de NetBox, marcándola como `Deprecated` o `Available`.
- [ ] **4. Limpieza de Monitoreo y Backups:**
    - [ ] Verificar que el nodo ha desaparecido de Prometheus.
    - [ ] Excluir el nodo de cualquier configuración de backup específica si existiera.
- [ ] **5. Notificar al Operador Local:** Comunicar que el nodo ha sido retirado lógicamente y puede ser desconectado físicamente.

--- 

## ✅ SOP-03: Prueba de Restauración de Backups (Trimestral)

**Propósito:** Verificar la integridad y la viabilidad de las copias de seguridad del plano de control.

**Responsable:** Gestor Remoto.

- [ ] **1. Aprovisionar un Servidor de Restauración Temporal:**
    - [ ] Levantar un nuevo VPS limpio.
- [ ] **2. Descargar el Último Backup:**
    - [ ] Obtener el último archivo `.tar.gz` del backup desde el almacenamiento off-site (ej: Rclone).
- [ ] **3. Restaurar Servicios:**
    - [ ] Desplegar una nueva instancia de NetBox y Grafana con Docker.
    - [ ] Seguir el manual de restauración documentado para importar el dump de la base de datos de NetBox y los datos del volumen de Grafana.
- [ ] **4. Validar la Restauración:**
    - [ ] Iniciar sesión en la instancia restaurada de NetBox y verificar que los datos (dispositivos, IPs, etc.) son correctos.
    - [ ] Iniciar sesión en la instancia restaurada de Grafana y verificar que los dashboards y las fuentes de datos están presentes.
- [ ] **5. Medir y Documentar:**
    - [ ] Registrar el tiempo total que tomó el proceso de restauración (RTO - Recovery Time Objective).
    - [ ] Actualizar el manual de restauración con cualquier lección aprendida.
- [ ] **6. Destruir el Entorno Temporal:**
    - [ ] Eliminar el VPS de restauración.
