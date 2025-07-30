# 📚 Glosario Técnico

## A

- **Ansible:** Herramienta de automatización de TI de código abierto que permite la gestión de configuración, el despliegue de aplicaciones y la orquestación de tareas de forma agentless (sin agentes).
- **Ansible Vault:** Una característica de Ansible que permite encriptar archivos de variables o variables individuales para gestionar secretos como contraseñas y tokens de API.
- **API (Application Programming Interface):** Una interfaz que permite que dos o más aplicaciones de software se comuniquen entre sí. Usamos la API de NetBox para la automatización.
- **AWX:** La versión de código abierto y comunitaria de Ansible Tower. Proporciona una interfaz web para gestionar ejecuciones de Ansible.

## C

- **Checklist:** Una lista de tareas estandarizadas que se deben realizar durante un proceso específico (ej: onboarding de un nuevo nodo) para asegurar la consistencia y la calidad.
- **Chocolatey:** Un gestor de paquetes para Windows que permite instalar, actualizar y desinstalar software desde la línea de comandos de forma desatendida.
- **CIS Benchmarks (Center for Internet Security):** Guías de configuración de seguridad reconocidas a nivel mundial para fortalecer sistemas operativos, software y dispositivos de red.
- **CRUD (Create, Read, Update, Delete):** Las cuatro funciones básicas de la persistencia de datos. Una API RESTful permite realizar estas cuatro operaciones sobre un recurso.

## D

- **Dashboard:** En Grafana, un conjunto de una o más visualizaciones (paneles) organizadas en una pantalla para proporcionar una visión general o específica de un sistema.
- **Defensa en Profundidad (Defense in Depth):** Una estrategia de ciberseguridad que implementa múltiples capas de controles de seguridad, asumiendo que ninguna capa es impenetrable.
- **Docker:** Una plataforma de software que permite crear, desplegar y gestionar aplicaciones en contenedores aislados.
- **Docker Compose:** Una herramienta para definir y ejecutar aplicaciones Docker de múltiples contenedores. Utiliza archivos YAML para configurar los servicios de la aplicación.

## E

- **Exporter (Prometheus):** Un software que se ejecuta en un host a monitorear y que expone las métricas de ese host en un formato que Prometheus puede entender y recolectar.

## G

- **Grafana:** Una plataforma de código abierto para la visualización y el análisis de datos interactivos. Se conecta a fuentes de datos como Prometheus para crear dashboards y alertas.

## H

- **Hardening (Fortalecimiento):** El proceso de asegurar un sistema mediante la reducción de su superficie de ataque, deshabilitando servicios innecesarios y aplicando políticas de seguridad estrictas.

## I

- **Idempotencia:** Una propiedad de una operación que asegura que, si se aplica varias veces, el resultado será el mismo que si se aplicara una sola vez. Es un principio clave de Ansible.
- **Infraestructura como Código (IaC):** La práctica de gestionar y aprovisionar la infraestructura de TI a través de archivos de definición legibles por máquina (como los playbooks de Ansible), en lugar de la configuración manual.
- **Inventario (Ansible):** Un archivo o script que proporciona a Ansible la lista de nodos que debe gestionar.

## L

- **Loki:** Un sistema de agregación de logs de código abierto, inspirado en Prometheus, que está diseñado para ser altamente eficiente en costos y fácil de operar.

## M

- **MFA (Multi-Factor Authentication):** Un método de seguridad que requiere que el usuario proporcione dos o más factores de verificación para acceder a un recurso, como una contraseña y un código de un solo uso.

## N

- **NetBox:** Una herramienta de código abierto para el modelado y la documentación de la infraestructura de red. Actúa como una Fuente Única de Verdad (SSoT).

## P

- **Playbook (Ansible):** Un archivo YAML que contiene una lista ordenada de tareas a ejecutar por Ansible en un conjunto de hosts.
- **Prometheus:** Un sistema de monitoreo y alerta de código abierto que recolecta y almacena sus métricas como datos de series temporales.
- **PromQL:** El lenguaje de consulta funcional de Prometheus, utilizado para seleccionar y agregar datos de series temporales en tiempo real.

## S

- **SSH (Secure Shell):** Un protocolo de red criptográfico para operar servicios de red de forma segura sobre una red no segura.
- **SSoT (Single Source of Truth):** La práctica de estructurar la información de manera que cada dato maestro se edite en un solo lugar. En este proyecto, NetBox es la SSoT para la infraestructura.

## T

- **Tag (Etiqueta):** Un par clave-valor de metadatos que se puede asignar a los objetos en NetBox para una clasificación flexible.
- **Tenant (Inquilino):** Un objeto en NetBox que representa a un cliente o a una unidad de negocio, permitiendo la agrupación lógica de los recursos.
