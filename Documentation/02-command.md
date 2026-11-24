# Comandos de IAM con AWS CLI

Los valores entre `< >` se reemplazan por los tuyos. Ejemplo: `<usuario>` → `ana`.

Escribe cada comando en una sola línea.

## 1. Antes de empezar

```bash
# ¿Quién soy? Córrelo siempre antes de crear algo
aws sts get-caller-identity

# ¿En qué región estoy?
aws configure get region
```

Los comandos de configuración de la CLI están en [`01-CONFIGURACION`](../01-CONFIGURACION/02-command.md).

## 2. Usuarios

```bash
# Crear
aws iam create-user --user-name <usuario>

# Listar todos
aws iam list-users
aws iam list-users --query "Users[].UserName" --output table

# Ver uno
aws iam get-user --user-name <usuario>

# Ver solo su ARN
aws iam get-user --user-name <usuario> --query "User.Arn" --output text

# Renombrar
aws iam update-user --user-name <usuario> --new-user-name <nuevo-nombre>

# Borrar (antes hay que quitarle políticas y llaves)
aws iam delete-user --user-name <usuario>
```

## 3. Etiquetas de usuario

```bash
# Agregar o cambiar una etiqueta (Key y Value van con mayúscula inicial)
aws iam tag-user --user-name <usuario> --tags Key=<clave>,Value=<valor>

# Ver las etiquetas
aws iam list-user-tags --user-name <usuario>

# Quitar una etiqueta
aws iam untag-user --user-name <usuario> --tag-keys <clave>
```

Las etiquetas no se crean aparte: `tag-user` la crea y la asigna en un solo paso.

## 4. Permisos de un usuario

### Políticas administradas (ya hechas por AWS)

```bash
# Adjuntar
aws iam attach-user-policy --user-name <usuario> --policy-arn arn:aws:iam::aws:policy/<NombrePolítica>

# Listar las adjuntas
aws iam list-attached-user-policies --user-name <usuario>

# Quitar
aws iam detach-user-policy --user-name <usuario> --policy-arn arn:aws:iam::aws:policy/<NombrePolítica>
```

El ARN de una política de AWS siempre es `arn:aws:iam::aws:policy/` seguido del nombre. No lleva tu número de cuenta ni el nombre del usuario.

Lista siempre antes de quitar: si la política no está adjunta, `detach-user-policy` responde `NoSuchEntity`.

### Políticas en línea (escritas por ti en un JSON)

```bash
# Crear o reemplazar
aws iam put-user-policy --user-name <usuario> --policy-name <nombre> --policy-document file://<archivo>.json

# Listar
aws iam list-user-policies --user-name <usuario>

# Ver el contenido
aws iam get-user-policy --user-name <usuario> --policy-name <nombre>

# Borrar
aws iam delete-user-policy --user-name <usuario> --policy-name <nombre>
```

Los comandos con `file://` buscan el archivo en la carpeta donde estás parado.

## 5. Llaves de acceso

```bash
# Crear (la clave secreta solo se muestra esta vez)
aws iam create-access-key --user-name <usuario>

# Listar
aws iam list-access-keys --user-name <usuario>

# Desactivar sin borrar
aws iam update-access-key --user-name <usuario> --access-key-id <id> --status Inactive

# Borrar
aws iam delete-access-key --user-name <usuario> --access-key-id <id>
```

Cada usuario puede tener máximo dos llaves. La clave secreta no se puede volver a consultar: si se pierde, se crea otra llave y se borra la anterior.

## 6. Probar los permisos de otro usuario

```bash
# Guardar las llaves del otro usuario en un perfil con nombre
aws configure --profile <perfil>

# Comprobar quién eres con ese perfil
aws sts get-caller-identity --profile <perfil>

# Ejecutar cualquier comando como ese usuario
aws ec2 describe-instances --profile <perfil>

# Ponerle región al perfil si responde NoRegion
aws configure set region <región> --profile <perfil>
```

> Usa siempre `--profile` con las llaves de otro usuario. Si corres `aws configure` sin esa opción, sobrescribes las llaves de tu usuario administrador.

