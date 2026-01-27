# Práctica: Instalación de Certificado Digital SSL/TLS en Apache

### 1. Explicación
En esta práctica se ha implementado la capa de transporte seguro (SSL/TLS) sobre la infraestructura ya blindada en las prácticas anteriores, culminando la arquitectura de "Defensa en Profundidad":

* **Estrategia en cascada:** Basada en la imagen `pps10549287/pps-pr4`, integrando Hardening (P1), ModSecurity (P2), OWASP CRS (P3) y protección DoS (P4).
* **Activación de HSTS:** Con la instalación del certificado SSL/TLS, la cabecera `Strict-Transport-Security` configurada previamente pasa a ser funcional, obligando a los navegadores a utilizar exclusivamente el puerto 443.
> [!IMPORTANT]
> **Cierre del ciclo de Hardening (P1):** La implementación de este certificado SSL/TLS es el requisito técnico indispensable para que la cabecera **HSTS (HTTP Strict Transport Security)**, configurada originalmente en la Práctica 1, sea efectiva. Con este paso, el servidor no solo solicita conexiones seguras, sino que el navegador tiene la infraestructura necesaria para acatar dicha política de seguridad.
* **Cifrado SSL/TLS:** Generación de un certificado X.509 autofirmado de 2048 bits mediante OpenSSL, configurado para el dominio `www.midominioseguro.com`.
* **Redirección Forzosa (HSTS):** Configuración de Apache para redirigir permanentemente todo el tráfico inseguro (HTTP/80) hacia el canal cifrado (HTTPS/443), garantizando la integridad y confidencialidad de los datos.

### A. Contenido del Dockerfile
En este punto, he tomado la base de la práctica anterior (que ya traía el WAF y el anti-DDoS) y me he centrado en asegurar que toda la comunicación sea privada y esté cifrada.

1. Generación de identidad (Certificado SSL)
Lo primero que hago es crear un "DNI" para nuestro servidor. Uso openssl para generar un certificado autofirmado de 2048 bits. Con esto, habilitamos el cifrado RSA para que la información que viaja entre el usuario y el servidor no pueda ser leída por terceros.

2. Configuración del sitio seguro
Copio el archivo default-ssl.conf para decirle a Apache exactamente cómo debe comportarse cuando alguien intente entrar de forma segura. Activo el módulo ssl y le indico al servidor que deje de usar la configuración por defecto (000-default.conf) para centrarse en nuestro sitio protegido.

### B. Contenido de default-ssl.conf
Este archivo es el cerebro del tráfico web y se divide en dos grandes misiones:

Misión A: Redirección Forzosa (Puerto 80)
He configurado un bloque que escucha por el puerto 80 (HTTP normal). Su única función es "cazar" a cualquiera que entre sin seguridad y mandarlo inmediatamente al puerto 443. Nadie se queda en el lado inseguro; es una redirección permanente.

Misión B: El búnker HTTPS (Puerto 443)
Aquí es donde ocurre la magia de la seguridad:

Encendido del motor SSL: Activo el SSLEngine y le digo dónde están guardadas las llaves que generamos antes.

HSTS (Seguridad Persistente): He añadido una cabecera de seguridad muy potente. El Strict-Transport-Security le dice al navegador del usuario: "A partir de ahora, aunque intentes entrar por HTTP, ni me preguntes; conecta siempre por HTTPS durante los próximos dos años". Esto evita ataques de degradación de SSL.

Entorno Seguro: Finalmente, aseguro que incluso si ejecutamos scripts (PHP o CGI), estos se manejen bajo el estándar de variables de entorno seguras de SSL.

### 2. Guía de Despliegue

**Paso 1: Configuración del entorno local (Host)**
Para que el navegador y las herramientas de test reconozcan el dominio, es necesario mapear la IP local en el archivo `/etc/hosts` de la máquina anfitriona:

`127.0.0.1  www.midominioseguro.com`

