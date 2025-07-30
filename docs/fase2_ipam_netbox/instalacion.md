# 🌐 Fase 2 – Instalación de NetBox con Docker

## 🎯 Objetivo
Instalar y configurar **NetBox**, la herramienta que servirá como la **Única Fuente de Verdad (Single Source of Truth - SSoT)** para toda tu infraestructura. Utilizaremos Docker y Docker Compose para un despliegue rápido, aislado y mantenible.

---

## 🤔 ¿Qué es NetBox y por qué es indispensable?

NetBox es una aplicación de modelado de infraestructura. Piensa en ella como una base de datos superpoderosa diseñada específicamente para redes. Te permite documentar y gestionar los activos más importantes de tu operación:

- **IPAM (Gestión de Direcciones IP):** Control total sobre tus prefijos (`50.190.105.80/28`), IPs individuales, VLANs, etc. Sabrás qué IP está asignada, a qué dispositivo y para qué cliente, en todo momento.
- **DCIM (Gestión de Infraestructura de Centro de Datos):** Aunque no uses un data center tradicional, puedes modelar tus **Sitios** (ej: `USA-MIA-01`), **Fabricantes** (`Beelink`), **Tipos de Dispositivos** (`MiniPC E2`), y cada **Dispositivo** individualmente.

**Beneficios Clave para tu Negocio:**
- **Elimina el Caos:** Reemplaza hojas de cálculo frágiles y propensas a errores. NetBox impone una estructura.
- **Profesionalismo:** Demuestra a tus clientes que utilizas herramientas estándar de la industria para gestionar su infraestructura.
- **Habilita la Automatización:** Este es el beneficio más importante. Tu herramienta de automatización (Ansible) consultará la API de NetBox para saber qué dispositivos existen y cómo debe configurarlos. Sin NetBox, la automatización a escala es imposible.

--- 

## 🐳 ¿Por qué usar Docker para la Instalación?

Podrías instalar NetBox y sus dependencias (PostgreSQL, Redis, etc.) directamente en el sistema operativo, pero esto es frágil y difícil de mantener. Docker nos permite empaquetar cada servicio en su propio **contenedor** aislado.

- **Aislamiento:** Las dependencias de NetBox no entrarán en conflicto con otro software que instales en el servidor.
- **Portabilidad:** Puedes mover tu instalación de NetBox a otro servidor simplemente moviendo los archivos de configuración de Docker y los volúmenes de datos.
- **Facilidad de Actualización:** Actualizar NetBox a una nueva versión es tan simple como cambiar la etiqueta de la imagen en el archivo de configuración y reiniciar los contenedores.

--- 

## 🛠️ Guía de Instalación Paso a Paso

Estos pasos se deben ejecutar en tu **servidor de gestión** (un VPS con Ubuntu 22.04 es ideal).

### 1. Preparar el Servidor

Conéctate a tu servidor Ubuntu por SSH y asegúrate de que todo esté actualizado:
```bash
sudo apt update && sudo apt upgrade -y
```

Instala Git, Docker y Docker Compose:
```bash
sudo apt install -y git docker.io docker-compose
```

### 2. Descargar la Configuración de NetBox Docker

El equipo de NetBox mantiene un repositorio de GitHub con toda la configuración de Docker lista para usar.

```bash
# Clona el repositorio en el directorio /opt, un buen lugar para software de terceros
sudo git clone -b release https://github.com/netbox-community/netbox-docker.git /opt/netbox-docker
cd /opt/netbox-docker
```

### 3. Configurar el Entorno de NetBox

Copia el archivo de configuración de ejemplo y luego genera una clave secreta, que NetBox usa para la seguridad interna de la aplicación.

```bash
# Copia el archivo de variables de entorno
sudo cp env/netbox.env-example env/netbox.env

# Genera una clave secreta y guárdala
SECRET_KEY=$(python3 contrib/generate-secret-key.py)

# Añade la clave al final del archivo de configuración
echo "SECRET_KEY=$SECRET_KEY" | sudo tee -a env/netbox.env
```

### 4. Iniciar NetBox

Con la configuración lista, levanta toda la pila de servicios con un solo comando:

```bash
sudo docker-compose up -d
```
- `up`: Crea e inicia los contenedores.
- `-d` (detach): Ejecuta los contenedores en segundo plano.

La primera vez, Docker descargará todas las imágenes necesarias (NetBox, PostgreSQL, Redis), lo que puede tardar unos minutos.

### 5. Acceder y Proteger tu Instancia de NetBox

Una vez que el comando anterior termine, NetBox estará accesible.

1.  **Accede a la Interfaz Web:** Abre tu navegador y ve a `http://<IP_DE_TU_SERVIDOR_NETBOX>`.
    - *Nota: La configuración por defecto ahora usa el puerto 80, no el 8000.*
2.  **Primer Inicio de Sesión:**
    -   **Usuario:** `admin`
    -   **Contraseña:** `admin`
3.  **CAMBIA LA CONTRASEÑA INMEDIATAMENTE:**
    -   Ve a **Admin > Users > admin**.
    -   Haz clic en el botón **"Change Password"** y establece una contraseña fuerte y única.

--- 

## ✅ Checklist de Validación

- [ ] El servidor de gestión está actualizado y tiene Docker y Docker Compose instalados.
- [ ] El repositorio `netbox-docker` está clonado en `/opt/netbox-docker`.
- [ ] El archivo `env/netbox.env` existe y contiene una `SECRET_KEY`.
- [ ] El comando `sudo docker-compose ps` muestra todos los contenedores de NetBox en estado `Up` o `running`.
- [ ] Puedes acceder a la interfaz web de NetBox desde tu navegador.
- [ ] Has iniciado sesión y has cambiado la contraseña por defecto del usuario `admin`.