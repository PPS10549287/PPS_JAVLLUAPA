# Práctica 1: Hardening de Apache (Capa Base)

### 1. Explicación
En esta capa base, se ha configurado un servidor Apache sobre **Debian Bullseye** aplicando medidas iniciales de endurecimiento (*hardening*). El objetivo es reducir la superficie de exposición y mejorar la seguridad de las comunicaciones.

**Medidas implementadas:**
* **Ocultación de Servidor:** Se configuró `ServerTokens ProductOnly` y `ServerSignature Off`.
* **HSTS (HTTP Strict Transport Security):** Implementación de la cabecera para obligar al navegador a usar HTTPS durante 2 años.
> [!NOTE]
> Aunque la cabecera está configurada en esta fase, su efectividad real se completará en la **Práctica 5 (Generar un certificado digital)**, una vez que el servidor cuente con un certificado SSL/TLS válido para establecer el canal cifrado.
* **CSP (Content-Security-Policy):** Capa de seguridad para prevenir ataques de inyección de datos y XSS.
* **Deshabilitación de Autoindex:** Se desactivó el listado automático de directorios.
* **Seguridad SSL/TLS:** Habilitación del `módulo SSL` y activación de `certificados *snakeoil*`.

### 2. Guía de Despliegue
Este repositorio utiliza una imagen preconfigurada alojada en Docker Hub. No necesitamos los archivos de configuración locales para lanzarlo, ya que el archivo `csp_hsts.conf` y el resto de ajustes están integrados en la imagen.

**Paso 1: Descargar la imagen**

`docker pull javi2332/pps_p1_javlluapa:latest`

**Paso 2: Lanzar el contenedor**
Mapeamos el puerto 8080 para HTTP y el 8081 para HTTPS (puerto 443 interno):

`docker run -d --name harden_base -p 8080:80 -p 8081:443 javi2332/pps_p1_javlluapa:latest`

### 3: Validación y Auditoría**

Para verificar que todas las medidas de seguridad se han aplicado correctamente, realizamos peticiones al contenedor:

Verificación de Ocultación y Cabeceras (HTTP y HTTPS)

# Comprobación de cabeceras en puerto 8080
`curl -I http://localhost:8080`

Resultado esperado:
<img width="1083" height="280" alt="image" src="https://github.com/user-attachments/assets/e0c20f23-efe3-440e-98f1-64f0a05a264b" />

*(En la respuesta se observa que el campo `Server` no revela la versión de Apache ni del SO, y se verifica la correcta implementación de la política `Content-Security-Policy`).*

# Comprobación de puerto seguro (HSTS) en puerto 8081
Nota: Usamos -k porque los certificados son autofirmados.

`curl -Ik https://localhost:8081`

Resultado esperado:
<img width="1083" height="280" alt="image" src="https://github.com/user-attachments/assets/f0371f0d-cf85-4e75-be64-ea6e586ef30c" />

*(La presencia de la cabecera `Strict-Transport-Security` confirma que el servidor obliga al navegador a mantener una conexión segura (HTTPS) durante el tiempo configurado).*

### 4. URL Docker Hub
`docker pull javi2332/pps_p1_javlluapa:latest`

### 5. Limpieza
Para detener y borrar el contenedor de prueba ejecutamos:

`docker stop harden_base`

`docker rm -f harden_base`

Resultado esperado:

<img width="492" height="95" alt="image" src="https://github.com/user-attachments/assets/01e90834-7bee-4a13-84a7-c01dd353f8a5" />

