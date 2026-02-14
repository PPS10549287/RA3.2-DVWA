# Inyección de comandos - DVWA

### 1. Descripción de la Vulnerabilidad
La inyección de comandos sucede cuando una aplicación web transmite datos introducidos por el usuario (en este caso, una IP para un comando "**ping**") directamente al shell del S.O sin validación o filtrado suficiente.
En este ejercicio, la aplicación solicita una dirección IP para ejecutar un comando de red estándar (**ping**).

#### El desafió del Nivel Medium
A diferencia del nivel bajo, el nivel Medium intenta implementar una "lista negra" (blacklist) de caracteres peligrosos. El desarrollador utiliza funciones de sustitución en PHP para eliminar operadores que permiten encadenar comandos. Por ejemplo, el código busca bloquear:
* **&&** (Ejecuta el segundo comando solo si el primero tiene éxito).
* **;** (Finaliza una instrucción y comienza otra).

#### Ejemplo de bloqueo en el código 


