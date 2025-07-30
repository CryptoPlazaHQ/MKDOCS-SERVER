# 🌐 Guía de Asignación de IP Fija para Operador Local

## 🎯 Objetivo
Proporcionar al **operador local** en Estados Unidos un manual de operaciones claro y detallado para configurar una dirección IP pública fija en cada MiniPC Windows. Este proceso es la base para que el gestor remoto pueda acceder a cada nodo de forma individual y predecible.

---

## 📦 Requisitos Previos

- **Acceso Físico:** El operador debe tener acceso físico al MiniPC.
- **Credenciales de Administrador:** Se necesita una cuenta de usuario en Windows con privilegios de administrador.
- **Información de Red del ISP:** El operador debe tener a mano los siguientes datos proporcionados por el proveedor de Internet (ej: Comcast, Spectrum):
    - La **Dirección IP** específica a asignar (ej: `50.190.105.82`).
    - La **Máscara de subred** (ej: `255.255.255.240`).
    - La **Puerta de enlace predeterminada** (Gateway) (ej: `50.190.105.94`).
    - Los **Servidores DNS** (ej: `8.8.8.8` y `1.1.1.1` son buenas opciones públicas).

---

## 🧭 Métodos de Configuración

Existen dos maneras de configurar la IP: a través de la interfaz gráfica (GUI) o mediante la línea de comandos (PowerShell). El operador puede elegir la que le resulte más cómoda.

### 🔧 Método 1: Interfaz Gráfica (GUI)

Este método es visual y recomendado para quienes no se sienten cómodos con la línea de comandos.

1.  **Abrir Conexiones de Red:**
    -   Presiona la tecla de `Windows + R` para abrir el cuadro de diálogo "Ejecutar".
    -   Escribe `ncpa.cpl` y presiona Enter. Esto abrirá directamente la ventana de Conexiones de Red.

2.  **Acceder a Propiedades:**
    -   Busca la conexión de red activa (usualmente llamada "Ethernet").
    -   Haz clic derecho sobre ella y selecciona **"Propiedades"**.

3.  **Configurar IPv4:**
    -   En la lista, selecciona **"Protocolo de Internet versión 4 (TCP/IPv4)"**.
    -   Haz clic en el botón **"Propiedades"**.

4.  **Ingresar los Datos de Red:**
    -   Selecciona la opción **"Usar la siguiente dirección IP"**.
    -   Rellena los campos con la información del ISP:
        -   **Dirección IP:** `50.190.105.82`
        -   **Máscara de subred:** `255.255.255.240`
        -   **Puerta de enlace predeterminada:** `50.190.105.94`
    -   Selecciona la opción **"Usar las siguientes direcciones de servidor DNS"**.
        -   **Servidor DNS preferido:** `8.8.8.8`
        -   **Servidor DNS alternativo:** `1.1.1.1`

5.  **Guardar y Cerrar:**
    -   Haz clic en **"Aceptar"** en ambas ventanas de propiedades para guardar los cambios.
    -   La conexión de red podría reiniciarse brevemente.

### 💻 Método 2: PowerShell (Más Rápido y Automatizable)

Este método es ideal para operadores con más experiencia o para automatizar el proceso en el futuro.

1.  **Abrir PowerShell como Administrador:**
    -   Busca "PowerShell" en el menú de inicio, haz clic derecho y selecciona "Ejecutar como administrador".

2.  **Identificar la Interfaz de Red:**
    -   Ejecuta `Get-NetAdapter` para ver el nombre exacto de la interfaz de red (ej: "Ethernet", "Ethernet 2"). Anota el valor de la columna `Name`.

3.  **Ejecutar el Comando de Configuración:**
    -   Copia y pega el siguiente bloque de comandos, reemplazando `"Ethernet"` si el nombre de tu interfaz es diferente, y ajustando las IPs si es necesario.

    ```powershell
    # Asigna la IP estática, la máscara de subred (PrefixLength 28 = 255.255.255.240) y el gateway
    New-NetIPAddress -InterfaceAlias "Ethernet" `
      -IPAddress "50.190.105.82" `
      -PrefixLength 28 `
      -DefaultGateway "50.190.105.94"

    # Configura los servidores DNS
    Set-DnsClientServerAddress -InterfaceAlias "Ethernet" `
      -ServerAddresses ("8.8.8.8", "1.1.1.1")
    ```

4.  **Verificar:** La configuración se aplica inmediatamente. Puedes usar `ipconfig` para verificar que los cambios se hayan realizado.

---

## ✅ Checklist de Entrega del Operador

Antes de notificar al gestor remoto que el nodo está listo, el operador local debe confirmar:

| ✅ | Acción | Comando de Verificación |
| --- | --- | --- |
| ☐ | La IP está correctamente asignada. | `ipconfig` |
| ☐ | El nodo puede contactar a su gateway. | `ping 50.190.105.94` |
| ☐ | El nodo tiene salida a Internet. | `ping 8.8.8.8` |
| ☐ | El servicio SSH está corriendo. | `Get-Service sshd` |
| ☐ | El puerto 22 está abierto en el firewall. | `Get-NetFirewallRule -DisplayName "SSH Server"` |
| ☐ | Se han entregado las credenciales de `admin` al gestor remoto. | (Confirmación manual) |