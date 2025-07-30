# 🛡️ Guía de Configuración del Firewall de Windows

## 🎯 Objetivo
Configurar de forma precisa y segura el Firewall de Windows en cada MiniPC. El objetivo es aplicar el **principio de mínimo privilegio**, permitiendo únicamente el tráfico estrictamente necesario (en este caso, el acceso SSH desde la IP del gestor remoto) y bloqueando todo lo demás. Esto reduce drásticamente la superficie de ataque del nodo.

---

## 🤔 ¿Por qué es tan importante el Firewall?

Un MiniPC con una IP pública en Internet es un blanco visible para escaneos automáticos y ataques de todo el mundo. Estos bots buscan puertos abiertos y vulnerabilidades conocidas. El Firewall de Windows es tu **primera línea de defensa a nivel de sistema operativo**. Si no se configura correctamente, dejas la puerta abierta a accesos no autorizados.

--- 

## 📜 El Proceso de Configuración en 3 Pasos

Realizaremos la configuración usando PowerShell, ya que es un método preciso, repetible y automatizable con Ansible.

### ✅ Paso 1: Crear una Regla Específica y Restrictiva

En lugar de simplemente abrir el puerto 22 para todo el mundo, crearemos una regla que solo permita el tráfico SSH (`puerto 22`) desde una única dirección IP de origen (la tuya en Colombia).

**Comando de PowerShell (ejecutar como Administrador en el MiniPC):**

```powershell
# Define la IP pública del gestor remoto en una variable
$remoteIp = "TU.IP.PUBLICA.COLOMBIA"

# Crea la regla del firewall
New-NetFirewallRule -DisplayName "Allow SSH Management (Colombia Only)" `
  -Name "AllowSSHManagement" `
  -Direction Inbound `
  -Protocol TCP `
  -LocalPort 22 `
  -RemoteAddress $remoteIp `
  -Action Allow `
  -Enabled True

Write-Host "Regla de firewall 'Allow SSH Management' creada para la IP $remoteIp"
```

**Análisis del Comando:**
- `-DisplayName`: Un nombre descriptivo que verás en la interfaz gráfica del firewall.
- `-Name`: Un identificador único para la regla.
- `-Direction Inbound`: La regla se aplica al tráfico que entra al MiniPC.
- `-LocalPort 22`: Especifica que la regla afecta solo al puerto SSH.
- `-RemoteAddress $remoteIp`: **Esta es la parte más importante.** Restringe la conexión para que solo se permita si se origina desde esta IP.
- `-Action Allow`: Permite el tráfico que cumple las condiciones.

### ✅ Paso 2: Verificar que la Regla General de SSH está Deshabilitada

El script `enable-ssh.ps1` de la Fase 0 creó una regla muy permisiva llamada `sshd`. Debemos asegurarnos de que esté deshabilitada o eliminarla para que nuestra regla más restrictiva sea la única que aplique.

**Comando de Verificación:**
```powershell
Get-NetFirewallRule -Name "sshd"
```
Si la regla existe y está `Enabled: True`, deshabilítala:

```powershell
Set-NetFirewallRule -Name "sshd" -Enabled False
Write-Host "Regla de firewall 'sshd' general deshabilitada."
```
O, para una limpieza total, elimínala (recomendado):
```powershell
Remove-NetFirewallRule -Name "sshd"
Write-Host "Regla de firewall 'sshd' general eliminada."
```

### ✅ Paso 3: Validar la Configuración Final

Para estar seguros de que solo tu regla está activa, puedes listar todas las reglas que afectan al puerto 22.

```powershell
Get-NetFirewallRule | Where-Object { $_.LocalPort -eq 22 -and $_.Enabled -eq $true } | Format-Table Name, DisplayName, RemoteAddress
```
- **Resultado esperado:** Deberías ver únicamente tu regla `AllowSSHManagement`, y en la columna `RemoteAddress` debe aparecer tu IP de Colombia.

--- 

## ⚠️ Gestión de IPs Dinámicas

Si tu IP pública en Colombia cambia con frecuencia, la regla del firewall dejará de funcionar y perderás el acceso. Tienes varias opciones:
1.  **Actualización Manual:** Conectarte al nodo a través de otro medio (si lo tienes) o pedirle al operador local que actualice la regla con tu nueva IP.
2.  **Usar un Rango de IPs:** Si tu ISP te asigna IPs dentro de un rango predecible, puedes especificar el rango en la regla (ej: `190.85.48.0/24`).
3.  **DDNS (DNS Dinámico):** Usar un servicio de DDNS para asociar un nombre de dominio a tu IP cambiante y crear una regla de firewall basada en ese nombre (más complejo).
4.  **Implementar una VPN (WireGuard):** Esta es la solución más robusta a largo plazo (discutida en el Roadmap), ya que crea un túnel seguro y el acceso ya no dependería de tu IP pública.