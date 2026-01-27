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

### A. Contenido del Dockerfile
Este es el punto de partida. Aquí no heredamos de nadie, sino que construimos desde cero sobre Debian para tener el control total de la seguridad desde el primer segundo.

1. Limpieza de superficie de ataque
Lo primero que hago es instalar Apache y, acto seguido, empiezo a recortar funciones innecesarias.

El comando clave: Deshabilito el módulo autoindex. Esto es vital porque evita que, si un directorio no tiene un index.html, Apache muestre la lista de archivos al público. Es el primer paso para evitar que un atacante cotillee nuestra estructura de archivos.

2. Activación de la "armadura" (SSL y Headers)
Preparo el servidor para el mundo moderno activando ssl para el cifrado y headers para la privacidad. También activo rewrite por si necesitamos redirigir tráfico de forma inteligente más adelante.

3. Implementación de CSP e HSTS
Aquí es donde inyecto nuestra política de seguridad personalizada (csp_hsts.conf).

CSP (Content Security Policy): Le decimos al navegador qué fuentes de contenido son de confianza, bloqueando de raíz la ejecución de scripts maliciosos externos.

HSTS: Obligamos a los navegadores a recordar que nuestro sitio solo se habla por HTTPS, eliminando el riesgo de que alguien intercepte la conexión en el paso de HTTP a SSL.

4. Configuración del Sitio Seguro
Para no complicar esta fase inicial con certificados externos, aprovecho los certificados snakeoil (los que vienen por defecto en Debian) y activo el sitio default-ssl. Así garantizamos que, desde el minuto uno, el contenedor ya responde por el puerto 443.

> [!IMPORTANT]
> <img width="999" height="703" alt="image" src="https://github.com/user-attachments/assets/65deb165-65f8-47f2-907c-fe13e50cef73" />

### B. Contenido del fichero csp_hsts.conf
Este fichero es el que realmente aplica el bastionado (hardening) a nivel de servidor. No se limita a cifrar, sino que reduce la información que le damos a un posible atacante y controla qué puede ejecutar el navegador.

1. El modo "Sigilo" (ServerTokens y Signature)
Aquí estoy forzando a Apache a que sea discreto.

ServerTokens ProductOnly: En lugar de decir "Soy Apache versión 2.4.52 ejecutándose en Debian", el servidor solo dirá "Apache".

ServerSignature Off: Elimino el pie de página que aparece en los errores del servidor (como el típico 404).

Por qué es importante: Si un atacante no sabe exactamente qué versión usas, no puede buscar exploits específicos para tu sistema.

2. Seguridad en el Transporte (HSTS)
Uso el módulo de cabeceras para inyectar el Strict-Transport-Security. Le digo a cualquier navegador que visite la web que, durante los próximos 2 años (max-age), la conexión debe ser HTTPS. Esto protege a nuestros usuarios de ataques donde alguien intenta "bajar" su conexión a una versión HTTP no cifrada.

3. Política de Contenido (CSP)
Esta es la parte más potente. Defino una Content-Security-Policy para evitar ataques como el Cross-Site Scripting (XSS).

default-src 'self': Solo permito contenido (scripts, estilos, etc.) que venga de mi propio servidor.

Filtros específicos: He configurado permisos específicos para imágenes y contenido multimedia de fuentes concretas.

Resultado: Si un atacante intenta inyectar un script desde una web maliciosa, el navegador del usuario lo bloqueará automáticamente porque no está en esta "lista blanca".

4. Bloqueo de Listado de Directorios
Aunque ya deshabilité el módulo en el Dockerfile, aquí refuerzo la seguridad con Options -Indexes para el directorio raíz.

Misión: Si alguien entra en una carpeta que no tiene un archivo de inicio, recibirá un error de "Acceso denegado" en lugar de ver una lista de todos nuestros archivos.

> [!IMPORTANT]
> <img width="1160" height="323" alt="image" src="https://github.com/user-attachments/assets/240931a1-3c3f-4e28-838f-7e955d803ae0" />

### 2. Guía de Despliegue
Este repositorio utiliza una imagen preconfigurada alojada en Docker Hub. No necesitamos los archivos de configuración locales para lanzarlo, ya que el archivo `csp_hsts.conf` y el resto de ajustes están integrados en la imagen.

**Paso 1: Descargar la imagen**

`docker pull pps10549287/pps-pr1:latest`

**Paso 2: Lanzar el contenedor**
Mapeamos el puerto 8080 para HTTP y el 8081 para HTTPS (puerto 443 interno):

`docker run -d --name pps-pr1-javlluapa -p 8080:80 -p 8081:443 pps10549287/pps-pr1:latest`

**Paso 3: Visualizar contenedor activo**
Comprobamos que el contenedor se encuentra corriendo y está operativo:

`docker ps`

### 3: Validación y Auditoría**

Para verificar que todas las medidas de seguridad se han aplicado correctamente, realizamos peticiones al contenedor:

Verificación de Ocultación y Cabeceras (HTTP y HTTPS)

# Comprobación de cabeceras en puerto 8080
`curl -I http://localhost:8080`

> [!IMPORTANT]
> Resultado esperado:
> <img width="1083" height="280" alt="image" src="https://github.com/user-attachments/assets/e0c20f23-efe3-440e-98f1-64f0a05a264b" />

> [!NOTE]
> *En la respuesta se observa que el campo `Server` no revela la versión de Apache ni del SO, y se verifica la correcta implementación de la política `Content-Security-Policy`.*

# Comprobación de puerto seguro (HSTS) en puerto 8081
> [!NOTE]
> *Usamos -k porque los certificados son autofirmados.*

`curl -Ik https://localhost:8081`

> [!IMPORTANT]
> Resultado esperado:
> <img width="1081" height="274" alt="image" src="https://github.com/user-attachments/assets/20602d7b-4fa8-42d1-b202-6bcedffd4ddb" />

> [!NOTE]
> *La presencia de la cabecera `Strict-Transport-Security` confirma que el servidor obliga al navegador a mantener una conexión segura (HTTPS) durante el tiempo configurado.*

### 4. URL Docker Hub
`docker pull pps10549287/pps-pr1:latest`

### 5. Limpieza
Para detener y borrar el contenedor de prueba ejecutamos:

`docker stop pps-pr1-javlluapa`

`docker rm pps-pr1-javlluapa`

Resultado esperado:

<img width="501" height="98" alt="image" src="https://github.com/user-attachments/assets/c26fcece-170c-40a3-954f-673e550ec5fe" />


