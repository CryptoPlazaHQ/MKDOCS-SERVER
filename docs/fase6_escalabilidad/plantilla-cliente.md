# 🧬 Fase 6 – Plantillas de Cliente para una Escalabilidad Ordenada

## 🎯 Objetivo
Definir y estandarizar el concepto de una **"Plantilla de Cliente"**. Esta plantilla es un conjunto de objetos y configuraciones lógicas en tus sistemas de gestión que te permitirán incorporar (onboard) y gestionar nuevos clientes de una manera **rápida, consistente y aislada**.

---

## 🤔 ¿Por qué necesitas una Plantilla?

A medida que creces, la complejidad aumenta. Gestionar 2 clientes con 10 nodos cada uno es muy diferente a gestionar 25 clientes. Sin un modelo estándar, te arriesgas a:
- **Configuraciones Inconsistentes:** Cada cliente se configura de una manera ligeramente diferente, lo que hace que la automatización sea una pesadilla.
- **Falta de Aislamiento:** Una operación destinada al Cliente A podría afectar accidentalmente al Cliente B.
- **Onboarding Lento:** Incorporar un nuevo cliente se convierte en un largo proyecto de descubrimiento en lugar de un proceso predecible.

Una plantilla resuelve esto al convertir el onboarding de clientes en un proceso de "rellenar los huecos".

--- 

## 🧱 Componentes de la Plantilla de Cliente en NetBox

NetBox está diseñado para la multi-tenencia. Usaremos sus características nativas para modelar a nuestros clientes.

### 1. Tenants (Inquilinos)

El objeto **Tenant** es la representación directa de un cliente. Proporciona el nivel más alto de agrupación lógica.

- **Creación:**
    -   **Ruta:** `Tenancy > Tenants > + Add`
    -   **Name:** `Cliente A` (El nombre comercial del cliente)
    -   **Group:** `Clientes Comerciales` (Puedes crear grupos para organizar tenants)

### 2. Tags (Etiquetas)

Los **Tags** son etiquetas de metadatos flexibles y de colores que son cruciales para la automatización. A diferencia de los Tenants, puedes asignar múltiples tags a un objeto.

- **Creación:**
    -   **Ruta:** `Organization > Tags > + Add`
    -   **Name:** `cliente-a` (**Mejor práctica:** usa un nombre en minúsculas, sin espacios, apto para URLs y nombres de grupo en Ansible. Esto se llama "slug").
    -   **Content Types:** Asigna el tag para que se pueda aplicar a `dcim > device`.

### 3. Asignación a Dispositivos

Cuando registras un nuevo dispositivo, ahora debes asignarle tanto el **Tenant** como el **Tag** correspondiente. Esto enriquece el modelo de datos y permite un filtrado y una automatización muy precisos.

--- 

## 🤖 Uso de la Plantilla en la Automatización (Ansible)

Aquí es donde el modelado de NetBox da sus frutos. Tu archivo de inventario dinámico (`produccion.netbox.yml`) ya está configurado con `group_by: - tags` y `group_by: - tenants`. Esto significa que Ansible crea automáticamente grupos basados en estos objetos.

### Ejemplo: Ejecutar un Playbook solo para el Cliente A

Imagina que el Cliente A solicita que se instale `Microsoft Edge` en todos sus nodos.

**`cliente_a_edge.yml`**
```yaml
---
- name: Instalar Microsoft Edge para el Cliente A
  # Usamos el grupo creado a partir del Tag. Es más limpio y específico.
  hosts: tag_cliente_a 
  gather_facts: no

  tasks:
    - name: Instalar el paquete de Microsoft Edge
      ansible.windows.win_chocolatey:
        name: microsoft-edge
        state: present
```

Al ejecutar este playbook, tienes la **garantía** de que solo se aplicará a los dispositivos que tengan el tag `cliente-a` en NetBox. Has logrado un **aislamiento total de la operación**.

--- 

## 📋 Proceso de Onboarding para un Nuevo Cliente ("Cliente B")

Este es ahora tu Manual de Operaciones Estándar para el crecimiento:

1.  **[NetBox] Crear Objetos Lógicos:**
    -   Crear el Tenant `Cliente B`.
    -   Crear el Tag `cliente-b`.

2.  **[Operador/Tú] Aprovisionamiento Físico y de Red:**
    -   El operador local instala los 10 MiniPCs.
    -   Se les asignan las IPs correspondientes.
    -   Se habilita el acceso SSH en cada uno.

3.  **[Tú - Automatización] Registro y Configuración Base:**
    -   Ejecutas tu script `registrar_nodos.py`, modificado para que ahora asigne el Tenant `Cliente B` y el Tag `cliente-b` a los nuevos dispositivos.
    -   Verificas que el nuevo grupo `tag_cliente_b` aparece en el inventario de Ansible (`ansible-inventory -i produccion.netbox.yml --graph`).
    -   Ejecutas tu conjunto de playbooks de configuración inicial (Chocolatey, exporter, hardening) **limitando la ejecución** al nuevo grupo:
        ```bash
        ansible-playbook -i produccion.netbox.yml install_base_software.yml -l tag_cliente_b
        ```
        - El flag `-l` (o `--limit`) es tu bisturí para aplicar operaciones a un subconjunto específico de tu infraestructura.

En menos de una hora, puedes tener un cliente completamente nuevo, con 10 nodos, registrado, configurado de forma consistente y monitoreado, todo gracias a haber definido una plantilla clara.