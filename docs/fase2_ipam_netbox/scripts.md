# ⚙️ Fase 2 – Automatización del Registro de Nodos con la API de NetBox

## 🎯 Objetivo
Pasar del registro manual de dispositivos a un **proceso de registro automatizado** utilizando un script de Python que interactúa con la API de NetBox. Esto es fundamental para escalar la operación y asegurar la consistencia de los datos.

---

## 🤔 ¿Por qué usar la API?

La interfaz web de NetBox es excelente para visualizar datos y hacer cambios puntuales, pero es completamente ineficiente para tareas masivas. Si necesitas registrar 10, 50 o 250 nodos, el registro manual es:
- **Lento y Tedioso:** Horas de trabajo repetitivo.
- **Propenso a Errores:** Un simple error de tipeo puede causar problemas de configuración más adelante.
- **No Escalable:** Es el cuello de botella que te impedirá crecer.

La **API (Interfaz de Programación de Aplicaciones)** de NetBox expone toda su funcionalidad a programas externos. Al usarla, podemos decirle a NetBox qué hacer mediante código, lo que nos permite automatizar cualquier tarea.

--- 

## 🐍 `pynetbox`: Tu Puente entre Python y NetBox

Para facilitar la interacción con la API, usaremos `pynetbox`, una librería de Python que traduce funciones de Python sencillas en las complejas llamadas a la API que NetBox entiende.

### ✅ Paso 1: Preparar el Entorno

1.  **Instalar `pynetbox`:** En tu máquina de gestión, ejecuta:
    ```bash
    pip install pynetbox
    ```

2.  **Generar un Token de API en NetBox:** El token es una contraseña para que tu script se autentique.
    -   En NetBox, ve a tu perfil de usuario (arriba a la derecha) > **API Tokens**.
    -   Haz clic en **+ Add a token**.
    -   **IMPORTANTE:** Marca la casilla **"Write Enabled"**. Sin esto, el token solo podrá leer datos, no crearlos.
    -   Copia la clave del token y guárdala en tu gestor de contraseñas. **No la volverás a ver.**

--- 

## 📜 Script de Registro Masivo

Este script registrará los 9 nodos restantes del `cliente-a` (del 2 al 10). Está diseñado para ser fácilmente adaptable a otros clientes y rangos de IPs.

Crea un archivo llamado `registrar_nodos.py`:

```python
import pynetbox
import os

# --- Configuración Segura ---
# Lee las credenciales desde variables de entorno para no exponerlas en el código.
# En tu terminal, ejecuta:
# export NETBOX_URL='http://<IP_DE_TU_SERVIDOR_NETBOX>'
# export NETBOX_TOKEN='TU_TOKEN_DE_API_AQUI'

NETBOX_URL = os.getenv('NETBOX_URL')
NETBOX_TOKEN = os.getenv('NETBOX_TOKEN')

if not all([NETBOX_URL, NETBOX_TOKEN]):
    print("Error: Asegúrate de haber definido las variables de entorno NETBOX_URL y NETBOX_TOKEN.")
    exit()

# --- Constantes del Modelo de Datos ---
# Cambia estos valores para registrar nodos para un cliente diferente.
CLIENTE_TAG = 'cliente-a'
SITIO_SLUG = 'usa-mia-01'
DEVICE_ROLE_SLUG = 'nodo-cliente'
DEVICE_TYPE_SLUG = 'minipc-e2'
INTERFACE_NAME = 'eth0'
IP_RANGE_START = 2  # El número del primer nodo a registrar
IP_RANGE_END = 10   # El número del último nodo a registrar
IP_BASE = "50.190.105."

# --- Script Principal ---
def main():
    # Inicializar API
    nb = pynetbox.api(url=NETBOX_URL, token=NETBOX_TOKEN)

    # Obtener objetos base de NetBox usando sus "slugs" (identificadores para URL)
    try:
        site_obj = nb.dcim.sites.get(slug=SITIO_SLUG)
        role_obj = nb.dcim.device_roles.get(slug=DEVICE_ROLE_SLUG)
        type_obj = nb.dcim.device_types.get(slug=DEVICE_TYPE_SLUG)
        tag_obj = nb.extras.tags.get(slug=CLIENTE_TAG)
        print("✅ Objetos base de NetBox obtenidos correctamente.")
    except Exception as e:
        print(f"❌ Error al obtener objetos base de NetBox: {e}")
        return

    # Bucle para crear los nodos
    for i in range(IP_RANGE_START, IP_RANGE_END + 1):
        device_name = f'{CLIENTE_TAG}-nodo-{i:02d}'
        ip_address = f'{IP_BASE}{80 + i}/28'

        print(f"\n--- Procesando: {device_name} --- ")

        # Verificar si el dispositivo ya existe
        if nb.dcim.devices.get(name=device_name):
            print(f"🟡 El dispositivo '{device_name}' ya existe. Se omite.")
            continue

        # Crear el dispositivo
        try:
            new_device = nb.dcim.devices.create(
                name=device_name,
                device_role=role_obj.id,
                device_type=type_obj.id,
                site=site_obj.id,
                status='active',
                tags=[tag_obj.id]
            )
            print(f"  -> ✔️ Dispositivo '{new_device.name}' creado.")

            # Crear la interfaz y asignar la IP
            interface = nb.dcim.interfaces.create(device=new_device.id, name=INTERFACE_NAME, type='1000base-t')
            ip = nb.ipam.ip_addresses.create(address=ip_address, status='active', assigned_object_type='dcim.interface', assigned_object_id=interface.id)
            new_device.primary_ip4 = ip.id
            new_device.save()
            print(f"  -> ✔️ IP {ip.address} asignada a la interfaz {interface.name} y establecida como primaria.")

        except pynetbox.RequestError as e:
            print(f"  -> ❌ ERROR al crear {device_name}: {e.error}")
            continue

    print("\n🎉 Proceso de registro completado.")

if __name__ == "__main__":
    main()

```

### Ejecución del Script

1.  **Exporta las variables de entorno** en tu terminal para mantener tus credenciales seguras.
2.  Ejecuta el script: `python registrar_nodos.py`.
3.  Observa la salida y luego verifica en la interfaz de NetBox que los nuevos dispositivos han sido creados, etiquetados y tienen sus IPs asignadas.

--- 

## ✅ Checklist de Validación

- [ ] La librería `pynetbox` está instalada.
- [ ] Se ha generado un token de API con permisos de escritura.
- [ ] Las variables de entorno `NETBOX_URL` y `NETBOX_TOKEN` están configuradas.
- [ ] El script se ejecuta sin errores de conexión o de lógica.
- [ ] Los 9 nodos nuevos aparecen en NetBox, cada uno con su nombre, IP, rol, tipo, sitio y tag correctos.
- [ ] Al volver a ejecutar el script, este omite los dispositivos que ya existen en lugar de dar un error.