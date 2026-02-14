# Fuerza bruta - DVWA

### 1. Descripción de la Vulnerabilidad
La vulnerabilidad de fuerza bruta consiste en el intento de adivinar credenciales (usuario y contraseña) mediante el envío masivo y automatizado de combinaciones posibles hasta dar con la correcta.

#### Desafío Nivel Medium
A diferencia del nivel bajo, el nivel Medium introduce una medida de seguridad básica: un retraso en la respuesta del servidor ante un inicio de sesión fallido (normalmente un sleep(2) en el código PHP).
* El objetivo es ralentizar las herramientas automáticas.
* Sin embargo, no existe un límite de intentos (bloqueo de cuenta), lo que permite que el ataque siga siendo viable con un poco más de tiempo.

### 2. Análisis del Entorno y Explotación
Práctica realizada en **Kali Linux** sobre **VirtualBox**.

#### Identificación de la debilidad
La aplicación no implementa mecanismos robustos como:
1. Bloqueo de cuenta tras X intentos fallidos.
2. Uso de Captchas.
3. Validación de tokens anti-CSRF en el formulario de login.

#### Validación del Ataque (Paso a Paso con Burp Suite)
Para este ataque utilizaremos el módulo Intruder de Burp Suite:
1. **Interceptación:** Abrimos el navegador integrado de Burp, con la interceptación del Proxy activa y realizamos un intento de login con cualquier usuario (ej. `admin:password`).
<img width="631" height="405" alt="image" src="https://github.com/user-attachments/assets/ef2c91e2-5d59-4499-b6f0-ccdef788c37e" />

2. **Enviar a Intruder:** Hacemos clic derecho sobre la petición interceptada y seleccionamos `Send to Intruder`.
<img width="1278" height="514" alt="image" src="https://github.com/user-attachments/assets/a4fa200a-cda3-4c99-9c7e-8889258fe4c3" />

4. **Configurar Posiciones:** En la pestaña Positions, pulsamos **Clear** para eliminar posiciones automáticas y marcamos los valores del usuario y la contraseña como objetivos (Clic en Add sobre ambos) (`payload positions`).
Visualizaremos el cuerop del POST así:
<img width="793" height="270" alt="image" src="https://github.com/user-attachments/assets/7b390e01-6d74-4591-992e-f007fa28d9ef" />

5. **Carga de Diccionarios (Payloads):** * En la pestaña Payloads, cargamos una lista de usuarios comunes para el primer campo.
  * Cargamos una lista de contraseñas (como `rockyou.txt` que viene en Kali) para el segundo o los añadimos de forma manual.
<img width="449" height="306" alt="image" src="https://github.com/user-attachments/assets/7462ba8d-0673-48d5-8c14-d3283dc07b7c" />

6. **Visualización de Configuración General:**
<img width="1279" height="879" alt="image" src="https://github.com/user-attachments/assets/054a52a0-1751-416c-97bd-c2003b8a9b1e" />

7. **Lanzar ataque:** Pulsamos `Start Attack`.


#### Resultados
Identificamos el éxito analizando la **Longitud (Length)** de la respuesta HTTP o el **Código de Estado**. Una respuesta exitosa tendrá un tamaño distinto a las fallidas.
A continuación se visualiza como admin y password tienen una longitud de **5115**, a diferencia de las demás que tienen una longitud de **5072** o **5071**, dejando ver cual es el usuario y contraseña solicitados.
<img width="1235" height="446" alt="image" src="https://github.com/user-attachments/assets/da23e385-0179-47c0-80e7-1e3505370e7f" />

<img width="1235" height="446" alt="image" src="https://github.com/user-attachments/assets/cd78bdeb-5456-485c-aa32-eba700f5de78" />

**Visualización General**
<img width="1281" height="885" alt="image" src="https://github.com/user-attachments/assets/f01fc755-7b65-4d84-ae3c-c8014b092494" />

**Resumen**

| Usuario | Contraseña | Resultado                  |
|----------|------------|----------------------------|
| admin    | admin      | Fallido                    |
| admin    | password   | Éxito (Welcome admin)      |

**Verificación de acceso**
Finalmente, accedemos con admin y password, verificando el resultado:
<img width="744" height="521" alt="image" src="https://github.com/user-attachments/assets/8a30115a-b9e3-4b3e-8711-3339575cbd70" />


### 3. Mitigación y Buenas Prácticas
Para corregir esta vulnerabilidad y asegurar una puesta en producción segura:
* **Bloqueo de cuentas:** Implementar un bloqueo temporal de la cuenta o de la dirección IP tras un número determinado de intentos fallidos.
* **Políticas de contraseñas:** Obligar al uso de contraseñas complejas y largas.
* **Segundo Factor de Autenticación (2FA):** Añadir una capa extra de seguridad.
* **Uso de CAPTCHA:** Para diferenciar entre humanos y herramientas automatizadas.

### 4.Bibliografía
Para el desarrollo de esta práctica y la comprensión de las vulnerabilidades en **Damn Vulnerable Web Application (DVWA)**, se han consultado las siguientes fuentes oficiales y recursos de seguridad:
* **Digininja.** (s. f.). *GitHub digininja/DVWA: Damn Vulnerable Web Application (DVWA)*. https://github.com/digininja/DVWA.
* **Sama, A.** (s. f.). *DVWA writeups.* https://aftabsama.com/writeups/dvwa/.
* **Documentación de la Unidad:** Ciberseguridad en entornos de las tecnologías de la información: Puesta en producción segura.
