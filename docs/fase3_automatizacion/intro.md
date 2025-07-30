# ⚙️ Fase 3 – Introducción a la Automatización con Ansible

## 🎯 Objetivo
Comprender los conceptos fundamentales de **Ansible** y su rol como la principal herramienta de **gestión de configuración y orquestación** del proyecto. Esta fase marca el cambio de una gestión manual (nodo por nodo) a una **infraestructura como código (IaC)**, donde el estado deseado de tus servidores se define en archivos de texto.

---

## 🤔 ¿Qué es Ansible y por qué es la elección correcta?

Ansible es un motor de automatización que se conecta a tus nodos remotos (vía SSH) y ejecuta tareas. Piensa en él como un "director de orquesta" para tus servidores. En lugar de darle instrucciones a cada músico (servidor) individualmente, le entregas al director (Ansible) una partitura (un Playbook), y él se encarga de que toda la orquesta toque en armonía.

**Ventajas Clave para tu Proyecto:**

1.  **Sin Agentes (Agentless):** Esta es su mayor ventaja. No necesitas instalar y mantener un software "agente" en cada MiniPC. Ansible se comunica usando el protocolo SSH, que ya hemos configurado y asegurado. Esto significa menos consumo de recursos en los nodos y una barrera de entrada más baja.

2.  **Lenguaje Simple y Declarativo (YAML):** Las "partituras" de Ansible, llamadas Playbooks, se escriben en YAML. YAML es un lenguaje de serialización de datos legible por humanos. No describes los *pasos* para llegar a un estado, sino que *declaras* el estado final que deseas. Por ejemplo, en lugar de decir "ejecuta el comando para instalar Chrome", dices "el paquete Chrome debe estar presente".

3.  **Idempotencia Garantizada:** Esta es una propiedad fundamental. Un playbook idempotente puede ejecutarse múltiples veces, pero solo realizará cambios la primera vez. En las ejecuciones posteriores, detectará que el nodo ya está en el estado deseado y no hará nada. Esto hace que la automatización sea segura y predecible.

4.  **Extensible y Conectado:** Ansible tiene un ecosistema masivo de **módulos** (pequeños plugins para tareas específicas) y **colecciones** (paquetes de módulos, como el que usaremos para NetBox). Esto te permite gestionar todo, desde un servicio de Windows hasta un balanceador de carga en la nube.

--- 

## 🧱 La Arquitectura de Ansible

![Diagrama de Arquitectura de Ansible](https://docs.ansible.com/ansible/latest/_images/ansible_architecture.svg)

- **Nodo de Control (Control Node):** La máquina donde instalas y ejecutas Ansible. En tu caso, tu ordenador de gestión en Colombia. Es el único lugar donde necesitas instalar Ansible.

- **Nodos Gestionados (Managed Nodes):** Los servidores que Ansible configura. En tu caso, los 250 MiniPCs en EE.UU. Solo necesitan tener Python (en Linux) o PowerShell (en Windows) y un acceso SSH.

- **Inventario (Inventory):** El corazón de Ansible. Es una lista de los nodos gestionados. Puede ser un archivo de texto simple (`.ini` o `.yml`) o, como haremos nosotros, un **inventario dinámico** que consulta a NetBox en tiempo real para obtener la lista de hosts. Este es el puente entre tu fuente de verdad y tu motor de automatización.

- **Playbooks:** Archivos YAML que definen un conjunto ordenado de **Tareas (Tasks)** a ejecutar sobre un grupo de hosts del inventario. Cada tarea es una llamada a un módulo de Ansible.

--- 

##  workflow

1.  **Definir el Inventario:** Le dices a Ansible "estos son los servidores con los que vamos a trabajar" (conectándolo a NetBox).
2.  **Escribir un Playbook:** Creas un archivo YAML declarando el estado deseado (ej: "el usuario 'backup' debe existir", "el paquete 'nginx' debe estar instalado", "el servicio 'sshd' debe estar corriendo").
3.  **Ejecutar el Playbook:** Desde tu nodo de control, ejecutas `ansible-playbook mi_playbook.yml`.
4.  **Ansible Actúa:**
    -   Lee el inventario para obtener la lista de IPs.
    -   Se conecta a cada nodo vía SSH.
    -   Ejecuta cada tarea del playbook en orden.
    -   Verifica si el estado deseado ya se cumple (idempotencia).
    -   Realiza los cambios solo si es necesario.
5.  **Recibir el Informe:** Ansible te devuelve un resumen detallado de qué nodos fueron modificados, cuáles ya estaban en el estado correcto y si hubo algún error.

--- 

## 📌 Próximo Paso

Ahora que entiendes la teoría, el siguiente paso es instalar Ansible, configurar la conexión a tus nodos Windows y ejecutar tu primer playbook para ver la magia en acción.