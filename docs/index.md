# Red de MiniPCs Inteligentes – Infraestructura Modular Descentralizada

## Visión General del Proyecto

Bienvenido a la documentación de la red de MiniPCs inteligentes. Este proyecto está diseñado para construir y gestionar una infraestructura descentralizada de nodos (MiniPCs con Windows 10/11), controlada de forma remota y segura.

El objetivo es ofrecer una plataforma robusta y escalable para servicios distribuidos, manteniendo un control centralizado sobre la configuración, el monitoreo y la seguridad de cada nodo.

## Arquitectura

La arquitectura se basa en un enfoque modular y por capas:

- **Capa de Acceso:** Conexión segura a cada nodo mediante SSH sobre IPs públicas fijas, con firewalls configurados para restringir el acceso.
- **Capa de Inventario:** NetBox actúa como la única fuente de verdad (SSoT) para todos los recursos de la red (IPs, dispositivos, sitios, clientes).
- **Capa de Automatización:** Ansible se utiliza para la gestión de la configuración, instalación de software y ejecución de tareas en masa, consultando el inventario de NetBox en tiempo real.
- **Capa de Monitoreo:** Un stack de Prometheus y Grafana recopila métricas de cada nodo y las presenta en dashboards visuales para un control proactivo de la salud y el rendimiento de la red.

## Guía de Inicio

Para empezar a explorar la documentación, utilice la barra de navegación para moverse a través de las distintas fases del proyecto. Cada sección contiene guías paso a paso, explicaciones conceptuales y scripts listos para usar.
