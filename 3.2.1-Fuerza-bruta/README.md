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
1. **Interceptación:** Abrimos el navegador integrado de Burp y realizamos un intento de login con cualquier usuario (ej. `admin:password`).
2. **Enviar a Intruder:** Hacemos clic derecho sobre la petición interceptada y seleccionamos `Send to Intruder`.
3. **Configurar Posiciones:** En la pestaña Positions, marcamos los valores del usuario y la contraseña como objetivos (`payload positions`).
4. **Carga de Diccionarios (Payloads):** * En la pestaña Payloads, cargamos una lista de usuarios comunes para el primer campo.
  * Cargamos una lista de contraseñas (como `rockyou.txt` que viene en Kali) para el segundo.
5. **Ajuste de hilos:** Debido al retraso del nivel Medium, ajustamos el número de hilos para no saturar la conexión.
6. **Lanzar ataque:** Pulsamos `Start Attack`.


#### Resultados
Identificamos el éxito analizando la **Longitud (Length)** de la respuesta HTTP o el **Código de Estado**. Una respuesta exitosa tendrá un tamaño distinto a las fallidas.

| Usuario | Contraseña | Resultado                  |
|----------|------------|----------------------------|
| admin    | admin      | Fallido                    |
| admin    | password   | Éxito (Welcome admin)      |

### 3. Mitigación y Buenas Prácticas
Para corregir esta vulnerabilidad y asegurar una puesta en producción segura:
* **Bloqueo de cuentas:** Implementar un bloqueo temporal de la cuenta o de la dirección IP tras un número determinado de intentos fallidos.
* **Políticas de contraseñas:** Obligar al uso de contraseñas complejas y largas.
* **Segundo Factor de Autenticación (2FA):** Añadir una capa extra de seguridad.
* **Uso de CAPTCHA:** Para diferenciar entre humanos y herramientas automatizadas.

### 4.Bibliografía
