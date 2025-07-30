# 🧱 Fase 2 – Modelado de la Infraestructura en NetBox

## 🎯 Objetivo
Crear el **modelo de datos fundamental** dentro de NetBox que representará fielmente tu infraestructura física y lógica. Este es un paso de diseño crítico: un buen modelo hace que la gestión y la automatización sean intuitivas; un mal modelo crea confusión y deuda técnica.

---

## 🤔 ¿Qué es el Modelado de Datos?

Modelar es el proceso de crear las **plantillas** y **categorías** antes de registrar los activos individuales. Es como diseñar los estantes y las etiquetas de un almacén antes de empezar a meter la mercancía. Si lo haces bien, encontrarás todo fácilmente.

En NetBox, nos enfocaremos en los siguientes modelos clave:

- **Organización:** ¿Quién es el fabricante? ¿Dónde está ubicado el equipo?
- **Dispositivos:** ¿Qué tipo de hardware es? ¿Qué función cumple en la red?
- **IPAM:** ¿A qué bloque de IPs pertenece esta dirección?

--- 

## 🛠️ Guía de Modelado Paso a Paso

Estos pasos se realizan en la interfaz web de NetBox.

### ✅ Paso 1: Modelar el Mundo Físico (Organization)

Primero, definimos el "quién" y el "dónde".

1.  **Crear el Fabricante (Manufacturer):** Esto agrupa los dispositivos por su marca.
    -   **Ruta:** `Organization > Manufacturers > + Add`
    -   **Name:** `Beelink`
    -   **Descripción:** `Fabricante de MiniPCs para nodos cliente.`

2.  **Crear el Sitio (Site):** Un sitio es una ubicación geográfica única.
    -   **Ruta:** `Organization > Sites > + Add`
    -   **Name:** `USA-MIA-01` (Un código nemotécnico: País-Ciudad-NúmeroDeSitio)
    -   **Status:** `Active`
    -   **Time Zone:** `America/New_York`
    -   **Descripción:** `Ubicación principal para los nodos del Cliente A en Miami.`

### ✅ Paso 2: Modelar las Plantillas de Dispositivos (Devices)

Ahora, definimos el "qué" y el "porqué".

1.  **Crear el Rol del Dispositivo (Device Role):** Define la función lógica de un equipo.
    -   **Ruta:** `Devices > Device Roles > + Add`
    -   **Name:** `Nodo Cliente`
    -   **Color:** `Green` (Elige colores para identificar roles de un vistazo)
    -   **Descripción:** `Un MiniPC gestionado que ejecuta cargas de trabajo para un cliente.`
    -   *Crea también un rol llamado `Servidor de Gestión` (color `Blue`) para tus propios servidores.* 

2.  **Crear el Tipo de Dispositivo (Device Type):** Esta es la plantilla de hardware.
    -   **Ruta:** `Devices > Device Types > + Add`
    -   **Manufacturer:** `Beelink`
    -   **Model:** `MiniPC E2`
    -   **Height (U):** `1` (Aunque no van en un rack, es una buena práctica)
    -   **Descripción:** `Modelo estándar de MiniPC para los nodos cliente.`
    -   *Consejo: Puedes añadir aquí las interfaces de red (`eth0`, etc.) para que cada nuevo dispositivo de este tipo se cree ya con sus interfaces listas.* 

### ✅ Paso 3: Modelar el Espacio de Red (IPAM)

Finalmente, definimos el "dónde" en la red.

1.  **Crear el Prefijo (Prefix):** Un prefijo es un bloque de direcciones IP.
    -   **Ruta:** `IPAM > Prefixes > + Add`
    -   **Prefix:** `50.190.105.80/28`
    -   **Status:** `Active`
    -   **Site:** `USA-MIA-01`
    -   **Description:** `Bloque de IPs públicas fijas de Comcast para nodos cliente en Miami.`

--- 

## 🚀 Registro de tu Primer Dispositivo Real

Con el modelo ya creado, registrar un nuevo dispositivo es un proceso de "rellenar los huecos" de las plantillas que has diseñado.

1.  **Crear el Dispositivo:**
    -   **Ruta:** `Devices > Devices > + Add`
    -   **Name:** `cliente-a-nodo-01` (Un nombre de host descriptivo)
    -   **Device Role:** `Nodo Cliente`
    -   **Device Type:** `MiniPC E2`
    -   **Site:** `USA-MIA-01`
    -   **Status:** `Active`

2.  **Crear y Asignar la Interfaz de Red:**
    -   Dentro de la página del nuevo dispositivo, ve a la pestaña **Interfaces** y haz clic en **+ Add**.
    -   **Name:** `eth0`
    -   **Type:** `1000BASE-T (1GE)`

3.  **Asignar la Dirección IP:**
    -   En la misma página de la interfaz, busca la sección **IP Addresses** y haz clic en **+ Assign IP Address**.
    -   **Address:** `50.190.105.81/28` (Escribe la IP y el prefijo).
    -   Marca la casilla **"Make this the primary IP for the device"**.

--- 

## ✅ Checklist de Validación

- [ ] El modelo de datos refleja la realidad de tu infraestructura (Fabricantes, Sitios, Roles, Tipos).
- [ ] El prefijo IP está correctamente definido y asociado a un sitio.
- [ ] Has registrado tu primer dispositivo físico, asociándolo a los modelos creados.
- [ ] La IP primaria del dispositivo está correctamente asignada a su interfaz de red.
- [ ] La estructura se siente lógica y escalable. Puedes imaginar cómo registrarías 100 dispositivos más usando este modelo.