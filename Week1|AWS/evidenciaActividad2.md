# Preguntas de reflexión

### 1. ¿Qué podía hacer el usuario justo después de crearlo, antes de adjuntarle una política? ¿Por qué?
**R//** El usuario no podía hacer nada, dado a que por defecto nace sin permisos o políticas.

---

### 2. En el error del paso 13, ¿qué tres datos te dice el mensaje?
**R//** El error decía **Acceso denegado**, porque la operación que quiere hacer el usuario no está autorizada para leer ni listar los usuarios.

---

### 3. ¿Por qué cambió el ARN al renombrar el usuario?
**R//** Porque el ARN de un usuario IAM incluye su nombre en la ruta; al renombrarlo, su ARN cambia para reflejar el nuevo nombre.

---

### 4. ¿Por qué AWS obliga a borrar las llaves antes de borrar el usuario?
**R//** Porque AWS te obliga a revisar y eliminar las credenciales activas antes de borrar el usuario, evitando dejar llaves huérfanas o romper aplicaciones y *scripts* que dependían de ellas.

---

### 5. ¿Qué pasaría si le hubieras dado `AdministratorAccess` en vez de solo lectura? ¿En qué caso real estaría justificado?
**R//** Si se le diera `AdministratorAccess`, se le habría dado acceso a opciones de crear, modificar o eliminar cualquier recurso y gestionar permisos en toda la cuenta, por lo que no se debería dar sin importancia dado que aumenta el riesgo de errores o vulnerabilidades. 

Tal vez estaría justificado para administradores de infraestructura o arquitectos *cloud* encargados de configurar la cuenta y gestionar permisos.