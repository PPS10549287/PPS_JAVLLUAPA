# Práctica Final: Golden Image Apache (Hardening Nivel Industrial)

## 1. Introducción
Esta imagen representa la culminación del proyecto de seguridad en servidores web. Se ha diseñado como una **Golden Image**, una imagen de referencia que consolida todas las capas de seguridad configuradas en las prácticas anteriores, sumando protecciones críticas a nivel de sistema operativo, usuario y protocolo.

La imagen sigue una **estratia de herencia en cascada**, garantizando que este contenedor final sea el más robusto de toda la serie:
* **P1 a P4:** Hardening básico, WAF (ModSecurity), Reglas OWASP y protección Anti-DoS (Mod_Evasive).
* **P5:** Cifrado SSL/TLS y activación real de la política **HSTS**.
* **GOLDEN IMAGE (Final):** Hardening de sistema, gestión de usuarios y blindaje de protocolos.

## 2. Justificación Técnica de la Implementación

Para esta fase final, se ha optado por un enfoque de **"Cirugía y Modularidad"** en lugar de sustitución completa:

### A. Uso de Archivo de Configuración Externo (`a2enconf`)
En lugar de inyectar decenas de líneas mediante comandos `echo` en el archivo principal, se ha creado el archivo `geekflare-hardening.conf`. 
* **Razón:** Esto permite mantener la herencia de las prácticas anteriores intacta. Al usar `a2enconf`, Apache carga estas nuevas reglas de seguridad de forma modular, permitiendo una auditoría más clara y evitando errores de sintaxis en el archivo maestro `apache2.conf`.

### B. Uso de `sed` para Variables de Entorno
Se ha utilizado el comando `sed` exclusivamente para modificar el archivo `/etc/apache2/envvars`.
* **Razón:** En distribuciones basadas en Debian, el usuario que ejecuta Apache se define en este archivo. El uso de `sed` es la forma más eficiente de realizar este cambio de identidad de forma persistente sin tener que sobrescribir el archivo completo del sistema.

### C. Contenido del Dockerfile
> [!IMPORTANT]
> <img width="984" height="550" alt="image" src="https://github.com/user-attachments/assets/e0f3d169-53e3-4029-86c8-2a2f8534a2ec" />

## 3. El Archivo Externo: `geekflare-hardening.conf`
Este archivo actúa como el "escudo final" del servidor. Su función es centralizar las directivas de seguridad avanzada que no se cubrieron en fases previas:

* **Hardening de Protocolo:** Incluye `TraceEnable Off` y `FileETag None` para evitar ataques de Cross-Site Tracing y fugas de información del inodo del sistema.
* **Gestión de Timeouts:** Reduce el `Timeout` a 60 segundos para mitigar ataques de denegación de servicio de conexión lenta (Slow Loris).
* **Blindaje de Directorios y Métodos:** * Desactiva el listado de directorios (`-Indexes`) y las inclusiones del lado del servidor (`-Includes`).
    * Implementa `<LimitExcept>`, que actúa como una lista blanca permitiendo solo los métodos `GET`, `POST` y `HEAD`, denegando cualquier otro intento (como `PUT` o `DELETE`).
* **Forzado de HTTP 1.1:** Utiliza el motor de reescritura para rechazar peticiones que utilicen el protocolo HTTP 1.0, eliminando riesgos de seguridad heredados.
* **Seguridad de Capa de Aplicación:** Define las cabeceras `X-Frame-Options` y `X-XSS-Protection` para proteger al cliente final.

> [!IMPORTANT]
> <img width="729" height="679" alt="image" src="https://github.com/user-attachments/assets/18aba9a7-51a2-47f5-a568-9a12704b97ee" />


## 4. Mejoras de Seguridad Implementadas (Capa Final)
* **Usuario sin privilegios:** El servicio Apache se ejecuta bajo el usuario **apache**, aislándolo de `root`.
* **Hardening de Permisos:** Aplicación de permisos restrictivos (`750`) en directorios de configuración.
* **Ocultación de Identidad:** Forzado de `ServerTokens Prod` para enmascarar la infraestructura.

### 5. Guía de Despliegue

**Paso 1: Descargar la imagen**

`docker pull pps10549287/pps-pr-gold:latest`

**Paso 2: Lanzar el contenedor**

`docker run -d --name pps-pr-gold-javlluapa -p 8080:80 -p 8081:443 pps10549287/pps-pr-gold:latest`

