# Comandos de configuración de la AWS CLI

Los valores entre `< >` se reemplazan por los tuyos. Ejemplo: `<región>` → `us-east-2`.

## 1\. Instalación

```sh
# Ver si está instalada y qué versión
aws --version

# Ver dónde está instalada
which aws

# Instalar en Linux
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o awscliv2.zip
unzip awscliv2.zip
sudo ./aws/install

# Actualizar en Linux (con el instalador ya descargado y descomprimido)
sudo ./aws/install --update

# Borrar el instalador
rm -r aws awscliv2.zip
```

## 2\. Configuración inicial

```sh
# Configurar llaves, región y formato
u

# Ver la configuración actual y de dónde sale cada valor
aws configure list
```

3\. Comprobar la conexión

```sh
# ¿Quién soy?
aws sts get-caller-identity

# Solo el número de cuenta
aws sts get-caller-identity --query "Account" --output text

# Solo el ARN
aws sts get-caller-identity --query "Arn" --output text
```

## 4\. Región

```sh
# Ver la región configurada
aws configure get region

# Cambiar la región
aws configure set region <región>

# Usar otra región en un solo comando
aws ec2 describe-instances --region <región>

# Usar otra región en toda la terminal
export AWS_REGION=<región>
unset AWS_REGION
```

## 5\. Formato de salida

```sh
# Ver el formato configurado
aws configure get output

# Cambiar el formato: json, table o text
aws configure set output <formato>

# Usar otro formato en un solo comando
aws iam list-users --output table
```

&nbsp;

| Formato | Cuándo usarlo |
| :---- | :---- |
| `json` | Por defecto. Muestra todos los datos |
| `table` | Para leer o proyectar en clase |
| `text` | Para copiar un solo valor o usarlo en un script |

## 6\. Perfiles

```sh
# Crear o modificar un perfil
aws configure --profile <perfil>

# Listar los perfiles guardados
aws configure list-profiles

# Ver la configuración de un perfil
aws configure list --profile <perfil>

# Usar un perfil en un solo comando
aws sts get-caller-identity --profile <perfil>

# Usar un perfil en toda la terminal
export AWS_PROFILE=<perfil>

# Volver al perfil por defecto
unset AWS_PROFILE

# Cambiar la región de un perfil
aws configure set region <región> --profile <perfil>
```

## 7\. Archivos de configuración

```sh
# Ver los archivos
ls ~/.aws

# Ver la región y el formato de cada perfil
cat ~/.aws/config

# Ver qué perfiles tienen llaves, sin mostrar las llaves
grep "\[" ~/.aws/credentials
```

&nbsp;

| Archivo | Qué contiene |
| :---- | :---- |
| `~/.aws/credentials` | Las llaves de acceso de cada perfil |
| `~/.aws/config` | La región y el formato de cada perfil |

Para eliminar un perfil, se borra su bloque en ambos archivos.

> No uses `cat ~/.aws/credentials` mientras compartes pantalla: muestra tus claves secretas.

## 8\. Filtrar la salida con \--query

```sh
# Un solo campo
aws sts get-caller-identity --query "Arn"

# Un campo de cada elemento de una lista
aws iam list-users --query "Users[].UserName"

# Varios campos, como tabla
aws iam list-users --query "Users[].[UserName,CreateDate]" --output table

# Solo el primer elemento
aws iam list-users --query "Users[0]"

# Contar elementos
aws iam list-users --query "length(Users)"
```

## 9\. Ayuda

```sh
# Ayuda general
aws help

# Comandos de un servicio
aws iam help

# Opciones de una acción
aws iam create-user help
```

La ayuda se abre en un visor: se sale con la tecla `q`.

## 10\. Comodidad

```sh
# Que las respuestas largas no se abran en un visor
aws configure set cli_pager ""

# Ver qué envía y recibe la CLI, para depurar
aws sts get-caller-identity --debug
```

> `--debug` puede mostrar datos sensibles. No pegues su salida en un chat ni en un repositorio.

## 11\. Variables de entorno

Tienen prioridad sobre los archivos de configuración y duran hasta cerrar la terminal.

| Variable | Qué define |
| :---- | :---- |
| `AWS_PROFILE` | El perfil a usar |
| `AWS_REGION` | La región a usar |
| `AWS_DEFAULT_OUTPUT` | El formato de salida |

&nbsp;

```sh
# Ver cuáles están definidas (solo los nombres)
env | grep AWS | cut -d= -f1

# Quitar una
unset <VARIABLE>
```

## 12\. Orden de prioridad

Cuando un valor está definido en varios lugares, gana el primero de esta lista:

1. La opción en el comando (`--region`, `--profile`, `--output`).  
2. Las variables de entorno.  
3. Los archivos `~/.aws/credentials` y `~/.aws/config`.

## 13\. Errores frecuentes

| Error | Causa | Solución |
| :---- | :---- | :---- |
| `aws: command not found` | La CLI no está instalada | Instalarla |
| `Unable to locate credentials` | No se ha corrido `aws configure` | `aws configure` |
| `InvalidClientTokenId` | La llave no es válida o fue borrada | Crear otra llave y reconfigurar |
| `SignatureDoesNotMatch` | La clave secreta está mal copiada | Repetir `aws configure` |
| `NoRegion` | El perfil no tiene región | `aws configure set region <región>` |
| `The config profile (x) could not be found` | El perfil no existe | `aws configure list-profiles` |
| `ParamValidation` | El comando está mal escrito | Revisar opciones y mayúsculas |

&nbsp;