# IAM

## ¿Qué es?

IAM (Identity and Access Management) es el servicio de AWS que controla quién puede entrar a la cuenta y qué puede hacer dentro de ella. Decide **quién** puede hacer **qué acción** sobre **qué recurso**. Es un servicio global (no depende de la región) y no tiene costo.

## Conceptos

- **Usuario:** identidad para una persona o aplicación, con credenciales permanentes (contraseña o llaves de acceso).
- **Grupo:** conjunto de usuarios que comparten los mismos permisos.
- **Rol:** identidad sin contraseña que se asume de forma temporal. Lo usan servicios como EC2 para acceder a otros servicios sin guardar llaves.
- **Política:** documento JSON que lista permisos. Indica el efecto (`Effect`), la acción (`Action`) y el recurso (`Resource`).
- **Perfil de instancia:** contenedor que conecta un rol con una instancia EC2.
- **Llave de acceso:** par ID + clave secreta para usar AWS desde la CLI o el SDK. Nunca se sube a GitHub.

## Reglas clave

- **Todo está negado por defecto:** un usuario nuevo no puede hacer nada hasta que una política se lo permita.
- **Un `Deny` explícito siempre gana** sobre un `Allow`.
- **Menor privilegio:** dar solo los permisos mínimos para hacer la tarea.

## Analogía

IAM es la recepción de un edificio. El usuario es el carné de cada empleado, el grupo es el área a la que pertenece, la política es la lista de pisos a los que puede entrar y el rol es un carné de visitante que se presta por un rato.

## Comandos básicos

### Identidad

| Comando                       | Qué hace                                  |
| ----------------------------- | ----------------------------------------- |
| `aws sts get-caller-identity` | Muestra con qué identidad estoy conectado |
| `aws configure get region`    | Muestra la región configurada             |

### Usuarios

| Comando                                                         | Qué hace                                   |
| --------------------------------------------------------------- | ------------------------------------------ |
| `aws iam create-user --user-name <Nombre de usuario>`                           | Crea un usuario                            |
| `aws iam list-users`                                            | Lista todos los usuarios                   |
| `aws iam get-user --user-name <Nombre de usuario>`                              | Muestra los datos de un usuario            |
| `aws iam attach-user-policy --user-name <Nombre de usuario> --policy-arn <arn>` | Le adjunta una política al usuario         |
| `aws iam list-attached-user-policies --user-name <Nombre de usuario>`           | Lista las políticas del usuario            |
| `aws iam detach-user-policy --user-name <Nombre de usuario> --policy-arn <arn>` | Le quita una política al usuario           |
| `aws iam delete-user --user-name <Nombre de usuario>`                           | Borra el usuario (sin políticas ni llaves) |

### Llaves de acceso

| Comando                                                          | Qué hace                                 |
| ---------------------------------------------------------------- | ---------------------------------------- |
| `aws iam create-access-key --user-name <Nombre de usuario>`                      | Crea una llave de acceso para el usuario |
| `aws iam list-access-keys --user-name <Nombre de usuario>`                       | Lista las llaves del usuario             |
| `aws iam delete-access-key --user-name <Nombre de usuario> --access-key-id <id>` | Borra una llave                          |

### Roles

| Comando                                                                                                   | Qué hace                                     |
| --------------------------------------------------------------------------------------------------------- | -------------------------------------------- |
| `aws iam create-role --role-name <rol> --assume-role-policy-document file://confianza.json`               | Crea un rol con su política de confianza     |
| `aws iam list-roles`                                                                                      | Lista todos los roles                        |
| `aws iam get-role --role-name <rol>`                                                                      | Muestra los datos de un rol                  |
| `aws iam put-role-policy --role-name <rol> --policy-name <nombre> --policy-document file://permisos.json` | Le da permisos al rol (política en línea)    |
| `aws iam list-role-policies --role-name <rol>`                                                            | Lista las políticas en línea del rol         |
| `aws iam get-role-policy --role-name <rol> --policy-name <nombre>`                                        | Muestra el contenido de una política del rol |
| `aws iam delete-role-policy --role-name <rol> --policy-name <nombre>`                                     | Borra una política del rol                   |
| `aws iam delete-role --role-name <rol>`                                                                   | Borra el rol (sin políticas ni perfil)       |

### Perfiles de instancia

| Comando                                                                                        | Qué hace                                |
| ---------------------------------------------------------------------------------------------- | --------------------------------------- |
| `aws iam create-instance-profile --instance-profile-name <nombre>`                             | Crea un perfil de instancia             |
| `aws iam add-role-to-instance-profile --instance-profile-name <nombre> --role-name <rol>`      | Mete el rol dentro del perfil           |
| `aws iam list-instance-profiles`                                                               | Lista los perfiles de instancia         |
| `aws iam get-instance-profile --instance-profile-name <nombre>`                                | Muestra un perfil y el rol que contiene |
| `aws iam remove-role-from-instance-profile --instance-profile-name <nombre> --role-name <rol>` | Saca el rol del perfil                  |
| `aws iam delete-instance-profile --instance-profile-name <nombre>`                             | Borra el perfil                         |

### Políticas administradas por AWS

| Comando                                            | Qué hace                          |
| -------------------------------------------------- | --------------------------------- |
| `aws iam list-policies --scope AWS --max-items 20` | Lista políticas ya hechas por AWS |
| `aws iam get-policy --policy-arn <arn>`            | Muestra los datos de una política |

### Trucos para leer la salida

| Opción                       | Qué hace                                       |
| ---------------------------- | ---------------------------------------------- |
| `--output table`             | Muestra la respuesta como tabla en vez de JSON |
| `--query "Roles[].RoleName"` | Filtra la respuesta y deja solo lo que pides   |
| `aws iam <comando> help`     | Muestra la ayuda de cualquier comando          |

### Orden para borrar

IAM no deja borrar algo que todavía está en uso. El orden es el inverso al de creación:

1. Sacar el rol del perfil de instancia y borrar el perfil.
2. Borrar las políticas del rol y luego el rol.
3. Quitar políticas y llaves del usuario y luego borrar el usuario.

## Cómo leer un error AccessDenied

- **"no identity-based policy allows":** al usuario le falta el permiso. Se arregla adjuntando una política.
- **"explicit deny in a service control policy":** la cuenta lo bloquea. No se arregla con permisos del usuario.

## Limitaciones de las cuentas nuevas (plan gratuito con proyectos)

- No se pueden crear grupos ni agregar usuarios a grupos. En su lugar, la política se adjunta directo al usuario.
- No se puede dar contraseña de consola a un usuario IAM.
- Los recursos como EC2 solo funcionan en la región del proyecto.

## Para qué lo usamos en la semana 1

Creamos el usuario `ana` con permisos de solo lectura sobre EC2 y el rol `ec2-lector-s3`, que la instancia EC2 asume para leer el bucket sin guardar llaves.