**Paso 3: Visualizar contenedor activo**
Comprobamos que el contenedor se encuentra corriendo y está operativo:

`docker ps`

## 6. Validación de la Seguridad (Evidencias)

### A. Prueba Maestra de Herencia y Prioridad de Capas
Para validar que la Gold Image respeta la herencia de las prácticas anteriores y aplica correctamente el endurecimiento final, realizamos una búsqueda recursiva de directivas críticas en todo el árbol de configuración de Apache.

**Comando:** `docker exec pps-pr-gold-javlluapa grep -rE "ServerTokens|ServerSignature|FileETag|TraceEnable" /etc/apache2/`

> [!IMPORTANT]
> **Captura de evidencia (Capas):**
> <img width="1291" height="318" alt="image" src="https://github.com/user-attachments/assets/acd30926-723a-48ff-bd42-0783764a58c3" />

> [!NOTE]
> **Interpretación:** El resultado del escaneo confirma la arquitectura de Defensa en Profundidad mediante la coexistencia de capas:

Capa de Herencia (P1): Se detectan las directivas en security-hardened.conf (ServerTokens ProductOnly y ServerSignature Off), demostrando que el endurecimiento inicial persiste tras todas las fases del proyecto.

Capa Gold Image: Se observan las nuevas restricciones en gold-image-hardening.conf (FileETag None y TraceEnable Off), inyectadas específicamente en esta fase final.

Prioridad de Carga: Al utilizar archivos .conf dentro de conf-available, Apache aplica los valores más restrictivos de las capas superiores sobre los valores por defecto del sistema, garantizando un servidor totalmente "mudo" y blindado.

### B. Verificación de Identidad y Cabeceras Globales
Realizamos una petición HTTPS para validar el stack completo de seguridad desde el punto de vista del cliente.

**Comando:** `curl -I -k https://www.midominioseguro.com:8081`

> [!IMPORTANT]
> **Captura de evidencia (Cabeceras):**
> <img width="1087" height="294" alt="image" src="https://github.com/user-attachments/assets/e0c6196a-8f1e-42af-a8ef-3101b1b1c51b" />

> [!NOTE]
> **Interpretación:** El servidor responde con el banner oculto (`Server: Apache`), el mecanismo de transporte seguro activo (`Strict-Transport-Security`) y las nuevas protecciones contra Clickjacking y XSS. Se confirma que el servidor es "mudo" ante intentos de reconocimiento de versión.

### C. Verificación de Usuario No Privilegiado
Auditamos los procesos en ejecución para asegurar que el compromiso de un hilo de ejecución no otorgue acceso de superusuario al atacante.

**Comando:** `docker exec pps-pr-gold-javlluapa ps -ef | grep apache`

> [!IMPORTANT]
> **Captura de evidencia (Procesos):**
> <img width="956" height="125" alt="image" src="https://github.com/user-attachments/assets/020215c4-c64f-4edb-9415-bfbe08cc5681" />

> [!NOTE]
> **Interpretación:** Se observa cómo el proceso padre corre como `root` para gestionar los sockets de red, mientras que los procesos trabajadores (workers) que procesan el tráfico externo han cambiado su identidad al usuario **apache**, cumpliendo el principio de mínimo privilegio.

### D. Persistencia de la Herencia (WAF y SSL)
Validamos que las defensas activas de las prácticas anteriores (ModSecurity) siguen operando correctamente tras el endurecimiento del sistema.

**Comando (Simulación de ataque):**
`curl -I -k "https://www.midominioseguro.com:8081/?exec=/bin/bash"`

> [!IMPORTANT]
> **Captura de evidencia (Persistencia):**
> <img width="865" height="161" alt="image" src="https://github.com/user-attachments/assets/4ad1d5a8-8e99-4d1e-9c0c-02fdf3b2fa09" />

> [!NOTE]
> **Interpretación:** El servidor devuelve un **403 Forbidden**. Esto confirma que el motor ModSecurity (P2) y las reglas OWASP (P3) siguen inspeccionando el tráfico una vez descifrado por la capa SSL (P5), bloqueando el intento de intrusión.


## 6. URL Docker Hub (Golden Image)
`docker pull pps10549287/pps-pr-gold:latest`

## 7. Conclusión
Este proyecto demuestra una arquitectura de **Defensa en Profundidad** real. Cada práctica ha añadido una capa de blindaje que la Golden Image final ha consolidado, resultando en un servidor Apache optimizado para resistir ataques modernos.