> [!IMPORTANT]
> Resultado esperado en la configuración:
> <img width="643" height="116" alt="image" src="https://github.com/user-attachments/assets/08c5a420-dac4-48cf-b95d-628b17a99a73" />

**Paso 2: Descargar la imagen**

`docker pull pps10549287/pps-pr-cert:latest`

**Paso 3: Lanzar el contenedor**
Mapeamos el puerto 8080 para HTTP y el 8081 para HTTPS (puerto 443 interno):
`docker run -d --name pps-pr-cert-javlluapa -p 8080:80 -p 8081:443 pps10549287/pps-pr-cert:latest`

**Paso 4: Visualizar contenedor activo**
Comprobamos que el contenedor se encuentra corriendo y está operativo:

`docker ps`

### 3. Validación y Auditoría

**A. Verificación de Redirección HTTP -> HTTPS**
Comprobamos que el servidor rechaza conexiones inseguras y fuerza el salto al puerto seguro:

`curl -I http://localhost:8080`

> [!IMPORTANT]
> Resultado esperado:
> 
> <img width="637" height="166" alt="image" src="https://github.com/user-attachments/assets/56e4da18-b1b7-43c6-8a2a-c0db88244d2b" />

> [!NOTE]
> *El código **301 Moved Permanently** hacia https://www.midominioseguro.com/ confirma la política de transporte seguro.*

**B. Inspección técnica del Certificado (Issuer)**
Validamos que el certificado contiene los datos de identidad configurados durante la construcción (Castellón, Seguridad, etc.):

`curl -Iv -k https://localhost:8081 2>&1 | grep "issuer"`

>  [!IMPORTANT]
> Resultado esperado:
> <img width="845" height="55" alt="image" src="https://github.com/user-attachments/assets/a359e6f2-9a23-4749-bbda-a1a7e25f0653" />

> [!NOTE]
> *La línea `issuer: C=ES; ST=Castellon; L=Castellon; O=Seguridad; CN=www.midominioseguro.com` confirma la autoría del certificado.*

**C. Validación Visual en Navegador**
Al acceder a `https://www.midominioseguro.com:8081`, se verifica el aviso de seguridad por certificado autofirmado y se inspeccionan los detalles:

> [!IMPORTANT]
> Resultado esperado:
><img width="1029" height="1278" alt="image" src="https://github.com/user-attachments/assets/d392389a-4986-4d60-8dce-4cd2bf1f2ef0" />

>  [!NOTE]
> Aviso de seguridad por certificado autofirmado

> [!IMPORTANT]
> <img width="1029" height="1278" alt="image" src="https://github.com/user-attachments/assets/ebfa7317-c247-486c-be13-487802c0e574" />

> [!NOTE]
> Detalles del certificado

> [!IMPORTANT]
> <img width="1111" height="1293" alt="image" src="https://github.com/user-attachments/assets/d017b8b0-ed97-418f-883a-551fa887e659" />

> [!NOTE]
> Página web en funcionamiento

**D. Persistencia de Seguridad (WAF + DoS + Hardening)**
Verificamos que las capas de las prácticas anteriores siguen activas bajo el túnel SSL:

`curl -I -k "https://localhost:8081/?exec=/bin/bash"`

> [!IMPORTANT]
> Resultado esperado:
> <img width="707" height="139" alt="image" src="https://github.com/user-attachments/assets/abdb97a7-2d65-4e89-86a2-0fb2f5a4faf0" />

> [!NOTE]
> *El código **403 Forbidden** demuestra que ModSecurity sigue inspeccionando el tráfico una vez descifrado.*

### 4. URL Docker Hub
`docker pull pps10549287/pps-pr-cert:latest`

### 5. Limpieza

`docker stop pps-pr-cert-javlluapa && docker rm -f pps-pr-cert-javlluapa`

> [!IMPORTANT]
> Resultado esperado:
> <img width="914" height="118" alt="image" src="https://github.com/user-attachments/assets/8cf748f2-9b73-4c29-9b43-aff3b11f1bab" />


