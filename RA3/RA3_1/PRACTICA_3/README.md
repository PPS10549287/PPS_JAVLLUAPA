# Práctica 3: ModSecurity + OWASP CRS

### 1. Explicación
Esta imagen aplica un nivel más de seguridad en la serie, heredando de la **P2 (WAF Base)** e integrando el conjunto de reglas más reconocido de la industria:

* **Estrategia en cascada:** Se utiliza la imagen `javi2332/pps_p2_javlluapa` como base, manteniendo el Hardening de la P1 y el motor ModSecurity de la P2.
* **OWASP Core Rule Set (CRS):** Integración de reglas avanzadas para proteger contra el "Top 10" de riesgos de seguridad (SQLi, XSS, Local File Inclusion, etc.).
* **Protección Inteligente:** El servidor ahora cuenta con una lógica de detección mucho más profunda y precisa gracias al repositorio oficial de `coreruleset`.

### 2. Guía de Despliegue
Este repositorio utiliza la imagen final con el conjunto de reglas OWASP ya preconfigurado y optimizado para su carga.

**Paso 1: Descargar la imagen**

`docker pull javi2332/pps_p3_javlluapa:latest`

**Paso 2: Lanzar el contenedor**
Mapeamos los puertos de servicio (8080 para HTTP y 8081 para HTTPS):

`docker run -d --name pps_p3_owasp -p 8080:80 -p 8081:443 javi2332/pps_p3_javlluapa:latest`

### 3. Validación y Auditoría

Al contar con el Core Rule Set de OWASP, realizamos pruebas de intrusión para verificar la respuesta del firewall:

**A. Bloqueo de ataque avanzado (Path Traversal)**

`curl -I "http://localhost:8080/?exec=/../../"`

Resultado esperado:

<img width="664" height="177" alt="image" src="https://github.com/user-attachments/assets/9f982ccc-b3ab-4e5a-8661-92b10eeb036d" />

*(El código **403 Forbidden** confirma que las reglas de OWASP han identificado el patrón de navegación prohibida por directorios).*

**B. Bloqueo de ataque avanzado (Command Injection)**

`curl -I "http://localhost:8080/?exec=/bin/bash"`

Resultado esperado:

<img width="683" height="153" alt="image" src="https://github.com/user-attachments/assets/516c7b20-186f-4569-b36b-784fa2c0f817" />


*(El código **403 Forbidden** confirma que las reglas de OWASP han identificado el patrón de navegación prohibida por directorios).*

**C. Visualización de Logs (Auditoría de OWASP)**
Para ver cómo el servidor "caza" el ataque en tiempo real:

`docker exec pps_p3_owasp tail -f /var/log/apache2/error.log`

Resultado esperado (Ataque 1):

<img width="1932" height="871" alt="image" src="https://github.com/user-attachments/assets/07ddd81b-7fe8-4292-b7d4-799d43ce982e" />

*(En los logs se puede observar el ID de la regla de OWASP activada y la descripción detallada del ataque bloqueado).*

Resultado esperado (Ataque 2):

<img width="1918" height="449" alt="image" src="https://github.com/user-attachments/assets/a14e7966-8193-4aec-95ad-30b845851196" />

*(En los logs se puede observar el ID de la regla de OWASP activada y la descripción detallada del ataque bloqueado).*

**D. Verificación de persistencia del Hardening (Capa 1)**

`curl -I http://localhost:8080`

Resultado esperado:

<img width="1091" height="293" alt="image" src="https://github.com/user-attachments/assets/74222962-ef5c-4e19-b92f-edf4388eb5c5" />

*(Se comprueba que la ocultación del servidor y las cabeceras de seguridad de la P1 siguen vigentes gracias a la herencia de imágenes).*


### 4. URL Docker Hub
`docker pull javi2332/pps_p3_javlluapa:latest`

### 5. Limpieza
Para detener y borrar el contenedor de prueba ejecutamos:

`docker stop pps_p3_owasp`

`docker rm -f pps_p3_owasp`

Resultado esperado:

<img width="499" height="99" alt="image" src="https://github.com/user-attachments/assets/2346d587-0627-4863-9ce2-c753f1b2e7a4" />
