# 🧪 Fase 0 – PoC Mínima y Setup Inicial

## 🎯 Objetivo
El objetivo de esta fase es establecer una **Prueba de Concepto (PoC)**. Se trata de validar la conexión más fundamental: que tú, como **gestor remoto** desde Colombia, puedas acceder de forma fiable a un **único MiniPC** en Estados Unidos. Este paso es la base sobre la que se construirá toda la infraestructura.

---

##  roles

- **Operador Local (EE.UU.):** La persona que tiene acceso físico a los equipos. Su tarea es conectar los cables, encender los MiniPCs y realizar la configuración inicial de red si es necesario.
- **Gestor Remoto (Tú, en Colombia):** El arquitecto y administrador de toda la red. Tu trabajo es configurar, gestionar, automatizar y monitorear todos los nodos de forma remota.

---

## 🧱 Infraestructura Mínima Requerida

| Rol | Equipo | Ubicación | Detalles Clave |
|-----|--------|-----------|----------|
| Nodo Remoto (Gestionado) | 1x MiniPC E2 | EE.UU. | Conectado a Internet. Debe tener una IP pública fija asignada por el ISP. |
| Plano de Control (Gestor) | Tu Laptop/PC | Colombia | Desde aquí ejecutarás todos los comandos de gestión (SSH, Ansible, etc.). |
| Red Física | Switch PoE & Router | EE.UU. | El MiniPC debe estar conectado al router que le provee la IP pública. |

---

## 📡 Verificación de Conectividad: El Primer Ping

Antes de cualquier configuración compleja, debemos verificar la conectividad a nivel de red. Tu ISP en EE.UU. te ha proporcionado la siguiente información:

- **Bloque de IPs:** `50.190.105.81 – 93`
- **Puerta de Enlace (Gateway):** `50.190.105.94`
- **Máscara de Subred:** `255.255.255.240`

El operador local debe haber configurado el primer MiniPC para usar la IP `50.190.105.81`.

Desde tu terminal en Colombia, ejecuta dos pruebas de `ping`:

1.  **Ping a la Puerta de Enlace:**
    ```bash
    ping 50.190.105.94
    ```
    - **Propósito:** Esto verifica que puedes llegar a la "puerta de entrada" de la red remota. Si esto funciona, tu conexión a Internet y el enrutamiento básico hacia el ISP de EE.UU. son correctos.

2.  **Ping al Nodo:**
    ```bash
    ping 50.190.105.81
    ```
    - **Propósito:** Esto verifica que el MiniPC específico está encendido, tiene la IP correctamente configurada y su firewall no está bloqueando las solicitudes de ping (ICMP).

Si ambos pings reciben respuesta, la conectividad de red básica está confirmada.

---

## 🔑 Habilitación del Acceso Remoto (SSH)

Ahora, necesitas una forma de ejecutar comandos en el nodo. Usaremos SSH.

Si el MiniPC tiene una instalación limpia de Windows, el servidor SSH no estará activado. El **operador local** debe ejecutar el siguiente script de PowerShell **como Administrador** en el MiniPC. Este script se detalla en la siguiente sección.

```powershell
# Script para habilitar el servidor OpenSSH en Windows
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
Start-Service sshd
Set-Service -Name sshd -StartupType 'Automatic'
New-NetFirewallRule -Name sshd -DisplayName 'SSH Server' -Enabled True -Direction Inbound -Protocol TCP -Action Allow -LocalPort 22
```

Una vez que el operador confirme que el script se ha ejecutado, puedes intentar la conexión desde Colombia:

```bash
# Reemplaza "admin" con el nombre de usuario del MiniPC
ssh admin@50.190.105.81
```

Si la conexión es exitosa, has validado el pilar fundamental de tu operación remota.

---

## ✅ Checklist de Validación de la Fase 0

- [ ] El rol del operador local y del gestor remoto están claramente definidos.
- [ ] El MiniPC está físicamente conectado y encendido en EE.UU.
- [ ] La IP `50.190.105.81` está correctamente asignada al MiniPC.
- [ ] El `ping` a la puerta de enlace (`.94`) es exitoso desde Colombia.
- [ ] El `ping` al nodo (`.81`) es exitoso desde Colombia.
- [ ] El script para habilitar SSH ha sido ejecutado por el operador local.
- [ ] La conexión SSH al nodo desde Colombia es exitosa.
- [ ] Las credenciales de acceso (usuario/contraseña o clave SSH) están guardadas de forma segura en un gestor de contraseñas (como Bitwarden, 1Password, etc.).