## 7. Buscar políticas de AWS

```bash
# Listar políticas administradas por AWS
aws iam list-policies --scope AWS --query "Policies[].PolicyName" --output table

# Buscar por nombre
aws iam list-policies --scope AWS --query "Policies[?contains(PolicyName, 'EC2')].PolicyName" --output table

# Ver los datos de una política
aws iam get-policy --policy-arn arn:aws:iam::aws:policy/<NombrePolítica>
```

Políticas que usamos en el curso:

| Política | Qué permite |
|---|---|
| `AdministratorAccess` | Todo. Solo para el usuario administrador |
| `AmazonEC2ReadOnlyAccess` | Ver recursos de EC2, sin crear ni borrar |
| `AmazonS3ReadOnlyAccess` | Ver y leer buckets y objetos |
| `ReadOnlyAccess` | Ver todo, sin modificar nada |

## 8. Roles y perfiles de instancia

Tienen su propia carpeta: [`04-ROLES`](../04-ROLES/02-command.md).

## 9. Grupos

No disponibles en las cuentas nuevas con proyectos (plan gratuito): AWS los bloquea. Se dejan como referencia para cuentas empresariales.

```bash
aws iam create-group --group-name <grupo>
aws iam attach-group-policy --group-name <grupo> --policy-arn arn:aws:iam::aws:policy/<NombrePolítica>
aws iam add-user-to-group --user-name <usuario> --group-name <grupo>
aws iam get-group --group-name <grupo>
aws iam remove-user-from-group --user-name <usuario> --group-name <grupo>
aws iam delete-group --group-name <grupo>
```

En las cuentas nuevas, la política se adjunta directo al usuario con `attach-user-policy`.

## 10. Opciones útiles en cualquier comando

| Opción | Qué hace |
|---|---|
| `--output table` | Muestra la respuesta como tabla |
| `--output text` | Muestra solo el texto, sin formato |
| `--query "<filtro>"` | Deja solo la parte de la respuesta que pides |
| `--profile <perfil>` | Ejecuta el comando con otro perfil |
| `help` | Muestra la ayuda: `aws iam create-user help` |

## 11. Cómo leer los nombres

| Prefijo | Qué hace |
|---|---|
| `create-` | Crea |
| `list-` | Lista todos |
| `get-` | Muestra uno |
| `update-` | Modifica |
| `attach-` / `put-` / `add-` | Agrega |
| `detach-` / `remove-` | Quita |
| `delete-` | Borra |

## 12. Orden para borrar un usuario

IAM no deja borrar algo que todavía está en uso.

1. Borrar sus llaves de acceso.
2. Quitar sus políticas administradas y borrar sus políticas en línea.
3. Borrar el usuario.
4. Eliminar su perfil de `~/.aws/credentials` y `~/.aws/config`, si lo configuraste.

## 13. Errores frecuentes

| Error | Qué significa | Qué hacer |
|---|---|---|
| `AccessDenied ... no identity-based policy allows` | A tu usuario le falta el permiso | Adjuntar la política que falta |
| `AccessDenied ... explicit deny in a service control policy` | La cuenta lo bloquea | No se arregla con permisos del usuario |
| `ParamValidation` | El comando está mal escrito | Revisar parámetros y mayúsculas |
| `NoSuchEntity` | Lo que buscas no existe o no está adjunto | Listar antes de quitar o borrar |
| `EntityAlreadyExists` | Ya existe algo con ese nombre | Usar otro nombre o borrar el anterior |
| `DeleteConflict` | Todavía tiene algo asociado | Seguir el orden para borrar |
| `LimitExceeded` al crear una llave | El usuario ya tiene dos llaves | Borrar una antes de crear otra |
| `NoRegion` | El perfil no tiene región | `aws configure set region <región> --profile <perfil>` |

## 14. Reglas de seguridad

- Nunca subas llaves de acceso a GitHub ni las pegues en un chat.
- No uses la cuenta raíz para el trabajo diario.
- Da solo los permisos mínimos para la tarea.
- Borra las llaves y los usuarios que ya no uses.