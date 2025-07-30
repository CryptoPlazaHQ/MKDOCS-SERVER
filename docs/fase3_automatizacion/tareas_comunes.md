# ⚙️ Fase 3 – Playbooks para Tareas Comunes de Gestión en Windows

## 🎯 Objetivo
Crear un conjunto de playbooks de Ansible modulares y reutilizables para ejecutar las tareas de administración más frecuentes en un entorno Windows. Esto incluye la gestión de paquetes de software, la manipulación de servicios y la transferencia de archivos, sentando las bases para una automatización más compleja.

---

## 📦 Playbook 1: Gestión de Software con Chocolatey

La gestión manual de software en Windows (descargar un `.exe`, hacer clic en "Siguiente") no es escalable. La solución es usar un gestor de paquetes. **Chocolatey** es el estándar de facto en la comunidad de automatización de Windows.

### a) Instalar Chocolatey

Este playbook instala el gestor de paquetes Chocolatey en los nodos, pero solo si no está ya presente.

**`install_chocolatey.yml`**
```yaml
---
- name: Instalar y Configurar Chocolatey en Nodos Windows
  hosts: Nodo_Cliente
  gather_facts: no

  tasks:
    - name: Instalar Chocolatey desde su script oficial
      ansible.windows.win_shell: |
        Set-ExecutionPolicy Bypass -Scope Process -Force;
        [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072;
        iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
      args:
        creates: 'C:\ProgramData\chocolatey\choco.exe'
      register: choco_install_result

    - name: Mostrar mensaje de estado
      debug:
        msg: "Chocolatey ya estaba instalado." 
      when: not choco_install_result.changed
```
- **Idempotencia:** La cláusula `args.creates` es la que garantiza que este playbook no haga nada si detecta que `choco.exe` ya existe, haciendo la operación segura para ejecuciones repetidas.

### b) Instalar, Actualizar y Eliminar Paquetes

Una vez instalado Chocolatey, podemos usar el módulo `win_chocolatey` de Ansible para gestionar el software de forma declarativa.

**`manage_software.yml`**
```yaml
---
- name: Gestionar Paquetes de Software Esenciales
  hosts: Nodo_Cliente
  gather_facts: no

  tasks:
    - name: Asegurar que 7-Zip y Notepad++ esten instalados
      ansible.windows.win_chocolatey:
        name:
          - 7zip
          - notepadplusplus
        state: present

    - name: Asegurar que una version especifica de Git este instalada
      ansible.windows.win_chocolatey:
        name: git
        version: '2.30.1'
        state: present

    - name: Asegurar que un paquete antiguo (ej: putty) este desinstalado
      ansible.windows.win_chocolatey:
        name: putty
        state: absent
```
- **Estado Declarativo:** Simplemente declaras el `state` que deseas (`present`, `absent`, `latest`) y Ansible se encarga del resto.

--- 

## ⚙️ Playbook 2: Gestión de Servicios de Windows

Este playbook asegura que los servicios críticos estén en el estado correcto (corriendo, detenido, etc.) y configurados para iniciarse correctamente.

**`manage_services.yml`**
```yaml
---
- name: Gestionar Servicios Críticos de Windows
  hosts: Nodo_Cliente
  gather_facts: no

  tasks:
    - name: Asegurar que el servicio SSHD este iniciado y en modo automatico
      ansible.windows.win_service:
        name: sshd
        state: started
        start_mode: auto

    - name: Asegurar que el servicio de Windows Update este habilitado pero detenido
      ansible.windows.win_service:
        name: wuauserv
        state: stopped
        start_mode: manual
      register: service_result

    - name: Reiniciar el servicio del exporter de monitoreo si fue modificado
      ansible.windows.win_service:
        name: windows_exporter
        state: restarted
      when: service_result.changed # Ejemplo de como reiniciar un servicio solo si otra tarea hizo un cambio
```
- **Control Fino:** Puedes definir no solo si un servicio está corriendo, sino también su modo de inicio, y puedes usar los resultados de una tarea para disparar otra (como reiniciar un servicio).

--- 

## 📂 Playbook 3: Gestión de Archivos y Plantillas

Este playbook muestra cómo copiar archivos y cómo usar plantillas para desplegar archivos de configuración personalizados.

**`manage_files.yml`**
```yaml
---
- name: Gestionar Archivos y Configuraciones
  hosts: Nodo_Cliente
  gather_facts: yes # Necesitamos los facts para las plantillas

  tasks:
    - name: Copiar un archivo de banner de bienvenida
      ansible.windows.win_copy:
        src: files/motd.txt # Un archivo en tu nodo de control
        dest: C:\etc\motd.txt

    - name: Desplegar un archivo de configuracion desde una plantilla
      ansible.windows.win_template:
        src: templates/config.ini.j2 # Un archivo de plantilla Jinja2
        dest: C:\ProgramData\MyApp\config.ini
```

**Ejemplo de plantilla `templates/config.ini.j2`:**
```ini
[Settings]
Hostname = {{ ansible_hostname }}
IP_Address = {{ ansible_default_ipv4.address }}
Memory_MB = {{ ansible_memory_mb.real.total }}
```
- **Plantillas Jinja2:** El módulo `win_template` te permite usar variables de Ansible (los `facts` que recolectamos) dentro de tus archivos. Al desplegar la plantilla, Ansible reemplazará `{{ ansible_hostname }}` por el nombre real del host, creando un archivo de configuración personalizado para cada nodo.