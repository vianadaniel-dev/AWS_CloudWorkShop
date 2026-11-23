# Semana 1

## Configuración inicial de AWS

Al terminar esta guía tendrás un usuario de trabajo, la AWS CLI instalada y tu terminal conectada a tu cuenta.

### Antes de empezar

- Una cuenta de AWS activa.
- MFA activado en tu cuenta.
- Saber cuál es la región de tu proyecto. La ves en la dirección de la consola: `...us-east-2.console.aws.amazon.com...` significa `us-east-2`.

### 1. Crear tu usuario de trabajo y sus llaves (en la consola)

1. Escribe **IAM** en la barra de búsqueda y entra al servicio.
2. Menú izquierdo: **Usuarios** → **Crear usuario**.
3. Nombre: por ejemplo `abraham-admin`. Siguiente.
4. Elige **Adjuntar políticas directamente**, busca `AdministratorAccess` y **márcala con el cuadro de selección**. Luego crea el usuario.
5. Entra al usuario recién creado → pestaña **Credenciales de seguridad** → **Crear clave de acceso**.
6. Caso de uso: **Interfaz de línea de comandos (CLI)**. Acepta el aviso y crea.
7. Descarga el archivo `.csv`. La clave secreta solo se muestra esta vez.

> Verifica en la pestaña **Permisos** del usuario que aparezca `AdministratorAccess`. Si no quedó marcada, el usuario no podrá hacer nada.

### 2. Instalar la AWS CLI v2

Primero revisa si ya la tienes:

```bash
aws --version
```

Si responde `aws-cli/2.x.x`, salta al paso 3. Si no, instálala.

**Linux (Ubuntu):**

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o awscliv2.zip
unzip awscliv2.zip
sudo ./aws/install
aws --version
```

Si `unzip` no existe: `sudo apt install unzip`.

> No uses `sudo apt install awscli`: puede instalar la versión 1 o una desactualizada.

**Windows y macOS:** sigue la guía oficial:
https://docs.aws.amazon.com/es_es/cli/latest/userguide/getting-started-install.html

Al terminar puedes borrar el instalador:

```bash
rm -r aws awscliv2.zip
```

### 3. Configurar la AWS CLI

```bash
aws configure
```

Pide cuatro datos:

| Dato | Qué escribir |
|---|---|
| AWS Access Key ID | El ID de tu llave, está en el `.csv` |
| AWS Secret Access Key | Tu clave secreta, está en el `.csv` |
| Default region name | La región de tu proyecto, por ejemplo `us-east-2` |
| Default output format | `json` |

### 4. Comprobar la conexión

```bash
aws sts get-caller-identity
```

Debe responder algo así:

```json
{
    "UserId": "AIDAXXXXXXXXXXXXXXXXX",
    "Account": "123456789012",
    "Arn": "arn:aws:iam::123456789012:user/abraham-admin"
}
```

El `Arn` debe terminar en `user/` seguido del nombre de tu usuario. Este comando es el "¿quién soy?" de AWS: córrelo siempre antes de crear algo.

### 5. Comprobar los permisos

```bash
aws iam list-users
```

Si muestra la lista de usuarios, tu usuario tiene permisos. Si responde `AccessDenied`, vuelve al paso 1 y revisa la política.

### 6. Ver o cambiar la configuración

```bash
# Ver toda la configuración actual
aws configure list

# Ver la región
aws configure get region

# Cambiar la región
aws configure set region us-east-2
```

### Dónde queda guardado

| Archivo | Qué contiene |
|---|---|
| `~/.aws/credentials` | Tus llaves de acceso |
| `~/.aws/config` | La región y el formato de salida |

Están fuera de la carpeta del proyecto, así que no se suben a GitHub por accidente.

### Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `aws: command not found` | La CLI no está instalada | Repetir el paso 2 |
| `InvalidClientTokenId` o `SignatureDoesNotMatch` | Llave mal copiada | Repetir `aws configure` |
| `AccessDenied ... no identity-based policy allows` | Al usuario le falta la política | Adjuntar `AdministratorAccess` |
| `NoRegion` | No se configuró región | `aws configure set region <región>` |
| "No veo mis recursos" | La CLI y la consola están en regiones distintas | Usar la misma región en ambas |

### Reglas de seguridad

- Nunca subas el `.csv` ni tus llaves a GitHub, ni las pegues en un chat.
- No guardes el `.csv` dentro de la carpeta del proyecto.
- Si una llave se expone, bórrala de inmediato en IAM y crea otra.