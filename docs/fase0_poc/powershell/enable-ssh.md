# ⚙️ Script PowerShell: Habilitar Acceso SSH en Windows

## 🎯 Propósito del Script
Este script de PowerShell automatiza todos los pasos necesarios para activar y configurar el servidor OpenSSH en un MiniPC con Windows 10 o 11. Su objetivo es preparar el nodo para que pueda ser gestionado remotamente de forma segura, eliminando la necesidad de acceso físico o de Escritorio Remoto (RDP) para tareas administrativas.

---

## 📜 El Código Completo

```powershell
# enable-ssh.ps1
# ------------------------------------------
# Activación y configuración del servidor SSH en Windows 10/11
# Ejecutar como administrador
# ------------------------------------------

# Escribe un mensaje en la consola para informar al operador de lo que está sucediendo.
Write-Host "🔧 Habilitando el componente OpenSSH.Server en este equipo..."

# Instala el servicio SSH si no está presente. Es una capacidad opcional de Windows.
# El modificador -Online significa que busca la capacidad en Windows Update si es necesario.
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0

# Inicia el servicio SSH (sshd). Si ya estaba en ejecución, no hace nada.
Write-Host "🚀 Iniciando el servicio sshd..."
Start-Service sshd

# Configura el servicio para que se inicie automáticamente cada vez que el sistema arranque.
# Esto es crucial para asegurar el acceso remoto persistente después de reinicios.
Write-Host "⚙️ Configurando el servicio para inicio automático..."
Set-Service -Name sshd -StartupType 'Automatic'

# Crea una regla en el Firewall de Windows para permitir conexiones entrantes en el puerto 22 (el puerto estándar de SSH).
# Sin esta regla, el firewall bloquearía todos los intentos de conexión.
Write-Host "🛡️ Creando regla en el Firewall para el puerto 22..."
New-NetFirewallRule -Name sshd -DisplayName 'SSH Server' `
  -Enabled True -Direction Inbound -Protocol TCP `
  -Action Allow -LocalPort 22

# Verifica el estado final del servicio y lo muestra en pantalla.
Write-Host "✅ Verificación final del estado del servicio:"
Get-Service sshd

Write-Host "`n🎉 ¡Éxito! El servidor SSH está habilitado y configurado. El nodo está listo para acceso remoto."
```

---

## 🚀 ¿Cómo Ejecutar el Script?

El operador local en EE.UU. debe ejecutar este script con privilegios de Administrador. Hay dos maneras de hacerlo:

### Método 1: Ejecución Local (Copiando el archivo)
1.  Guarda el código anterior en un archivo llamado `enable-ssh.ps1`.
2.  Transfiere el archivo al MiniPC (vía USB, email, etc.).
3.  Haz clic derecho sobre el archivo y selecciona **"Ejecutar con PowerShell"**. Asegúrate de que se ejecute en una ventana de PowerShell (Administrador).

### Método 2: Ejecución Remota (Recomendado)

Esta es la forma más profesional. Alojas el script en un único lugar (como un Gist de GitHub) y el operador solo necesita ejecutar un comando para descargarlo y ejecutarlo al mismo tiempo. Esto asegura que siempre se use la versión más actualizada del script.

1.  **Aloja el script:** Crea un [Gist público en GitHub](https://gist.github.com/) y pega el código del script. Obtén la URL "Raw" del archivo.

2.  **Comando de ejecución para el operador:** El operador abre PowerShell (Administrador) en el MiniPC y ejecuta:

    ```powershell
    # Reemplaza la URL con la URL "Raw" de tu Gist
    irm https://gist.githubusercontent.com/tu-usuario/tu-gist-id/raw/enable-ssh.ps1 | iex
    ```

    - `irm` (`Invoke-RestMethod`): Descarga el contenido de la URL.
    - `|` (Pipe): Envía la salida del primer comando (el texto del script) al siguiente.
    - `iex` (`Invoke-Expression`): Ejecuta el texto recibido como un comando de PowerShell.

---

## ✅ Resultado Esperado

Independientemente del método, el resultado final es que el MiniPC:
- Queda accesible vía SSH desde Internet a través de su IP pública (ej: `ssh admin@50.190.105.81`).
- No interfiere con el funcionamiento de otras aplicaciones como Salad, Honeygain o EarnApp.
- El acceso SSH persistirá incluso después de que el MiniPC se reinicie.