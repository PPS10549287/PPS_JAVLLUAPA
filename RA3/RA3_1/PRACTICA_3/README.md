# Práctica 3: ModSecurity + OWASP CRS

### 1. Explicación
Esta imagen aplica un nivel más de seguridad en la serie, heredando de la **P2 (WAF Base)** e integrando el conjunto de reglas más reconocido de la industria:

* **Estrategia en cascada:** Se utiliza la imagen `javi2332/pps_p2_javlluapa` como base, manteniendo el Hardening de la P1 y el motor ModSecurity de la P2.
* **OWASP Core Rule Set (CRS):** Integración de reglas avanzadas para proteger contra el "Top 10" de riesgos de seguridad (SQLi, XSS, Local File Inclusion, etc.).
* **Protección Inteligente:** El servidor ahora cuenta con una lógica de detección mucho más profunda y precisa gracias al repositorio oficial de `coreruleset`.

### 2. Guía de Despliegue
Este repositorio utiliza la imagen final con el conjunto de reglas OWASP ya preconfigurado y optimizado para su carga.

**Paso 1: Descargar la imagen**

`docker pull pps10549287/pps-pr3:latest`

**Paso 2: Lanzar el contenedor**
Mapeamos los puertos de servicio (8080 para HTTP y 8081 para HTTPS):

`docker run -d --name pps-pr3-javlluapa -p 8080:80 -p 8081:443 pps10549287/pps-pr3:latest`

### 3. Validación y Auditoría

Al contar con el Core Rule Set de OWASP, realizamos pruebas de intrusión para verificar la respuesta del firewall:

**A. Bloqueo de ataque avanzado (Path Traversal)**

`curl -I "http://localhost:8080/?exec=/../../"`

> [!IMPORTANT]
> Resultado esperado:
> <img width="651" height="144" alt="image" src="https://github.com/user-attachments/assets/4f59283e-f207-4cc6-acca-a10288c97785" />

> [!NOTE]
> *El código **403 Forbidden** confirma que las reglas de OWASP han identificado el patrón de navegación prohibida por directorios.*

**B. Bloqueo de ataque avanzado (Command Injection)**

`curl -I "http://localhost:8080/?exec=/bin/bash"`

> [!IMPORTANT]
> Resultado esperado:
> <img width="672" height="144" alt="image" src="https://github.com/user-attachments/assets/d66dad38-4a9b-48cb-bd29-e995f972446a" />

> [!NOTE]
> *El código **403 Forbidden** confirma que las reglas de OWASP han identificado el patrón de navegación prohibida por directorios.*

**C. Visualización de Logs (Auditoría de OWASP)**
Para ver cómo el servidor "caza" el ataque en tiempo real:

`docker exec pps-p3-javlluapa tail -f /var/log/apache2/error.log`

> [!IMPORTANT]
> Resultado esperado (Ataque 1):
> <img width="1185" height="1081" alt="image" src="https://github.com/user-attachments/assets/71b3a817-8b37-427e-8ec0-aeae86497458" />

> [!NOTE]
> *En los logs se puede observar el ID de la regla de OWASP activada y la descripción detallada del ataque bloqueado.*

> [!IMPORTANT]
> Resultado esperado (Ataque 2):
> <img width="1177" height="410" alt="image" src="https://github.com/user-attachments/assets/d418a5b5-4750-4392-9279-dbca3567aca6" />

> [!NOTE]
> *En los logs se puede observar el ID de la regla de OWASP activada y la descripción detallada del ataque bloqueado.*

**D. Verificación de persistencia del Hardening (Capa 1)**

`curl -I http://localhost:8080`

> [!IMPORTANT]
> Resultado esperado:
> <img width="1084" height="282" alt="image" src="https://github.com/user-attachments/assets/51c5574b-8309-4286-ad45-a3c5cd7adf1a" />

> [!NOTE]
> *Se comprueba que la ocultación del servidor y las cabeceras de seguridad de la P1 siguen vigentes gracias a la herencia de imágenes.*

### 4. URL Docker Hub
`docker pull pps10549287/pps-pr3:latest`

### 5. Limpieza
Para detener y borrar el contenedor de prueba ejecutamos:

`docker stop pps-pr3-javlluapa`

`docker rm pps-pr3-javlluapa`

> [!IMPORTANT]
> Resultado esperado:
>
> <img width="498" height="92" alt="image" src="https://github.com/user-attachments/assets/4b601ee2-a2f1-4c90-b389-62c3d71b866a" />

