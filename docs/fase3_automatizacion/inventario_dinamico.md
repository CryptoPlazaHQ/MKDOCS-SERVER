# ⚙️ Fase 3 – Inventario Dinámico desde NetBox

## 🎯 Objetivo
Evolucionar de un inventario estático (un archivo de texto que debes mantener a mano) a un **inventario dinámico**. Esta configuración le permite a Ansible consultar a NetBox, nuestra fuente única de verdad, en tiempo real cada vez que se ejecuta. Esto es la clave para una automatización verdaderamente escalable y libre de errores.

---

## 🤔 ¿Por qué es un Salto Cualitativo?

| Inventario Estático (`.ini`) | Inventario Dinámico (NetBox) |
| :--- | :--- |
| **Frágil:** Propenso a errores de tipeo y a quedar desactualizado. | **Robusto:** Los datos provienen de una fuente centralizada y estructurada. |
| **Manual:** Debes editar el archivo cada vez que añades o quitas un nodo. | **Automático:** Si un nodo se añade a NetBox, Ansible lo ve. Si se elimina, desaparece. |
| **Limitado:** La agrupación de hosts es básica. | **Inteligente:** Puedes agrupar hosts basándote en cualquier dato de NetBox: sitio, rol, fabricante, tag, tenant, etc. |

--- 

## ✅ Paso 1: Instalar el Plugin de Inventario de NetBox

Ansible se integra con otras herramientas a través de **colecciones**, que son paquetes de plugins y módulos. Necesitamos instalar la colección oficial de NetBox.

```bash
# Asegúrate de que tu entorno virtual de Ansible esté activado
ansible-galaxy collection install netbox.netbox
```
- `ansible-galaxy`: Es el comando de Ansible para gestionar colecciones y roles de la comunidad.

--- 

## ✅ Paso 2: Configurar el Plugin de Inventario

Ahora, en lugar de un archivo `.ini`, crearemos un archivo `.yml` que le dice a Ansible cómo usar el plugin de NetBox. Por convención, se suele nombrar según el entorno (ej: `produccion.netbox.yml`).

Crea un archivo llamado `produccion.netbox.yml`:

```yaml
# Fichero de configuración para el inventario dinámico de NetBox
plugin: netbox.netbox.nb_inventory

# --- Conexión a la API de NetBox ---
api_endpoint: http://<IP_DE_TU_SERVIDOR_NETBOX>
token: "{{ lookup('env', 'NETBOX_TOKEN') }}" # Lee el token de una variable de entorno
validate_certs: false # Cambiar a 'true' si usas un certificado SSL válido

# --- Mapeo de Datos --- 
# Le decimos a Ansible cómo construir el inventario a partir de los datos de NetBox

# Queremos usar el nombre del dispositivo de NetBox como el nombre en el inventario
hostname_preference:
  - name

# --- Agrupación Inteligente ---
# Aquí está la magia: creamos grupos de Ansible basados en los atributos de NetBox
group_by:
  - device_roles  # Crea un grupo por cada Rol (ej: grupo "Nodo_Cliente")
  - sites         # Crea un grupo por cada Sitio (ej: grupo "USA-MIA-01")
  - tags          # Crea un grupo por cada Tag (ej: grupo "tag_cliente_a")
  - tenants       # Crea un grupo por cada Cliente/Tenant

# --- Composición de Variables ---
# Podemos crear variables para nuestros hosts usando datos de NetBox
compose:
  # Definimos la variable ansible_host para que apunte a la IP primaria del dispositivo
  ansible_host: primary_ip4.address | split('/')[0]
```

**Análisis de la Configuración:**
- `token`: Usamos `lookup('env', 'NETBOX_TOKEN')` para leer el token de una variable de entorno. Esto es más seguro que ponerlo directamente en el archivo.
- `group_by`: Le damos a Ansible una lista de atributos de NetBox. Creará automáticamente un grupo por cada valor único de esos atributos.
- `compose`: Esta sección es muy poderosa. Nos permite construir variables de Ansible. Aquí, estamos creando la variable `ansible_host` (que le dice a Ansible a qué IP conectarse) extrayendo la dirección IP primaria del dispositivo desde NetBox.

--- 

## ✅ Paso 3: Probar el Inventario Dinámico

Antes de usarlo, vamos a pedirle a Ansible que nos muestre el inventario que construiría con esta configuración.

1.  **Exporta tu token como variable de entorno:**
    ```bash
    export NETBOX_TOKEN='TU_TOKEN_DE_API_AQUI'
    ```
2.  **Ejecuta `ansible-inventory`:**
    ```bash
    ansible-inventory -i produccion.netbox.yml --graph
    ```
    - Deberías ver una estructura de árbol (`--graph`) que muestra cómo Ansible ha agrupado tus 10 nodos por rol, sitio y cualquier otro `group_by` que hayas definido.

--- 

## ✅ Paso 4: Usar el Inventario Dinámico en un Playbook

Ahora, simplemente reemplaza el archivo de inventario estático por el dinámico al ejecutar tu playbook.

```bash
# Ejecuta el mismo playbook de antes, pero con el nuevo inventario
ansible-playbook -i produccion.netbox.yml check_reachability.yml
```

El playbook ahora se ejecutará sobre el grupo `windows_nodes` (o `Nodo_Cliente`, si ajustas el `hosts:` en el playbook), pero la fuente de esa lista de servidores es ahora, y para siempre, NetBox.

**¡Has desacoplado tu automatización de una lista manual y la has conectado a tu fuente de verdad!**