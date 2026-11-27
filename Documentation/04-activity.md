# Actividad: un rol para tu instancia EC2

## Objetivo

Crear un rol de IAM con menor privilegio y dejarlo listo dentro de un perfil de instancia, para usarlo en la práctica de EC2.

## Antes de empezar

- AWS CLI v2 configurada con tu usuario administrador.
- Corre `aws sts get-caller-identity` y `aws configure get region` para confirmar quién eres y en qué región trabajas.
- Ten abiertos `01-info.md` y `02-command.md`: ahí están los comandos que necesitas.
- Crea una carpeta de trabajo llamada `practica-roles` y entra en ella. Todos los comandos se corren desde ahí.

## Parte 1: Preparar el recurso a proteger

1. Crea un bucket llamado `reportes-<tu-nombre>-<número>`, por ejemplo `reportes-laura-4821`.
2. Crea un archivo `reporte.txt` con una línea de contenido y súbelo al bucket.
3. Lista el contenido del bucket y confirma que aparece.

**Resultado esperado:** el bucket existe y tiene un objeto.

## Parte 2: Escribir las dos políticas

4. Crea el archivo `confianza.json` con una política que permita asumir el rol **solo al servicio EC2**.
5. Crea el archivo `permisos.json` con una política que permita **solo leer objetos** de tu bucket.
6. Comprueba que los dos archivos son JSON válido.

Pista: los dos modelos están en `01-info.md`. Cambia el nombre del bucket por el tuyo.

## Parte 3: Crear el rol

7. Intenta crear el rol `ec2-lector-s3` **sin** indicar la política de confianza y lee el error.
8. Ahora créalo correctamente, con `confianza.json`.
9. Muestra los datos del rol y anota su `Arn`.
10. Lista las políticas en línea del rol.

**Resultado esperado:** el rol existe, tiene política de confianza y todavía no tiene ningún permiso.

## Parte 4: Dar permisos

11. Agrégale al rol una política en línea llamada `leer-reportes` usando `permisos.json`.
12. Lista de nuevo las políticas en línea y confirma que aparece.
13. Muestra el contenido de la política `leer-reportes` desde AWS.

**Resultado esperado:** la política muestra una sola acción, `s3:GetObject`, sobre tu bucket.

## Parte 5: Editar

14. Agrégale al rol una descripción: `Rol para que EC2 lea reportes`.
15. Agrégale una etiqueta con clave `semana` y valor `1`.
16. Muestra los datos del rol y sus etiquetas para verificar los cambios.

## Parte 6: Perfil de instancia

17. Crea un perfil de instancia llamado `ec2-lector-s3`.
18. Mete el rol dentro del perfil.
19. Muestra el perfil y confirma que contiene el rol.

**Resultado esperado:** el perfil `ec2-lector-s3` tiene el rol `ec2-lector-s3` adentro.

## Parte 7: Comprobar las reglas de IAM

20. Intenta borrar el rol con `aws iam delete-role` y lee el error.

**Resultado esperado:** el comando falla con `DeleteConflict`. El rol sigue existiendo.

## Importante: no borres nada todavía

El bucket, el rol y el perfil de instancia se usan en la práctica de EC2. Al terminar esta actividad deben quedar creados:

| Recurso | Nombre |
|---|---|
| Bucket | `reportes-<tu-nombre>-<número>` con `reporte.txt` |
| Rol | `ec2-lector-s3` |
| Perfil de instancia | `ec2-lector-s3` |

Estos tres recursos no generan costo mientras esperan.

## Limpieza (solo al terminar la práctica de EC2)

Hazlo en este orden:

1. Saca el rol del perfil de instancia y borra el perfil.
2. Borra la política en línea del rol.
3. Borra el rol.
4. Vacía el bucket y bórralo.

## Entregable

Un archivo `evidencia.md` con:

- El contenido de `confianza.json` y `permisos.json`.
- Captura o salida de los pasos 7, 9, 13, 16, 19 y 20.
- La respuesta a las preguntas de reflexión.

## Preguntas de reflexión

1. ¿Qué diferencia hay entre la política de confianza y la política de permisos? ¿Qué pasaría si el rol tuviera solo una de las dos?
2. ¿Por qué es más seguro que la instancia use un rol en vez de guardar una llave de acceso con `aws configure`?
3. Con la política `leer-reportes`, ¿la instancia podrá listar el contenido del bucket? ¿Y borrar `reporte.txt`? Explica por qué.
4. En el error del paso 20, ¿qué le falta por hacer antes de poder borrar el rol?
5. Si mañana la instancia también necesita subir archivos al bucket, ¿qué cambiarías y en cuál de los dos archivos?

## Criterios de evaluación

| Criterio | Cumple |
|---|---|
| La política de confianza permite asumir el rol solo al servicio EC2 | |
| La política de permisos solo permite `s3:GetObject` sobre su propio bucket | |
| El rol tiene descripción y etiqueta | |
| El perfil de instancia contiene el rol | |
| Se evidencian los dos errores provocados (pasos 7 y 20) | |
| El bucket, el rol y el perfil quedaron creados para la práctica de EC2 | |
| Las preguntas están respondidas con sus propias palabras | |
