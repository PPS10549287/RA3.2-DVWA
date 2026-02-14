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
 Archivo Subido | Tipo Real            | Tipo MIME Falsificado                              | Resultado                                  |
|-------------------|----------------------------|------------------------------------------------|---------------------------------------------------------------|
| `shell.php`            | PHP Script        | `image/jpeg` | **Éxito(Uploaded)**       
