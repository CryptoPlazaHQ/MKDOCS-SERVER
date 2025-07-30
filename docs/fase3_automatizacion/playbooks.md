# ⚙️ Fase 3 – Primeros Pasos con Ansible: Inventario y Playbooks

## 🎯 Objetivo
Instalar Ansible, configurar la conexión a los nodos Windows y ejecutar un primer playbook de diagnóstico. En esta sección, usaremos un **inventario estático** (un archivo de texto simple) como primer paso para verificar que la comunicación entre el nodo de control y los nodos gestionados funciona correctamente antes de pasar a un inventario dinámico más complejo.

---

## ✅ Paso 1: Instalar Ansible en el Nodo de Control

Ansible se instala únicamente en tu máquina de gestión (tu ordenador en Colombia). La forma más común es a través de `pip`, el gestor de paquetes de Python.

```bash
# Es una buena práctica crear un entorno virtual de Python primero
python3 -m venv ansible_env
source ansible_env/bin/activate

# Instalar Ansible dentro del entorno virtual
pip install ansible
```

- **Entorno Virtual:** Crear un entorno virtual (`venv`) aísla las dependencias de Ansible de otros proyectos de Python que puedas tener, evitando conflictos.
- **Verificación:** Una vez instalado, ejecuta `ansible --version` para confirmar que la instalación fue exitosa.

--- 

## ✅ Paso 2: Configurar la Conexión a Nodos Windows

Ansible fue creado originalmente para gestionar servidores Linux, por lo que la comunicación con Windows requiere una configuración específica. Afortunadamente, el soporte para Windows es hoy muy robusto.

El método moderno y recomendado es usar la conexión **SSH** que ya configuramos en la Fase 1. Necesitamos decirle a Ansible cómo debe comunicarse con nuestros nodos Windows.

--- 

## ✅ Paso 3: Crear un Inventario Estático Inicial

El inventario es el listado de servidores que Ansible gestionará. Empezaremos con un archivo simple en formato INI.

Crea un archivo llamado `inventory.ini`:

```ini
# =====================================
# Inventario para la Red de MiniPCs
# =====================================

# Definimos un grupo llamado [windows_nodes]
[windows_nodes]
# Usamos los alias que configuramos en nuestro archivo ~/.ssh/config
# Esto es una buena práctica porque centraliza la información de conexión (IP, usuario, clave)
# en un solo lugar.
nodo-a-01
nodo-a-02
nodo-a-03
nodo-a-04
nodo-a-05
nodo-a-06
nodo-a-07
nodo-a-08
nodo-a-09
nodo-a-10

# Definimos variables específicas para el grupo [windows_nodes]
[windows_nodes:vars]
# Le dice a Ansible que, una vez conectado, debe usar una shell de PowerShell.
# ¡Esto es fundamental para que los comandos se ejecuten correctamente en Windows!
ansible_shell_type = powershell
```

**Nota Importante:** No necesitamos especificar `ansible_user` o `ansible_ssh_private_key_file` aquí porque ya lo hemos definido en nuestro archivo `~/.ssh/config`. Ansible es lo suficientemente inteligente como para usar esa configuración.

--- 

## ✅ Paso 4: Tu Primer Playbook de Diagnóstico

Un playbook es una lista de tareas. Crearemos uno muy simple para verificar que podemos conectarnos a todos los nodos y obtener información de ellos.

Crea un archivo llamado `check_reachability.yml`:

```yaml
---
- name: Test de Conectividad y Recolección de Datos para Nodos Windows
  hosts: windows_nodes
  gather_facts: no # Desactivamos la recolección automática por ahora

  tasks:
    - name: 1. Probar la conectividad basica (ping)
      ansible.windows.win_ping:
      register: ping_result

    - name: 2. Mostrar el resultado del ping
      debug:
        var: ping_result

    - name: 3. Recolectar "facts" del sistema (informacion de hardware, SO, etc.)
      ansible.windows.setup:
      register: windows_facts

    - name: 4. Mostrar el nombre del SO y la memoria RAM de cada nodo
      debug:
        msg: "El sistema operativo es {{ windows_facts.ansible_facts.ansible_os_distribution }} y tiene {{ windows_facts.ansible_facts.ansible_memory_mb.real.total }} MB de RAM."
```

**Análisis de este Playbook:**
- `hosts: windows_nodes`: Apunta al grupo que definimos en nuestro inventario.
- `win_ping`: Es el módulo de Ansible para Windows que no solo hace un `ping` de red, sino que verifica que la conexión de Ansible y la ejecución de PowerShell funcionan. Es un "hola mundo" para la conectividad de Ansible.
- `register`: Guarda toda la salida de una tarea en una variable.
- `debug`: Un módulo para imprimir variables o mensajes, indispensable para depurar.
- `setup`: Un módulo especial que recolecta una gran cantidad de información (facts) sobre el sistema remoto.

--- 

## ✅ Paso 5: Ejecutar el Playbook

Desde tu terminal (con el entorno virtual activado), ejecuta:

```bash
ansible-playbook -i inventory.ini check_reachability.yml
```

- `-i inventory.ini`: Especifica qué archivo de inventario usar.

Verás cómo Ansible se conecta a cada host y ejecuta las tareas. La salida de la última tarea te mostrará un mensaje personalizado con el sistema operativo y la memoria de cada uno de los 10 nodos, confirmando que la automatización funciona.