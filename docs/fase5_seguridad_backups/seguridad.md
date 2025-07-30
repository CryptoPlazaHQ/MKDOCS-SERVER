# 🔒 Fase 5 – Hardening, Secretos y Control de Acceso

## 🎯 Objetivo
Implementar una estrategia de seguridad de varias capas para proteger la infraestructura. Esto implica fortalecer la configuración de los servicios expuestos (`hardening`), gestionar de forma segura las credenciales y tokens (`secret management`), y asegurar las interfaces de gestión con autenticación de múltiples factores (`MFA`).

---

## 🤔 El Principio de Mínimo Privilegio

La estrategia de seguridad se basa en este principio: cada componente del sistema solo debe tener los permisos estrictamente necesarios para realizar su función, y nada más. Aplicaremos esto a los secretos, el acceso SSH y las cuentas de usuario.

--- 

## 🔑 Gestión de Secretos con Ansible Vault

**El Problema:** Poner contraseñas, tokens de API o claves SSH en texto plano dentro de tu código es una de las peores prácticas de seguridad. Si tu código se filtra o se sube a un repositorio público, toda tu infraestructura queda comprometida.

**La Solución:** **Ansible Vault** es una herramienta integrada en Ansible que permite encriptar archivos de variables o incluso variables individuales. Estos "vaults" solo se pueden desencriptar con una contraseña que tú posees.

### 1. Crear un Vault Central

Por convención, los secretos que se aplican a todos los hosts se guardan en `group_vars/all/vault.yml`.

```bash
# Crea el directorio si no existe
mkdir -p group_vars/all

# Crea y edita el archivo encriptado
ansible-vault create group_vars/all/vault.yml
```
- Se te pedirá que crees y confirmes una contraseña para la bóveda. **Usa una contraseña fuerte y guárdala en tu gestor de contraseñas.**
- Se abrirá un editor de texto. Añade tus secretos:
  ```yaml
  # group_vars/all/vault.yml
  netbox_api_token: TU_TOKEN_DE_API_DE_NETBOX_AQUI
  ```

### 2. Referenciar los Secretos de Forma Segura

Actualiza tu archivo de inventario dinámico (`produccion.netbox.yml`) para que lea el token desde el vault. Ansible lo hará automáticamente por convención de nombres, pero para ser explícitos:

```yaml
# produccion.netbox.yml
plugin: netbox.netbox.nb_inventory
api_endpoint: http://<IP_DE_TU_SERVIDOR_NETBOX>
token: "{{ netbox_api_token }}" # Ansible buscará esta variable en los archivos de group_vars
# ... resto del archivo
```

### 3. Ejecutar Playbooks con el Vault

Para que Ansible pueda desencriptar el archivo, debes proporcionar la contraseña al ejecutar un playbook. Hay dos maneras:

- **Interactivo (Recomendado para uso manual):**
  ```bash
  ansible-playbook -i produccion.netbox.yml mi_playbook.yml --ask-vault-pass
  ```
- **Usando un Archivo de Contraseña (Para Scripts o CI/CD):**
  1.  Crea un archivo (ej: `.vault_pass`) que contenga solo la contraseña del vault.
  2.  Asegúrate de que este archivo tenga permisos muy restrictivos: `chmod 600 .vault_pass`.
  3.  Ejecuta el playbook: `ansible-playbook -i produccion.netbox.yml mi_playbook.yml --vault-password-file .vault_pass`

--- 

## 🛡️ Hardening del Servidor SSH

El acceso SSH es la puerta de entrada a tus nodos. Debemos hacerlo lo más seguro posible.

**Playbook `harden_ssh.yml`**
```yaml
---
- name: Fortalecer la Configuracion del Servidor SSH en Nodos Windows
  hosts: Nodo_Cliente
  gather_facts: no

  tasks:
    - name: Aplicar configuraciones de seguridad a sshd_config
      ansible.windows.win_lineinfile:
        path: C:\ProgramData\ssh\sshd_config
        regexp: "^#?{{ item.key }}"
        line: "{{ item.key }} {{ item.value }}"
        state: present
      loop:
        - { key: 'PasswordAuthentication', value: 'no' } # Deshabilita login con contraseña
        - { key: 'PermitRootLogin', value: 'no' } # Prohíbe el login del usuario Administrador
        - { key: 'PubkeyAuthentication', value: 'yes' } # Asegura que la autenticación por clave esté activa
        - { key: 'PermitEmptyPasswords', value: 'no' } # No permitir contraseñas vacías

    - name: Reiniciar el servicio SSHD para aplicar los cambios
      ansible.windows.win_service:
        name: sshd
        state: restarted
```
- **Loop:** Este playbook usa un `loop` para aplicar múltiples configuraciones de forma limpia y eficiente.
- **¡ADVERTENCIA!** Antes de ejecutar esto, asegúrate de que tu acceso con clave SSH funciona perfectamente. Si no, **perderás el acceso a tus nodos.**

--- 

## 🔐 Autenticación de Múltiples Factores (MFA/2FA)

Proteger tus interfaces de gestión es tan importante como proteger los nodos. Un atacante que acceda a tu NetBox o Grafana puede causar estragos.

- **NetBox:** Ve a tu perfil de usuario > **Two-Factor Authentication**. Habilítalo usando una aplicación de autenticación como Google Authenticator, Authy o una llave de seguridad física como YubiKey.
- **Grafana:** Ve a tu perfil de usuario > **Security** > **Two-factor auth**. El proceso es similar.

**MFA es una de las formas más efectivas de prevenir el acceso no autorizado a cuentas.**