# AWS CLI

## ¿Qué es?

La AWS CLI (Command Line Interface) es la herramienta oficial para manejar AWS desde la terminal. Hace lo mismo que la consola web, pero con comandos en vez de clics: ambas llaman a las mismas APIs de AWS.

## ¿Para qué sirve?

- **Automatizar:** un comando se puede repetir, guardar en un script y compartir.
- **Documentar:** un comando escrito es una instrucción exacta; una serie de clics no.
- **Trabajar más rápido:** crear diez recursos es repetir una línea, no llenar diez formularios.

## Conceptos

- **Llave de acceso (access key):** par de ID y clave secreta que identifica a un usuario IAM desde la terminal. Es el equivalente a usuario y contraseña, pero para programas.
- **Access Key ID:** la parte pública de la llave. Empieza por `AKIA`.
- **Secret Access Key:** la parte secreta. Solo se muestra una vez, al crearla.
- **Región:** la zona geográfica donde se crean los recursos, por ejemplo `us-east-2` (Ohio).
- **Formato de salida:** cómo se muestran las respuestas: `json`, `table` o `text`.
- **Perfil:** un juego de llaves, región y formato guardado con un nombre. El que se usa cuando no se indica ninguno se llama `default`.

## Cómo funciona

1. Escribes un comando en la terminal.
2. La CLI lo firma con tu llave de acceso.
3. AWS verifica quién eres y si tienes permiso.
4. AWS ejecuta la acción y devuelve la respuesta.

No existe "iniciar sesión" ni "cerrar sesión": cada comando viaja firmado con las llaves del perfil que esté en uso.

## Estructura de un comando

```
aws <servicio> <acción> [opciones]
```

Ejemplo:

```bash
aws iam create-user --user-name ana
```

| Parte | Valor | Significa |
|---|---|---|
| Programa | `aws` | Llama a la CLI |
| Servicio | `iam` | Con qué servicio hablas |
| Acción | `create-user` | Qué quieres hacer |
| Opciones | `--user-name <Nombre de usuaroi>` | Los datos de la acción |

## Dónde se guarda la configuración

| Archivo | Qué contiene |
|---|---|
| `~/.aws/credentials` | Las llaves de acceso de cada perfil |
| `~/.aws/config` | La región y el formato de cada perfil |

Son archivos de texto en tu computador, fuera de la carpeta del proyecto.

## Analogía

La consola web es ir al banco y hacer el trámite en la ventanilla. La CLI es hacer el mismo trámite por la aplicación: más rápido y repetible, pero necesitas tu clave. La llave de acceso es esa clave, y el perfil es la cuenta con la que entras.

## Comandos básicos

### Instalación y versión

| Comando | Qué hace |
|---|---|
| `aws --version` | Muestra la versión instalada |
| `aws help` | Muestra la ayuda general |
| `aws <servicio> help` | Muestra los comandos de un servicio |
| `aws <servicio> <acción> help` | Muestra las opciones de una acción |

### Configuración

| Comando | Qué hace |
|---|---|
| `aws configure` | Configura llaves, región y formato del perfil por defecto |
| `aws configure list` | Muestra la configuración actual |
| `aws configure get region` | Muestra la región configurada |
| `aws configure set region <región>` | Cambia la región |
| `aws configure set output <formato>` | Cambia el formato de salida |

### Identidad

| Comando | Qué hace |
|---|---|
| `aws sts get-caller-identity` | Muestra con qué identidad estás conectado |

### Perfiles

| Comando | Qué hace |
|---|---|
| `aws configure --profile <perfil>` | Crea o modifica un perfil con otro nombre |
| `aws configure list-profiles` | Lista los perfiles guardados |
| `aws <comando> --profile <perfil>` | Ejecuta un comando con ese perfil |
| `export AWS_PROFILE=<perfil>` | Usa ese perfil en toda la terminal |
| `unset AWS_PROFILE` | Vuelve al perfil por defecto |

### Opciones que sirven en cualquier comando

| Opción | Qué hace |
|---|---|
| `--output table` | Muestra la respuesta como tabla |
| `--output text` | Muestra solo el texto |
| `--query "<filtro>"` | Deja solo la parte de la respuesta que pides |
| `--region <región>` | Ejecuta el comando en otra región |

## Tipos de error

| Error | Qué significa |
|---|---|
| `ParamValidation` | El comando está mal escrito. No llegó a AWS |
| `AccessDenied` | El comando llegó, pero no tienes permiso |
| `NoRegion` | El perfil no tiene región configurada |
| `InvalidClientTokenId` | La llave de acceso no es válida o fue borrada |

## Reglas clave

- **La CLI distingue mayúsculas de minúsculas** en los nombres de los parámetros.
- **La región de la CLI y la de la consola deben coincidir**, o no verás lo que creaste.
- **Los servicios globales, como IAM, no necesitan región.** Los regionales, como EC2, sí.
- **Las llaves nunca se suben a GitHub** ni se guardan en la carpeta del proyecto.

## Para qué lo usamos en la semana 1

Es la herramienta con la que hacemos toda la práctica: crear usuarios y roles en IAM, subir archivos a S3 y lanzar la instancia EC2. Lo que se configura aquí es lo que usa después el código TypeScript con el SDK, que lee las mismas llaves de `~/.aws/credentials`.