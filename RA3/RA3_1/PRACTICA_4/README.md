# Práctica 4: Protección DoS con mod_evasive

### 1. Explicación
Esta imagen representa la capa final de la arquitectura de seguridad, heredando la robustez de la **P3 (OWASP CRS)** y añadiendo un escudo especializado contra ataques de Denegación de Servicio (DoS) y fuerza bruta:

* **Estrategia en cascada:** Basada en la imagen `javi2332/pps_p3_javlluapa`, mantiene el Hardening (P1), el motor ModSecurity (P2) y las reglas de OWASP (P3).
* **mod_evasive:** Implementación de un módulo de detección activa que rastrea IPs sospechosas basándose en la frecuencia de sus peticiones a nivel de aplicación.
* **Umbrales de Bloqueo:** Configurado para detectar ráfagas de más de 5 peticiones por segundo a una misma página, bloqueando la IP automáticamente con un código **403 Forbidden** para preservar la disponibilidad del servidor.

### 2. Guía de Despliegue
Este repositorio contiene la suite completa de seguridad activa y probada en un entorno contenedorizado.

**Paso 1: Descargar la imagen**

`docker pull javi2332/pps_p4_javlluapa:latest`

**Paso 2: Lanzar el contenedor**
Desplegamos el servicio mapeando los puertos de seguridad (8080 para HTTP y 8081 para HTTPS):

`docker run -d --name pps_p4_full -p 8080:80 -p 8081:443 javi2332/pps_p4_javlluapa:latest`

### 3. Validación y Auditoría

Para demostrar la efectividad de la protección anti-DoS y la persistencia de las capas anteriores, realizamos las siguientes pruebas:

**A. Ejecución del script de prueba de carga (Perl)**
Utilizamos el script de prueba localizado en `/root/test.pl` que simula una inundación de peticiones HTTP/1.1:

`docker exec pps_p4_full perl /root/test.pl`

Resultado esperado:

<img width="699" height="977" alt="image" src="https://github.com/user-attachments/assets/45f8ea51-c789-4d2c-b16a-1adb64873aed" />

*(Se observa cómo las primeras peticiones devuelven **200 OK** y, tras superar el umbral de 5 peticiones, el servidor comienza a responder con **403 Forbidden**, confirmando el bloqueo activo de la IP).*

**B. Verificación de registros de bloqueo (Evidencia de DoS)**
Comprobamos que el módulo ha creado un registro físico de la IP bloqueada en el directorio de logs configurado:

`docker exec pps_p4_full ls -l /var/log/mod_evasive`

Resultado esperado:

<img width="769" height="81" alt="image" src="https://github.com/user-attachments/assets/3cb36d89-b231-4c76-bb88-d906455848c7" />

*(La existencia del archivo `dos-127.0.0.1` con el propietario `www-data` confirma que el motor de evasión tiene permisos de escritura y está registrando los ataques detectados).*

**C. Prueba de carga masiva con Apache Bench (ab)**
Para generar un informe técnico de denegación de servicio, lanzamos 100 peticiones concurrentes desde el host:

`ab -n 100 -c 5 http://localhost:8080/`

Resultado obtenido:

<img width="746" height="996" alt="image" src="https://github.com/user-attachments/assets/af674894-9299-4bc3-a871-e298b8c0ead9" />

> [!TIP]
> **Informe Detallado:** Se adjunta un reporte técnico completo en el archivo [informe_apache_bench.txt](./informe_apache_bench.txt) dentro de este repositorio para la revisión detallada. Este informe no es el que se visualiza en la imagen pertenece a una segunda prueba realizada, ya que si no no se podía mostrar la ejecución tal y como se visualiza en la imagen.

<img width="858" height="35" alt="image" src="https://github.com/user-attachments/assets/296930a2-2ebd-4f36-83d9-540159071d59" />

* **Análisis del informe:** Se observa que de las 100 peticiones realizadas, **90 fueron rechazadas (Non-2xx responses)**. Esto demuestra que tras las primeras peticiones exitosas, el módulo `mod_evasive` identificó el exceso de tráfico y activó el bloqueo 403, limitando el consumo de recursos del servidor de forma efectiva.

**D. Persistencia de capas de seguridad (Heredabilidad de P1, P2 y P3)**
Comprobamos que las reglas de OWASP siguen filtrando ataques de inyección (XSS) a pesar de la nueva capa de protección DoS:

`curl -I "http://localhost:8080/?test=<script>alert(1)</script>"`

Resultado esperado:

<img width="854" height="142" alt="image" src="https://github.com/user-attachments/assets/584a266c-6ecc-4f7b-9f30-0aa7f1e9e61f" />

*(El código **403 Forbidden** y la presencia de cabeceras de seguridad de la P1 demuestran que el Hardening y el WAF siguen protegiendo el servidor).*

### 4. URL Docker Hub
`docker pull javi2332/pps_p4_javlluapa:latest`

### 5. Limpieza
Para detener y borrar el entorno completo de prueba:

`docker stop pps_p4_full`

`docker rm -f pps_p4_full`

Resultado esperado:

<img width="499" height="96" alt="image" src="https://github.com/user-attachments/assets/63261341-66e9-454d-b31a-310712d57c1b" />

