# 🧬 Fase 6 – Roadmap Estratégico de la Infraestructura

## 🎯 Objetivo
Definir una hoja de ruta clara y estratégica para la evolución de la plataforma. Este documento no es una lista de tareas, sino una guía de visión a futuro que te ayudará a tomar decisiones tecnológicas, planificar inversiones de tiempo y mejorar continuamente el valor de tu servicio.

---

## 🗺️ La Hoja de Ruta por Horizontes de Tiempo

### 🚀 Horizonte 1: Mejoras Fundamentales (Corto Plazo: Próximos 3-6 meses)

*   **Centralización de Logs con Loki:**
    -   **Problema:** Los logs de eventos y aplicaciones residen en cada nodo, lo que hace que la investigación de incidentes sea lenta y difícil.
    -   **Solución:** Desplegar **Grafana Loki**, el componente de logging del stack de Grafana. Es ligero, se integra nativamente con Prometheus y te permite buscar y analizar logs de toda tu flota desde la misma interfaz de Grafana.
    -   **Beneficio:** Capacidad de correlacionar métricas (ej: un pico de CPU) con los logs exactos que ocurrieron en ese mismo instante, mejorando drásticamente la velocidad de diagnóstico (MTTR).

*   **Interfaz Gráfica para Ansible con AWX/Tower:**
    -   **Problema:** La ejecución de playbooks desde la línea de comandos es potente pero carece de control de acceso, auditoría y visibilidad para un equipo.
    -   **Solución:** Instalar **AWX** (la versión de código abierto de Ansible Tower). AWX proporciona una interfaz web para gestionar tu automatización con dashboards, control de acceso basado en roles (RBAC), programación de trabajos y un historial completo.
    -   **Beneficio:** Puedes delegar de forma segura la ejecución de ciertos playbooks a otros usuarios sin darles acceso SSH ni a los secretos del vault.

*   **Hardening de Seguridad Automatizado:**
    -   **Problema:** La seguridad es un proceso continuo, no una configuración única.
    -   **Solución:** Desarrollar un playbook de Ansible dedicado a aplicar una línea base de seguridad (basada en estándares como **CIS Benchmarks**). Este playbook verificaría y corregiría periódicamente cientos de configuraciones de seguridad en el sistema operativo.
    -   **Beneficio:** Aumenta exponencialmente la postura de seguridad de los nodos y te permite generar informes de cumplimiento.

### 📈 Horizonte 2: Expansión de Capacidades (Mediano Plazo: Próximos 6-12 meses)

*   **Plano de Control Seguro con WireGuard:**
    -   **Problema:** La dependencia de IPs públicas y reglas de firewall por IP puede ser frágil, especialmente con clientes detrás de redes restrictivas.
    -   **Solución:** Implementar una **Overlay VPN** con **WireGuard** para crear un plano de control seguro y privado. Los nodos se conectarían a esta red privada para la gestión, independientemente de su IP pública.
    -   **Beneficio:** Elimina la necesidad de exponer el puerto SSH a Internet, simplifica la conectividad y aumenta la seguridad del tráfico de gestión.

*   **Gestión Avanzada de Secretos con HashiCorp Vault:**
    -   **Problema:** Ansible Vault es bueno, pero no ofrece rotación de secretos, leases dinámicos ni una API centralizada.
    -   **Solución:** Migrar a una solución de gestión de secretos dedicada como **HashiCorp Vault**.
    -   **Beneficio:** Seguridad de nivel empresarial, con la capacidad de generar credenciales de corta duración y auditar centralmente cada acceso a un secreto.

### 🔭 Horizonte 3: Madurez y Optimización del Negocio (Largo Plazo: 12+ meses)

*   **Infraestructura como Código (IaC) para el Plano de Control:**
    -   **Problema:** El propio servidor de gestión (con NetBox, Grafana, etc.) todavía se configura manualmente.
    -   **Solución:** Usar **Terraform** para definir toda la infraestructura del plano de control (el VPS, las redes, los firewalls de nube) como código. Esto te permitiría destruir y reconstruir todo tu entorno de gestión de forma 100% automática.
    -   **Beneficio:** Resiliencia definitiva y capacidad de recuperación ante desastres para tu propio negocio.

*   **Portal de Cliente y Autoservicio:**
    -   **Problema:** Todas las solicitudes de los clientes (ver estado, pedir un reinicio) requieren tu intervención manual.
    -   **Solución:** Desarrollar un portal web simple donde los clientes puedan iniciar sesión y ver los dashboards de Grafana de *sus propios* nodos (usando las capacidades multi-tenant de Grafana) y quizás disparar acciones pre-aprobadas (como un reinicio) que ejecuten un playbook de Ansible a través de la API de AWX.
    -   **Beneficio:** Aumenta enormemente el valor del servicio para el cliente y reduce tu carga operativa.