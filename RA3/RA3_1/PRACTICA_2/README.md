# Práctica 2: ModSecurity (WAF Base)

### 1. Explicación
Esta imagen hereda de la **P1 (Hardening Base)** y añade una capa de defensa activa mediante un **WAF (Web Application Firewall)**:

* **Estrategia en cascada:** Se utiliza la imagen `javi2332/pps_p1_javlluapa` como base, manteniendo CSP, HSTS y SSL.
* **ModSecurity:** Instalación y activación del motor en modo bloqueo (`SecRuleEngine On`).
* **Protección Activa:** El servidor ahora es capaz de interceptar peticiones maliciosas antes de que lleguen a la aplicación.

### 2. Guía de Despliegue
Este repositorio utiliza una imagen preconfigurada alojada en Docker Hub. Al heredar de la P1, los ajustes de endurecimiento y el firewall ya están integrados en la imagen.

**Paso 1: Descargar la imagen**

`docker pull javi2332/pps_p2_javlluapa:latest`

**Paso 2: Lanzar el contenedor**
Mapeamos el puerto 8080 para HTTP y el 8081 para HTTPS (puerto 443 interno):

`docker run -d --name pps_p2_waf -p 8080:80 -p 8081:443 javi2332/pps_p2_javlluapa:latest`

### 3. Validación y Auditoría

Para verificar que el WAF está operativo y bloqueando amenazas, realizamos una prueba de ataque simulado:

**Comprobación de bloqueo (Path Traversal)**

`curl -I "http://localhost:8080/index.php?exec=/bin/bash"`

Resultado esperado:

<img width="775" height="149" alt="image" src="https://github.com/user-attachments/assets/1be94ee9-81d9-493b-9bc9-f71333c34f75" />

*(El código **403 Forbidden** indica que el WAF ha detectado un intento de acceso a binarios del sistema a través de la URL y lo ha bloqueado automáticamente).*

**Comprobación de protección en Formularios (POST - XSS)**

`curl -i -X POST http://localhost:8080/post.php -d "campo=<script>alert(1)</script>"`

Resultado esperado:

<img width="1042" height="343" alt="image" src="https://github.com/user-attachments/assets/db9478f3-f7ba-43cc-a53e-75f2ccd1a71f" />

*(El servidor bloquea el intento de inyección de script con un código 403, protegiendo la integridad de la aplicación).*

**Verificación de que el módulo está cargado**

`docker exec pps_p2_waf apache2ctl -M | grep security`

Resultado esperado:

<img width="1027" height="118" alt="image" src="https://github.com/user-attachments/assets/21aabf86-857f-41fb-beac-bb398a0ffc78" />

*(Esta salida confirma que el motor ModSecurity está activo y vinculado correctamente al proceso de Apache).*

### 4. URL Docker Hub
`docker pull javi2332/pps_p2_javlluapa:latest`

### 5. Limpieza
Para detener y borrar el contenedor de prueba ejecutamos:

`docker stop pps_p2_waf`

`docker rm -f pps_p2_waf`

Resultado esperado:

<img width="483" height="97" alt="image" src="https://github.com/user-attachments/assets/76b50800-3e77-4431-8f87-64d9f90dc7d7" />

