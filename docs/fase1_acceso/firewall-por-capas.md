# 🛡️ Arquitectura de Seguridad: Firewall por Capas

## 🎯 Objetivo
Comprender el concepto de **Defensa en Profundidad (Defense in Depth)** aplicado a la seguridad de red. Específicamente, diferenciar entre el firewall que gestiona tu proveedor de Internet (ISP) y el firewall que tú gestionas en cada MiniPC con Windows, y entender cómo ambos contribuyen a una postura de seguridad robusta.

---

## 🤔 ¿Qué es la Defensa en Profundidad?

La Defensa en Profundidad es una estrategia de ciberseguridad que no confía en una única barrera. En su lugar, implementa **múltiples capas de controles de seguridad**, asumiendo que una capa podría fallar. Si un atacante logra superar una barrera, se encontrará con otra, y luego otra.

En tu arquitectura, tienes al menos dos capas de firewall fundamentales.

![Diagrama de Firewall por Capas](https://i.imgur.com/3oXgqB6.png)

--- 

## 🧱 Capa 1: El Firewall del ISP (La Muralla Exterior)

- **¿Qué es?** Es un dispositivo físico (generalmente parte de un router de borde o un equipo de seguridad dedicado) que se encuentra en la red de tu proveedor de Internet. Gestiona todo el tráfico que entra y sale del bloque de IPs públicas que te han asignado.
- **¿Quién lo controla?** El ISP. Tú, como cliente, puedes solicitar cambios en su configuración.
- **¿Qué controla?** El tráfico a nivel de red. Puede bloquear puertos, protocolos o IPs de origen **antes de que el tráfico llegue siquiera a tus MiniPCs**.

**Analogía:** Es la muralla que rodea tu castillo. Detiene a los ejércitos invasores en la frontera, lejos de tu puerta principal.

**Acción Recomendada:**
Aunque no es estrictamente necesario si confías en tu firewall de Windows, la mejor práctica es pedirle al ISP que aplique una regla en su extremo. Puedes enviarles un ticket de soporte diciendo:

> *"Por favor, para nuestro bloque de IPs `50.190.105.80/28`, restrinjan todo el tráfico entrante excepto el tráfico TCP en el puerto 22 que se origine desde nuestra IP de gestión `TU.IP.PUBLICA.COLOMBIA`."*

**Ventajas de esta capa:**
- **Reduce el "ruido":** Filtra la gran mayoría de escaneos y ataques automatizados, por lo que ni siquiera aparecerán en los logs de tus MiniPCs.
- **Protección Centralizada:** Una sola regla en el ISP protege a todos tus nodos a la vez.
- **Seguridad Adicional:** Si alguien comete un error y deshabilita el firewall en un MiniPC, esta capa externa seguiría protegiéndolo.

--- 

## 🖥️ Capa 2: El Firewall de Windows (La Puerta Blindada)

- **¿Qué es?** Es un componente de software que se ejecuta directamente en el sistema operativo de cada MiniPC.
- **¿Quién lo controla?** Tú, a través de la interfaz de Windows o, preferiblemente, de forma remota con PowerShell y Ansible.
- **¿Qué controla?** El tráfico a nivel de host. Decide qué conexiones se le permite hacer a una aplicación específica o qué puertos puede abrir el sistema operativo.

**Analogía:** Es la puerta blindada de tu habitación dentro del castillo. Aunque alguien haya saltado la muralla, todavía no puede entrar a tus aposentos.

**Acción Requerida:**
Esta capa es **obligatoria** y está bajo tu control total. Es aquí donde implementas las reglas detalladas que creamos en la sección anterior, permitiendo el acceso SSH solo desde tu IP.

**Ventajas de esta capa:**
- **Control Granular:** Puedes crear reglas muy específicas por aplicación o servicio.
- **Independencia:** No dependes del tiempo de respuesta del ISP para hacer un cambio de seguridad urgente.
- **Portabilidad:** Si mueves el MiniPC a otra red, las reglas del firewall de Windows se mueven con él.

--- 

## 🧠 Conclusión Estratégica

Nunca confíes en una sola capa de seguridad. La combinación del **firewall del ISP** y el **firewall de Windows** crea una defensa en profundidad robusta.

- El **firewall del ISP** actúa como un filtro grueso, eliminando la mayor parte del tráfico malicioso.
- El **firewall de Windows** actúa como un control fino y específico, asegurando que solo las conexiones autorizadas lleguen a los servicios que se ejecutan en el nodo.

Al implementar ambas, construyes una infraestructura significativamente más resiliente y segura, lo que es un argumento de venta clave para tu servicio de gestión profesional.