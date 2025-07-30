# 🧬 Fase 6 – Manual de Operaciones: Procesos de Replicación

## 🎯 Objetivo
Documentar los **Manuales de Operaciones Estándar (SOPs)** para los dos escenarios de crecimiento más comunes: añadir un nuevo nodo a un cliente existente y dar de alta a un cliente completamente nuevo. El objetivo es que cualquier miembro del equipo (o incluso tú mismo en el futuro) pueda seguir estos pasos de forma mecánica y sin errores.

---

## Escenario 1: Añadir un Nuevo Nodo a un Cliente Existente

**Situación:** El `Cliente A` está satisfecho y quiere añadir un 11º MiniPC a su red.

### 📋 Checklist del Proceso (SOP-ADD-NODE-01)

**Fase I: Aprovisionamiento Físico (Responsable: Operador Local)**
1.  ☐ **Conectar Hardware:** Conectar el nuevo MiniPC al switch PoE y a la corriente.
2.  ☐ **Asignar IP Fija:** Configurar el sistema operativo con la siguiente IP pública disponible del bloque del cliente (ej: `50.190.105.91`).
3.  ☐ **Habilitar Acceso:** Ejecutar el script `enable-ssh.ps1` para activar el servidor SSH.
4.  ☐ **Validación Local:** Realizar el checklist de validación local de la Fase 0 (ipconfig, ping al gateway, ping a 8.8.8.8).
5.  ☐ **Notificar al Gestor:** Comunicar al gestor remoto que el nodo está físicamente listo, proporcionando la IP asignada.

**Fase II: Integración Lógica (Responsable: Gestor Remoto)**
1.  ☐ **Validar Acceso Remoto:**
    -   Verificar que puedes hacer `ping` a la nueva IP desde Colombia.
    -   Verificar que el puerto 22 está abierto: `nc -zv 50.190.105.91 22`.
    -   Realizar una conexión SSH exitosa: `ssh admin@50.190.105.91`.
2.  ☐ **Registrar en NetBox:**
    -   Añadir el nuevo dispositivo `cliente-a-nodo-11`.
    -   Asignarle la IP `50.190.105.91`.
    -   **Crucial:** Asignarle el **Tenant** `Cliente A` y el **Tag** `cliente-a` para que herede las políticas y la automatización del cliente.
3.  ☐ **Ejecutar Configuración Base:**
    -   Verificar que el nuevo nodo aparece en el inventario dinámico: `ansible-inventory -i produccion.netbox.yml --graph | grep cliente-a-nodo-11`.
    -   Ejecutar el conjunto de playbooks de configuración inicial, **limitando la ejecución** al nuevo host para máxima seguridad y eficiencia:
        ```bash
        ansible-playbook -i produccion.netbox.yml install_base.yml --limit cliente-a-nodo-11
        ansible-playbook -i produccion.netbox.yml install_monitoring.yml --limit cliente-a-nodo-11
        ansible-playbook -i produccion.netbox.yml harden_security.yml --limit cliente-a-nodo-11
        ```
        *(Nota: Es una buena práctica agrupar playbooks en uno maestro, como `install_base.yml`)*
4.  ☐ **Verificación Final:**
    -   Comprobar en Prometheus (Targets) que el nuevo nodo está `UP`.
    -   Verificar en Grafana que las métricas del `cliente-a-nodo-11` son visibles.

--- 

## Escenario 2: Onboarding de un Nuevo Cliente

**Situación:** La `Empresa B` te contrata para gestionar un nuevo clúster de 10 MiniPCs.

### 📋 Checklist del Proceso (SOP-NEW-CLIENT-01)

**Fase I: Preparación Lógica (Responsable: Gestor Remoto)**
1.  ☐ **Crear Tenant en NetBox:** Crear el Tenant `Cliente B`.
2.  ☐ **Crear Tag en NetBox:** Crear el Tag `cliente-b`.
3.  ☐ **Asignar Recursos de Red:** Si el cliente tiene su propio bloque de IPs, crear el nuevo `Prefix` en el IPAM de NetBox.
4.  ☐ **Preparar Variables de Ansible:** Si el cliente tiene configuraciones especiales, crear un archivo de variables para su grupo (ej: `group_vars/tag_cliente_b/vars.yml`).

**Fase II: Aprovisionamiento Físico en Lote (Responsable: Operador Local)**
1.  ☐ **Ejecutar SOP-ADD-NODE-01** para cada uno de los 10 nuevos MiniPCs, usando el rango de IPs asignado al Cliente B.

**Fase III: Integración y Configuración en Lote (Responsable: Gestor Remoto)**
1.  ☐ **Validar Acceso Remoto** a los 10 nuevos nodos.
2.  ☐ **Registrar en Lote en NetBox:** Ejecutar el script `registrar_nodos.py`, adaptado con las variables del Cliente B (nombre, tag, rango de IPs).
3.  ☐ **Ejecutar Configuración Base en Lote:**
    -   Verificar que el nuevo grupo `tag_cliente_b` existe en el inventario.
    -   Ejecutar los playbooks de configuración inicial, esta vez **limitando la ejecución** al nuevo grupo de tags:
        ```bash
        ansible-playbook -i produccion.netbox.yml install_base.yml -l tag_cliente_b
        ```
4.  ☐ **Verificación Final:**
    -   Verificar en Prometheus y Grafana que los 10 nuevos nodos del Cliente B están siendo monitoreados.

--- 

## 🧠 Clave del Éxito: El Checklist

Tener estos procesos documentados como checklists es la diferencia entre una operación artesanal y una operación de nivel empresarial. Asegura que no se omita ningún paso, que la calidad sea consistente y te permite delegar estas tareas en el futuro con confianza.