# 🔒 Fase 5 – Hardening Automatizado de Nodos Windows

## 🎯 Objetivo
Ir más allá de la configuración básica de seguridad y aplicar un conjunto de **configuraciones de fortalecimiento (hardening)** de forma automática y consistente a todos los nodos. El objetivo es reducir la superficie de ataque del sistema operativo Windows, deshabilitando características innecesarias y aplicando políticas de seguridad estrictas.

---

## 🤔 ¿Qué es el Hardening?

El hardening es el proceso de asegurar un sistema mediante la reducción de sus vulnerabilidades. Un sistema operativo, por defecto, viene con muchas características habilitadas para maximizar la compatibilidad y la facilidad de uso, no la seguridad. El hardening revierte esto, aplicando el principio de "lo que no es explícitamente necesario, debe estar deshabilitado".

Para una operación profesional, no puedes hacer esto manualmente. Necesitas un playbook de Ansible que verifique y aplique estas configuraciones de forma idempotente.

--- 

## 🛡️ Playbook de Hardening Básico

Este playbook es un ejemplo que aplica algunas de las recomendaciones de seguridad más comunes. Un playbook de hardening en un entorno de producción real podría tener cientos de tareas.

**`harden_windows.yml`**
```yaml
---
- name: Aplicar Linea Base de Seguridad a Nodos Windows
  hosts: Nodo_Cliente
  gather_facts: no

  tasks:
    - name: Deshabilitar el servicio de Escritorio Remoto (RDP)
      ansible.windows.win_service:
        name: TermService # Nombre del servicio de RDP
        state: stopped
        start_mode: disabled
      when: "'servidor' not in group_names" # Ejemplo: no aplicar a servidores de gestión

    - name: Deshabilitar NetBIOS sobre TCP/IP
      ansible.windows.win_regedit:
        path: HKLM:\SYSTEM\CurrentControlSet\Services\NetBT\Parameters\Interfaces\Tcpip_{{ item.Name }}
        name: NetbiosOptions
        data: 2
        type: dword
      loop: "{{ ansible_interfaces | selectattr('ipv4.address', 'defined') | list }}"
      loop_control:
        label: "{{ item.name }}"

    - name: Configurar politica de contrasenas
      community.windows.win_security_policy:
        section: System Access
        key: MinimumPasswordLength
        value: 14

    - name: Habilitar el Firewall de Windows para todos los perfiles
      ansible.windows.win_firewall:
        profiles: 
          - Domain
          - Private
          - Public
        state: on
        block_inbound_rules: yes

    - name: Deshabilitar cuentas de invitado
      ansible.windows.win_user:
        name: Guest
        state: present
        account_disabled: yes
```

**Análisis de las Tareas de Hardening:**
1.  **Deshabilitar RDP:** El Escritorio Remoto es un vector de ataque muy común. A menos que sea estrictamente necesario, debe estar deshabilitado. La condición `when` es un ejemplo de cómo podrías evitar aplicar esta regla a ciertos grupos de hosts.
2.  **Deshabilitar NetBIOS:** Un protocolo antiguo que puede ser usado para obtener información de la red. Se deshabilita modificando una clave del registro de Windows para cada interfaz de red.
3.  **Política de Contraseñas:** Se utiliza el módulo `win_security_policy` para forzar políticas de seguridad locales, como la longitud mínima de la contraseña.
4.  **Habilitar Firewall:** Se asegura de que el Firewall de Windows esté activado para todos los perfiles de red y que, por defecto, bloquee las conexiones entrantes que no tengan una regla explícita de "Allow".
5.  **Deshabilitar Cuenta de Invitado:** La cuenta `Guest` es un riesgo de seguridad y debe estar siempre deshabilitada.

--- 

## 🚀 Implementación y Mejora Continua

- **Ejecución Periódica:** Este playbook no es para ejecutarlo una sola vez. Deberías programarlo (usando AWX o cron) para que se ejecute periódicamente (ej: semanalmente) en toda tu flota. Esto corrige cualquier "desviación de configuración" que haya podido ocurrir.
- **Basado en Estándares:** Para un hardening de nivel profesional, basa tus playbooks en estándares reconocidos como los **CIS Benchmarks (Center for Internet Security)** o las guías **STIG (Security Technical Implementation Guides)**. Existen roles de Ansible pre-hechos (aunque a menudo comerciales) que automatizan la aplicación de estos estándares.
- **Auditoría:** El resultado de la ejecución de este playbook es, en sí mismo, un informe de auditoría. Te muestra qué nodos estaban cumpliendo la política y cuáles necesitaron ser corregidos.
