# Actividad: tu primer servidor en EC2

## Objetivo

Lanzar una instancia EC2 con un servidor web, protegerla con un security group y comprobar que usa un rol de IAM para leer un bucket sin tener llaves guardadas.

## Antes de empezar

- AWS CLI v2 configurada con tu usuario administrador.
- Haber terminado la actividad de [`04-ROLES`](../04-ROLES/03-activity.md). Deben existir tu bucket con `reporte.txt`, el rol `ec2-lector-s3` y el perfil de instancia `ec2-lector-s3`.
- Ten abiertos `01-info.md` y `02-command.md`: ahí están los comandos que necesitas.
- Crea una carpeta de trabajo llamada `practica-ec2` y entra en ella. Todos los comandos se corren desde ahí.

> Esta actividad consume créditos mientras la instancia está encendida. Hazla de corrido y termina con la Parte 8.

## Parte 1: Reconocer el terreno

1. Confirma quién eres y en qué región trabajas.
2. Confirma que el perfil de instancia `ec2-lector-s3` existe y contiene el rol.
3. Busca el ID de la AMI de Amazon Linux 2023 en tu región y anótalo.
4. Busca el ID de tu VPC por defecto y anótalo.

**Resultado esperado:** tienes anotados un `ami-...` y un `vpc-...`.

## Parte 2: Par de llaves

5. Crea un par de llaves llamado `clave-<tu-nombre>` y guarda la llave privada en un archivo `.pem`.
6. Protege el archivo con `chmod 400`.
7. Lista los pares de llaves y confirma que aparece.

**Resultado esperado:** tienes un archivo `.pem` que solo tú puedes leer.

## Parte 3: Security group

8. Crea un security group llamado `web-<tu-nombre>` y anota su `sg-...`.
9. Consulta tu IP pública.
10. Permite el puerto 80 desde cualquier lugar.
11. Permite el puerto 22 **solo desde tu IP**.
12. Muestra las reglas del security group.

**Resultado esperado:** dos reglas: el 80 para `0.0.0.0/0` y el 22 para tu IP con `/32`.

## Parte 4: Lanzar la instancia

13. Crea el archivo `inicio.sh` con un script que instale el servidor web y publique una página con tu nombre.
14. Lanza una instancia `t3.micro` con tu AMI, tu par de llaves, tu security group, el perfil `ec2-lector-s3`, el script `inicio.sh` y el nombre `web-<tu-nombre>`. Anota su `i-...`.
15. Espera a que la instancia esté encendida.
16. Consulta su IP pública y anótala.

**Resultado esperado:** la instancia está en estado `running` y tiene IP pública.

## Parte 5: Probar el servidor web

17. Espera dos minutos y abre `http://<ip>` en el navegador.
18. Haz la misma petición desde la terminal con `curl`.

**Resultado esperado:** aparece la página con tu nombre.

## Parte 6: Entrar y probar el rol

19. Conéctate a la instancia por SSH.
20. Dentro de la instancia, pregunta con qué identidad trabaja:

    `aws sts get-caller-identity`

21. Dentro de la instancia, muestra el contenido de `reporte.txt` desde tu bucket.
22. Dentro de la instancia, intenta listar el contenido del bucket.
23. Dentro de la instancia, intenta listar los usuarios de IAM.
24. Sal de la instancia con `exit`.

**Resultado esperado:** el paso 20 muestra el rol `ec2-lector-s3` y no tu usuario. El paso 21 funciona. Los pasos 22 y 23 devuelven `AccessDenied`. En ningún momento corriste `aws configure` dentro de la instancia.

## Parte 7: Estados de la instancia

25. Apaga la instancia y consulta su estado hasta que sea `stopped`.
26. Enciéndela de nuevo, espera a que esté `running` y consulta su IP pública.
27. Compara la IP nueva con la del paso 16.

**Resultado esperado:** la IP pública cambió.

## Parte 8: Eliminar

Hazlo en este orden:

28. Termina la instancia y espera a que quede `terminated`.
29. Borra el security group.
30. Borra el par de llaves en AWS y el archivo `.pem` de tu computador.
31. Lista las instancias y confirma que ninguna está en `running` ni `stopped`.
32. Haz la limpieza de la actividad de roles: perfil de instancia, política, rol y bucket.

**Reto:** intenta borrar el security group antes de que la instancia quede `terminated` y lee el error.

## Entregable

Un archivo `evidencia.md` con:

- Captura o salida de los pasos 12, 16, 17, 20, 21, 22, 23, 27 y 31.
- El contenido de tu `inicio.sh`.
- La respuesta a las preguntas de reflexión.

**Nunca incluyas el contenido del archivo `.pem` en la evidencia ni lo subas a GitHub.**

## Preguntas de reflexión

1. En el paso 20, la instancia responde con un rol y no con un usuario. ¿De dónde sacó esas credenciales si nunca corriste `aws configure`?
2. ¿Por qué el puerto 80 puede estar abierto a todo internet y el 22 no?
3. La instancia pudo leer `reporte.txt` pero no listar el bucket. ¿Dónde se decidió eso?
4. ¿Por qué cambió la IP pública al apagar y encender? ¿Qué problema traería eso en un sitio web real?
5. EC2 es infraestructura como servicio (IaaS). En esta práctica, ¿qué fue responsabilidad tuya y qué fue responsabilidad de AWS?
6. Para tener este mismo servidor en una oficina, sin la nube, ¿qué habrías tenido que comprar y configurar?

## Criterios de evaluación

| Criterio | Cumple |
|---|---|
| La instancia responde por HTTP con la página propia | |
| El puerto 22 solo está abierto para su IP | |
| La instancia usa el rol y no tiene llaves configuradas | |
| Se evidencia una acción permitida y dos denegadas desde la instancia | |
| Se evidencia el cambio de IP al apagar y encender | |
| La evidencia no expone el `.pem` ni ninguna clave | |
| No quedaron instancias, security groups, llaves, roles ni buckets de la actividad | |
| Las preguntas están respondidas con sus propias palabras | |