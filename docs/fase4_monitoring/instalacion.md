# 📊 Fase 4 – Instalación del Stack de Monitoreo: Prometheus y Grafana

## 🎯 Objetivo
Desplegar la pila de monitoreo y observabilidad estándar de la industria, compuesta por **Prometheus** para la recolección y almacenamiento de métricas, y **Grafana** para la visualización y creación de dashboards. Esta instalación se realizará utilizando Docker para asegurar la portabilidad y facilidad de mantenimiento.

---

## 🤔 ¿Qué es un Stack de Monitoreo y por qué es vital?

Un stack de monitoreo es un conjunto de herramientas que trabajan juntas para darte visibilidad sobre la salud y el rendimiento de tu infraestructura. Sin él, estás operando "a ciegas".

| Herramienta | Función | Analogía |
| :--- | :--- | :--- |
| **Prometheus** | **El Motor de Recolección y Almacenamiento.** Es una base de datos de series temporales (TSDB) que, a intervalos regulares, "raspa" (scrapes) puntos de datos (métricas) de tus nodos a través de HTTP. | Es el **enfermero** que pasa cada 15 segundos por la habitación de cada paciente (nodo) y anota sus signos vitales (CPU, RAM, disco) en un historial clínico. |
| **Grafana** | **La Plataforma de Visualización.** Es una herramienta que se conecta a fuentes de datos como Prometheus y te permite construir dashboards interactivos con gráficos, medidores y alertas. | Es el **monitor de cabecera** del hospital que toma el historial clínico y lo convierte en gráficos de electrocardiograma y medidores de pulso que el doctor (tú) puede entender de un vistazo. |

--- 

## 🐳 Guía de Instalación con Docker Compose

Estos pasos se deben ejecutar en tu **servidor de gestión** (Ubuntu 22.04).

### 1. Estructura de Directorios

Una buena organización de los archivos de configuración es clave.

```bash
# Crear una estructura de directorios dedicada para el stack de monitoreo
sudo mkdir -p /opt/monitoring/prometheus
cd /opt/monitoring
```

### 2. Archivo de Orquestación `docker-compose.yml`

Crea un archivo `sudo nano docker-compose.yml` con el siguiente contenido:

```yaml
version: '3.7'

services:
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    restart: unless-stopped
    volumes:
      - ./prometheus:/etc/prometheus/      # Mapea la configuración local al contenedor
      - prometheus_data:/prometheus         # Persiste la base de datos de métricas
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.retention.time=1y' # Retener métricas por 1 año
    ports:
      - "9090:9090"

  grafana:
    image: grafana/grafana-oss:latest
    container_name: grafana
    restart: unless-stopped
    volumes:
      - grafana_data:/var/lib/grafana       # Persiste los dashboards y la configuración
    ports:
      - "3000:3000"

volumes:
  prometheus_data:
  grafana_data:
```
- **Volúmenes (`volumes`):** Son cruciales. Separan los datos (que son persistentes) de los contenedores (que son efímeros). Esto te permite actualizar o recrear los contenedores sin perder tu configuración o tus métricas.
- **Retención (`retention.time`):** Le hemos dicho a Prometheus que guarde los datos durante un año. El valor por defecto es de solo 15 días.

### 3. Archivo de Configuración de Prometheus

Crea un archivo `sudo nano prometheus/prometheus.yml`. Por ahora, solo se monitoreará a sí mismo para verificar que funciona.

```yaml
# /opt/monitoring/prometheus/prometheus.yml
global:
  scrape_interval: 15s # Frecuencia con la que se recolectan las métricas
  evaluation_interval: 15s # Frecuencia con la que se evalúan las reglas de alerta

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
```

### 4. Iniciar el Stack

Desde el directorio `/opt/monitoring`, ejecuta:
```bash
sudo docker-compose up -d
```

--- 

## ✅ Paso 5: Configuración Inicial de Grafana

1.  **Acceso y Cambio de Contraseña:**
    -   Abre tu navegador: `http://<IP_DE_TU_SERVIDOR>:3000`
    -   Inicia sesión con `admin` / `admin` y establece una nueva contraseña segura.

2.  **Conectar Grafana con Prometheus:**
    -   Ve a **Configuration (⚙️) > Data Sources**.
    -   Haz clic en **Add data source** y selecciona **Prometheus**.
    -   En el campo **URL**, escribe `http://prometheus:9090`.
        -   **¿Por qué `prometheus:9090`?** Docker Compose crea una red virtual para los servicios definidos en el archivo. Dentro de esta red, los contenedores pueden comunicarse entre sí usando su nombre de servicio como si fuera un nombre de host (DNS).
    -   Haz clic en **Save & test**. Deberías ver un mensaje verde de éxito.

--- 

## ✅ Checklist de Validación

- [ ] La estructura de directorios en `/opt/monitoring` está creada.
- [ ] Los contenedores de Prometheus y Grafana están en estado `Up`.
- [ ] La interfaz web de Prometheus es accesible en el puerto `9090`.
- [ ] La interfaz web de Grafana es accesible en el puerto `3000`.
- [ ] La contraseña de administrador de Grafana ha sido cambiada.
- [ ] La fuente de datos de Prometheus ha sido añadida y probada con éxito en Grafana.