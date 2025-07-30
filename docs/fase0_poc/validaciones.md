# ✅ Fase 0 – Validaciones de Red y Conectividad

## 🎯 Objetivo
Documentar un conjunto de pruebas y comandos de validación que tanto el **operador local** como el **gestor remoto** deben ejecutar para confirmar que la configuración inicial del nodo es correcta y que la comunicación es fluida. Este proceso de validación es crucial para diagnosticar problemas tempranamente.

---

## 1️⃣ Validaciones del Operador Local (En el MiniPC de EE.UU.)

Después de asignar la IP fija y ejecutar el script `enable-ssh.ps1`, el operador local debe realizar estas comprobaciones desde una ventana de PowerShell en el propio MiniPC.

### a) Verificar la Configuración IP

```powershell
ipconfig
```
- **Qué buscar:**
  - **Dirección IPv4:** Debe ser la IP pública asignada (ej: `50.190.105.81`).
  - **Máscara de subred:** Debe coincidir con la del ISP (ej: `255.255.255.240`).
  - **Puerta de enlace predeterminada:** Debe ser el gateway del ISP (ej: `50.190.105.94`).

### b) Probar la Conexión a la Puerta de Enlace

```powershell
ping 50.190.105.94
```
- **Propósito:** Confirma que el MiniPC puede comunicarse con su router o gateway más cercano. Si esto falla, hay un problema de red local (cable, switch, configuración IP).

### c) Probar la Salida a Internet

```powershell
ping 8.8.8.8
```
- **Propósito:** Confirma que el nodo puede alcanzar un servidor externo en Internet (8.8.8.8 es un servidor DNS público de Google). Si el paso anterior funcionó pero este no, podría haber un problema en el router del ISP.

### d) Verificar la IP Pública Real

```powershell
curl ifconfig.me
```
- **Propósito:** Confirma que la IP pública con la que el nodo sale a Internet es la misma que se configuró manualmente. Esto es vital para asegurar que no hay un doble NAT o una configuración de red inesperada.

### e) Comprobar el Estado del Servicio SSH

```powershell
Get-Service sshd
```
- **Qué buscar:**
  - **Status:** Debe ser `Running`.
  - **StartType:** Debe ser `Automatic`.

--- 

## 2️⃣ Validaciones del Gestor Remoto (Desde Colombia)

Una vez que el operador local ha confirmado que todo está correcto en el sitio, es tu turno para validar desde tu puesto de gestión.

### a) Validar la Ruta de Red

```bash
# En Linux/macOS
traceroute 50.190.105.81

# En Windows
tracert 50.190.105.81
```
- **Propósito:** Muestra todos los "saltos" (routers) por los que pasa tu conexión hasta llegar al nodo. Es útil para diagnosticar dónde podría estar un problema de latencia o un bloqueo de red si el ping no funcionara.

### b) Probar la Apertura del Puerto SSH

Necesitas verificar si el puerto 22 está abierto y escuchando en el nodo remoto. Un simple `ping` no es suficiente.

```bash
# Usando Netcat (nc), disponible en Linux/macOS/WSL
nc -zv 50.190.105.81 22
```
- **Respuesta esperada:** `Connection to 50.190.105.81 port 22 [tcp/ssh] succeeded!`
- **Si falla:** Significa que algo está bloqueando el puerto 22. Las causas más probables son:
  - La regla del Firewall de Windows no se aplicó correctamente.
  - El firewall del ISP (a nivel de proveedor) está bloqueando el puerto.

### c) Intento de Conexión SSH Final

```bash
ssh -v admin@50.190.105.81
```
- **Propósito:** Es la prueba definitiva. El flag `-v` (verbose) te dará una salida detallada del proceso de conexión, lo que es extremadamente útil para diagnosticar problemas de autenticación (ej: clave incorrecta, permisos de archivo, etc.).
