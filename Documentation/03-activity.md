# Actividad: ciclo de vida de un bucket en S3

## Objetivo

Crear un bucket, subir, consultar, modificar y compartir objetos desde la AWS CLI, y protegerlo con una política de menor privilegio.

## Antes de empezar

- AWS CLI v2 configurada con tu usuario administrador.
- Corre `aws sts get-caller-identity` y `aws configure get region` para confirmar quién eres y en qué región trabajas.
- Ten abierto `info.md`: ahí están los comandos que necesitas.
- Crea una carpeta de trabajo llamada `practica-s3` y entra en ella.

## Parte 1: Crear

1. Crea un bucket llamado `practica-<tu-nombre>-<número>`, por ejemplo `practica-laura-4821`.
2. Lista todos los buckets de la cuenta y confirma que aparece.
3. Consulta en qué región quedó tu bucket.

**Resultado esperado:** el bucket existe, está vacío y está en la región de tu proyecto.

Pista: si el nombre ya existe en AWS, cambia el número. Solo se permiten minúsculas, números y guiones.

## Parte 2: Subir

4. Crea tres archivos de texto: `notas.txt`, `tareas.txt` y `secreto.txt`, cada uno con una línea de contenido.
5. Sube `notas.txt` al bucket.
6. Sube los otros dos con un solo comando usando `sync`.
7. Lista el contenido del bucket con tamaños legibles y el total.

**Resultado esperado:** el bucket tiene tres objetos.

## Parte 3: Consultar y descargar

8. Muestra el contenido de `notas.txt` en la terminal sin descargarlo.
9. Muestra el tamaño y la fecha de `tareas.txt` sin descargarlo.
10. Crea una carpeta `descargas` y descarga ahí `secreto.txt`.

## Parte 4: Editar

11. Renombra `tareas.txt` a `pendientes.txt` dentro del bucket.
12. Modifica tu `notas.txt` local agregándole una línea y vuelve a subirlo.
13. Muestra el contenido de `notas.txt` desde el bucket y verifica el cambio.

**Resultado esperado:** el objeto se sobrescribe; S3 no guarda la versión anterior.

## Parte 5: Compartir sin hacerlo público

14. Genera un enlace temporal de 60 segundos para `notas.txt`.
15. Ábrelo en el navegador antes de que pase el minuto.
16. Espera a que expire y vuelve a abrirlo.

**Resultado esperado:** el enlace funciona primero y luego devuelve un error de acceso. El bucket nunca dejó de ser privado.

## Parte 6: Menor privilegio

17. Crea un usuario llamado `lector-s3-<tu-nombre>`.
18. Crea un archivo `solo-lectura.json` con esta política, usando el nombre de tu bucket:

    {
      "Version": "2012-10-17",
      "Statement": [{
        "Effect": "Allow",
        "Action": ["s3:GetObject"],
        "Resource": "arn:aws:s3:::TU-BUCKET/*"
      }]
    }

19. Adjúntasela al usuario como política en línea:

    `aws iam put-user-policy --user-name lector-s3-<tu-nombre> --policy-name leer-objetos --policy-document file://solo-lectura.json`

20. Crea una llave de acceso para el usuario y configura el perfil `lector-s3`. Escribe la región de tu proyecto cuando la pida.
21. Con ese perfil, muestra el contenido de `notas.txt`.
22. Con ese perfil, intenta listar el contenido del bucket.
23. Con ese perfil, intenta borrar `secreto.txt`.

**Resultado esperado:** el paso 21 funciona; los pasos 22 y 23 devuelven `AccessDenied`.

## Parte 7: Eliminar

Hazlo en este orden, con tu usuario administrador:

24. Intenta borrar el bucket sin vaciarlo y lee el error.
25. Borra todos los objetos del bucket.
26. Borra el bucket.
27. Borra la llave de acceso, la política en línea y el usuario `lector-s3-<tu-nombre>`.
28. Elimina el bloque `[lector-s3]` de `~/.aws/credentials` y de `~/.aws/config`.
29. Lista los buckets y los usuarios y confirma que no quedó nada de la actividad.

## Entregable

Un archivo `evidencia.md` con:

- Captura o salida de los pasos 3, 7, 13, 16, 21, 22, 23, 24 y 29.
- La respuesta a las preguntas de reflexión.

**Nunca incluyas la clave secreta de la llave de acceso en la evidencia ni la subas a GitHub.**

## Preguntas de reflexión

1. ¿Por qué el nombre de tu bucket tuvo que ser distinto al de tus compañeros si cada uno tiene su propia cuenta?
2. El usuario del paso 21 pudo leer un archivo, pero en el paso 22 no pudo ver la lista. ¿Qué acción le faltaba en la política?
3. En la política, ¿qué significa el `/*` al final del `Resource`? ¿Qué pasaría si se quitara?
4. ¿En qué se diferencia compartir un archivo con un enlace temporal de hacer público el bucket?
5. Según el modelo de responsabilidad compartida, si alguien deja un bucket público con datos de clientes, ¿de quién es la responsabilidad?

## Criterios de evaluación

| Criterio | Cumple |
|---|---|
| El bucket fue creado, usado y eliminado en el orden correcto | |
| Se evidencia el enlace temporal funcionando y luego expirado | |
| Se evidencia una acción permitida y dos denegadas con el usuario lector | |
| La política solo permite `s3:GetObject` sobre su propio bucket | |
| La evidencia no expone ninguna clave secreta | |
| No quedaron buckets, usuarios ni llaves de la actividad en la cuenta | |