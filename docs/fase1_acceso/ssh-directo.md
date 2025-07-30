# 🔐 Fase 1 – Acceso Remoto Seguro (SSH sobre IP Pública)

## 🎯 Objetivo
Establecer un método de acceso remoto que sea **seguro, estable y profesional**, utilizando claves SSH en lugar de contraseñas. Además, se busca optimizar el flujo de trabajo del gestor remoto mediante el uso de un archivo de configuración de SSH local.

---

## 🤔 ¿Por qué no usar Contraseñas?

Aunque la autenticación con contraseña es común, es **altamente insegura** para sistemas expuestos a Internet:
- **Vulnerables a Fuerza Bruta:** Bots automatizados escanean constantemente Internet en busca de puertos SSH abiertos e intentan adivinar contraseñas comunes (`admin`, `password123`, etc.).
- **Filtraciones:** Si esa misma contraseña se usa en otro servicio que es hackeado, los atacantes la probarán en tus servidores.
- **Gestión Ineficiente:** Recordar y rotar contraseñas para 250 nodos es una pesadilla operativa.

La **autenticación por clave pública (SSH Keys)** resuelve estos problemas. Es el estándar de la industria.

--- 

## KeyPair)

Funciona con un par de claves criptográficas:
- **Clave Privada (`id_ed25519`):** Es tu secreto. Vive en tu máquina de gestión (Colombia) y nunca debe ser compartida. Es como la llave física de tu casa.
- **Clave Pública (`id_ed25519.pub`):** Es la cerradura. La copias a los servidores remotos que quieres gestionar. Puedes compartirla libremente, no es un secreto.

Cuando te conectas, el servidor usa la clave pública para verificar la firma digital creada por tu clave privada, probando tu identidad sin necesidad de una contraseña.

### ✅ Paso 1: Generar tu Par de Claves SSH (Si no lo has hecho)

En tu máquina de gestión en Colombia, abre una terminal y ejecuta:

```bash
# ed25519 es el algoritmo recomendado actualmente: es seguro y rápido.
# -C añade un comentario, usualmente tu email, para identificar la clave.
ssh-keygen -t ed25519 -C "tu_correo@ejemplo.com"
```
- Acepta la ruta por defecto (`~/.ssh/id_ed25519`).
- **IMPORTANTE:** Se te pedirá una **passphrase**. Es una contraseña para proteger tu clave privada. Si alguien robara tu archivo de clave privada, aún necesitaría esta passphrase para poder usarla. **Se recomienda encarecidamente configurar una.**

### ✅ Paso 2: Copiar tu Clave Pública al Nodo Remoto

Ahora, el operador local debe tomar el contenido de tu clave pública (`~/.ssh/id_ed25519.pub`) y añadirlo a un archivo específico en el MiniPC.

**En el MiniPC (ejecutado por el operador local):**

```powershell
# Crear la carpeta .ssh si no existe
New-Item -Path "C:\Users\admin\.ssh" -ItemType Directory -Force

# Crear el archivo authorized_keys y pegar el contenido de la clave pública
# Reemplazar "CONTENIDO_DE_LA_CLAVE_PUBLICA" con tu clave real (ssh-ed25519 AAAA...)
"CONTENIDO_DE_LA_CLAVE_PUBLICA" | Out-File -FilePath "C:\Users\admin\.ssh\authorized_keys" -Encoding utf8 -Append
```

### ✅ Paso 3: Optimizar tu Flujo de Trabajo con `~/.ssh/config`

Para evitar escribir `ssh -i ~/.ssh/id_ed25519 admin@50.190.105.81` cada vez, puedes crear alias en un archivo de configuración.

En tu máquina de gestión, edita o crea el archivo `~/.ssh/config` y añade una entrada por cada nodo:

```sshconfig
# =====================================
# Cliente A - Nodos en Miami
# =====================================
Host nodo-a-01
  HostName 50.190.105.81
  User admin
  IdentityFile ~/.ssh/id_ed25519

Host nodo-a-02
  HostName 50.190.105.82
  User admin
  IdentityFile ~/.ssh/id_ed25519

# ... y así sucesivamente para los 10 nodos
```

Ahora, para conectarte al primer nodo, todo lo que necesitas escribir es:

```bash
ssh nodo-a-01
```

--- 

## 🛡️ Siguiente Nivel de Seguridad: IP Allowlist

Incluso con claves SSH, es una buena práctica restringir desde qué IPs se puede intentar una conexión. Esto se hace en el firewall del nodo.

Desde el MiniPC, ejecuta este comando PowerShell (como Admin):

```powershell
# Permitir conexiones al puerto 22 solo desde una IP específica (la tuya en Colombia)
# Reemplaza "TU.IP.PUBLICA.COLOMBIA" con tu IP pública actual.
New-NetFirewallRule -DisplayName "Allow SSH Colombia Only" `
  -Direction Inbound -Protocol TCP -LocalPort 22 `
  -RemoteAddress "TU.IP.PUBLICA.COLOMBIA" `
  -Action Allow
```

**Nota sobre IP Dinámica:** Si tu IP en Colombia cambia, deberás actualizar esta regla. En fases futuras, una VPN (WireGuard) puede ser una solución más robusta para este problema.