# Notas Adicionales: Configuración del Emulador de AWS

Guía rápida de variables de entorno para apuntar la AWS CLI al emulador local (puerto `9297`).

---

## 1. Limpiar configuración previa

Elimina perfiles y credenciales activas en la sesión actual para evitar conflictos con el emulador:

```bash
unset AWS_PROFILE AWS_ACCESS_KEY_ID AWS_SECRET_ACCESS_KEY AWS_SESSION_TOKEN
```

## 2. Configurar archivos del emulador

Define las rutas personalizadas para la configuración y las credenciales proporcionadas por el emulador:

```bash
export AWS_CONFIG_FILE="$HOME/.aws/emulator/9297/config"
export AWS_SHARED_CREDENTIALS_FILE="$HOME/.aws/emulator/9297/emulator_credentials"
```

## 3. Endpoint local

Redirige todas las llamadas de la AWS CLI hacia el emulador local:

```bash
export AWS_ENDPOINT_URL="https://127.0.0.1:9297/"
```

## 4. Certificado SSL

Indica a la AWS CLI el certificado CA autofirmado del emulador para validar conexiones HTTPS locales:

```bash
export AWS_CA_BUNDLE="$HOME/.aws/emulator/9297/ca.pem"
```

## 5. Desactivar paginador

Evita que la AWS CLI abra visores interactivos (como `less`) al mostrar la salida de los comandos:

```bash
export AWS_PAGER=""
```

---

## Resumen: Configuración en un solo comando

Puedes copiar y pegar este bloque completo en tu terminal para configurar el entorno de una sola vez:

```bash
# Limpiar configuración anterior
unset AWS_PROFILE AWS_ACCESS_KEY_ID AWS_SECRET_ACCESS_KEY AWS_SESSION_TOKEN

# Configurar archivos del emulador
export AWS_CONFIG_FILE="$HOME/.aws/emulator/9297/config"
export AWS_SHARED_CREDENTIALS_FILE="$HOME/.aws/emulator/9297/emulator_credentials"

# Endpoint local
export AWS_ENDPOINT_URL="https://127.0.0.1:9297/"

# Certificado SSL
export AWS_CA_BUNDLE="$HOME/.aws/emulator/9297/ca.pem"

# Desactivar paginador
export AWS_PAGER=""
```

### Verificación

Para verificar que las variables se hayan configurado correctamente en tu sesión:

```bash
env | grep AWS_
```

```

---

## Resumen: Configuración en un solo comando

Puedes copiar y pegar este bloque completo en tu terminal para configurar el entorno de una sola vez:

```bash
# Limpiar configuración anterior
unset AWS_PROFILE AWS_ACCESS_KEY_ID AWS_SECRET_ACCESS_KEY AWS_SESSION_TOKEN

# Configurar archivos del emulador
export AWS_CONFIG_FILE="$HOME/.aws/emulator/9297/config"
export AWS_SHARED_CREDENTIALS_FILE="$HOME/.aws/emulator/9297/emulator_credentials"

# Endpoint local
export AWS_ENDPOINT_URL="https://127.0.0.1:9297/"

# Certificado SSL
export AWS_CA_BUNDLE="$HOME/.aws/emulator/9297/ca.pem"

# Desactivar paginador
export AWS_PAGER=""
```

### Verificación

Para verificar que las variables se hayan configurado correctamente en tu sesión:

```bash
env | grep AWS_
```