# ✅ Validaciones de Conectividad Remota desde Colombia

## 🎯 Objetivo
Verificar que el MiniPC remoto con IP pública fija está completamente accesible y seguro para gestión remota, sin afectar su funcionamiento normal.

---

## 🧭 ¿Desde dónde estás trabajando?

Desde Colombia, usando:

- Una conexión residencial
- IP pública dinámica (usualmente)
- Acceso SSH desde terminal local

---

## 1️⃣ Verificar conectividad con `ping`

```bash
ping 50.190.105.81
```

✅ Debes recibir respuesta de `50.190.105.81` con tiempos de latencia (~80–120ms)

* * *

2️⃣ Probar acceso por puerto 22 (SSH)
-------------------------------------

```bash
nc -zv 50.190.105.81 22
```

✅ Debes recibir: `Connection to 50.190.105.81 port 22 [tcp/ssh] succeeded!`

Si no, revisar:

*   Firewall Windows
    
*   IP allowlist
    
*   Puerto 22 habilitado
    

* * *

3️⃣ Iniciar sesión SSH con clave
--------------------------------

```bash
ssh admin@50.190.105.81
```

Si usas clave privada:

```bash
ssh -i ~/.ssh/id_ed25519 admin@50.190.105.81
```

* * *

4️⃣ Confirmar sistema remoto (comandos de diagnóstico)
------------------------------------------------------

Una vez dentro del MiniPC:

```powershell
ipconfig
```

Verifica que estás usando la IP fija asignada (`50.190.105.81`)  
Revisa que el gateway y DNS estén bien configurados.

* * *

5️⃣ Probar conexión saliente desde el MiniPC
--------------------------------------------

Esto asegura que las apps monetizadas seguirán funcionando:

```powershell
ping 8.8.8.8
curl ifconfig.me
```
