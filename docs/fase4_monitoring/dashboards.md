# 📊 Fase 4 – Visualización de Datos con Dashboards de Grafana

## 🎯 Objetivo
Transformar los datos de métricas en bruto recolectados por Prometheus en **información visual, accionable e intuitiva** mediante la creación y personalización de dashboards en Grafana. El objetivo es tener una visión general de la salud de la red (`monitoring`) y la capacidad de investigar problemas específicos (`troubleshooting`).

---

## 🤔 El Poder de la Visualización

Una base de datos con millones de puntos de datos de series temporales es inútil sin una forma de interpretarla. Grafana es la herramienta que te permite:
- **Detectar Anomalías de un Vistazo:** Un pico repentino en un gráfico de CPU es mucho más fácil de detectar que un número en una tabla.
- **Correlacionar Datos:** Puedes superponer gráficos de diferentes métricas (ej: uso de CPU y latencia de red) para encontrar la causa raíz de un problema.
- **Comunicar el Estado:** Un dashboard claro es la mejor manera de mostrar el estado de la infraestructura a partes interesadas no técnicas.

--- 

## ✅ Paso 1: Importar un Dashboard Comunitario (El Camino Rápido)

La comunidad de Grafana ha creado miles de dashboards. Aprovecharemos este trabajo para tener una base sólida al instante.

1.  **Encontrar el Dashboard Adecuado:** El dashboard con ID `14694` es una excelente opción para el `windows_exporter`. Proporciona una visión muy completa del estado de un host Windows.

2.  **Proceso de Importación:**
    -   En Grafana, ve a **Dashboards > New > Import**.
    -   Pega el ID `14694` en el campo "Import via grafana.com" y haz clic en **Load**.
    -   **Punto Clave:** En la parte inferior de las opciones de importación, Grafana te pedirá que selecciones la fuente de datos de Prometheus. Asegúrate de elegir el nombre de la fuente de datos que creaste en el paso anterior.
    -   Haz clic en **Import**.

3.  **Explorar el Dashboard:**
    -   Inmediatamente tendrás un dashboard operativo. En la parte superior, verás un menú desplegable llamado `instance` o `host`. Úsalo para cambiar la vista entre los diferentes MiniPCs de tu red.
    -   Dedica tiempo a explorar los diferentes paneles para familiarizarte con las métricas que `windows_exporter` proporciona.

--- 

## ✅ Paso 2: Crear un Dashboard Personalizado (La Vista del Gestor)

El dashboard importado es excelente para ver un solo nodo, pero tú necesitas una **vista de flota**, un resumen de alto nivel de toda tu red. Crearemos un dashboard para esto.

1.  **Crear un Nuevo Dashboard:** Ve a **Dashboards > New > New Dashboard**.

2.  **Añadir un Panel de "Estado General":**
    -   **+ Add visualization > Stat**
    -   **Consulta (Query):** `count(up{job="windows_nodes"})`
    -   **Título del Panel:** `Nodos Activos`
    -   Este panel mostrará un número grande y claro: la cantidad de nodos que Prometheus está monitoreando activamente.

3.  **Añadir una Tabla de "Top 5 Nodos por CPU":**
    -   **+ Add visualization > Table**
    -   **Consulta (Query):** `topk(5, 100 - (avg by (instance) (rate(windows_cpu_time_total{mode="idle"}[2m])) * 100))`
    -   **Título del Panel:** `Top 5 - Uso de CPU`
    -   Esto te muestra instantáneamente qué nodos están trabajando más, un indicador clave para investigar problemas de rendimiento.

4.  **Añadir un Gráfico de "Espacio en Disco Crítico":**
    -   **+ Add visualization > Time series**
    -   **Consulta (Query):** `100 - (windows_logical_disk_free_bytes{volume=~"C:"} / windows_logical_disk_size_bytes{volume=~"C:"} * 100)`
    -   **Título del Panel:** `Uso de Disco C:`
    -   **Umbrales (Thresholds):** En las opciones del panel, puedes configurar umbrales para que el gráfico cambie de color (ej: a rojo cuando el uso supere el 85%).

5.  **Guardar el Dashboard:** Guarda tu nuevo dashboard con un nombre como `[Resumen de Flota] - Estado General`.

--- 

## ✅ Paso 3: Configurar Alertas Proactivas

El monitoreo pasivo (mirar dashboards) no es suficiente. Necesitas que el sistema te notifique *a ti* cuando algo va mal. 

1.  **Crear un Canal de Notificación:**
    -   Ve a **Alerting > Notification channels**.
    -   Configura un canal. Puede ser tu email, un canal de Slack, Telegram, etc. Sigue las instrucciones para el canal que elijas.

2.  **Crear una Regla de Alerta:**
    -   Vuelve a tu panel de "Uso de Disco C:".
    -   Ve a la pestaña **Alert** del panel.
    -   Crea una regla que se dispare **CUANDO** el `valor promedio` de la `consulta` en los `últimos 5 minutos` **ES SUPERIOR A** `85`.
    -   Asigna la alerta para que envíe notificaciones a tu canal recién creado.

Ahora, si el disco C: de cualquier nodo supera el 85% de uso durante 5 minutos, recibirás una notificación automática sin necesidad de estar mirando el dashboard.

--- 

## ✅ Checklist de Validación

- [ ] El dashboard comunitario para Windows ha sido importado y muestra datos correctamente.
- [ ] Se ha creado un dashboard personalizado de "Resumen de Flota".
- [ ] El panel de "Nodos Activos" muestra el número correcto de nodos.
- [ ] Los paneles de "Top CPU" y "Uso de Disco" muestran datos coherentes.
- [ ] Se ha configurado al menos un canal de notificación.
- [ ] Se ha creado una regla de alerta funcional para el espacio en disco.