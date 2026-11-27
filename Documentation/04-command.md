# Comandos de roles y perfiles de instancia

Los valores entre `< >` se reemplazan por los tuyos. Ejemplo: `<rol>` → `ec2-lector-s3`.

Los comandos con `file://` buscan el archivo en la carpeta donde estás parado. Escribe cada comando en una sola línea.

## 1. Preparar los archivos JSON

```bash
# Política de confianza: quién puede asumir el rol
cat > confianza.json << 'EOF'
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": { "Service": "ec2.amazonaws.com" },
    "Action": "sts:AssumeRole"
  }]
}
EOF

# Política de permisos: qué puede hacer (cambia <bucket> por el tuyo)
cat > permisos.json << 'EOF'
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": ["s3:GetObject"],
    "Resource": "arn:aws:s3:::<bucket>/*"
  }]
}
EOF

# Comprobar que un JSON está bien escrito
python3 -m json.tool confianza.json
python3 -m json.tool permisos.json
```

## 2. Crear un rol

```bash
# Crear con su política de confianza
aws iam create-role --role-name <rol> --assume-role-policy-document file://confianza.json

# Crear con una descripción
aws iam create-role --role-name <rol> --assume-role-policy-document file://confianza.json --description "<texto>"
```

## 3. Consultar roles

```bash
# Listar todos
aws iam list-roles --query "Roles[].RoleName" --output table

# Ver uno
aws iam get-role --role-name <rol>

# Ver solo su ARN
aws iam get-role --role-name <rol> --query "Role.Arn" --output text

# Ver solo su política de confianza
aws iam get-role --role-name <rol> --query "Role.AssumeRolePolicyDocument"
```

## 4. Permisos del rol

### Política en línea (escrita por ti en un JSON)

```bash
# Crear o reemplazar
aws iam put-role-policy --role-name <rol> --policy-name <nombre> --policy-document file://permisos.json

# Listar
aws iam list-role-policies --role-name <rol>

# Ver el contenido
aws iam get-role-policy --role-name <rol> --policy-name <nombre>

# Borrar
aws iam delete-role-policy --role-name <rol> --policy-name <nombre>
```

### Política administrada (ya hecha por AWS)

```bash
# Adjuntar
aws iam attach-role-policy --role-name <rol> --policy-arn arn:aws:iam::aws:policy/<NombrePolítica>

# Listar las adjuntas
aws iam list-attached-role-policies --role-name <rol>

# Quitar
aws iam detach-role-policy --role-name <rol> --policy-arn arn:aws:iam::aws:policy/<NombrePolítica>
```

## 5. Editar un rol

```bash
# Cambiar la descripción
aws iam update-role --role-name <rol> --description "<texto>"

# Reemplazar la política de confianza
aws iam update-assume-role-policy --role-name <rol> --policy-document file://confianza.json

# Agregar o cambiar una etiqueta
aws iam tag-role --role-name <rol> --tags Key=<clave>,Value=<valor>

# Ver las etiquetas
aws iam list-role-tags --role-name <rol>

# Quitar una etiqueta
aws iam untag-role --role-name <rol> --tag-keys <clave>
```

El nombre de un rol no se puede cambiar. Para renombrarlo hay que crear uno nuevo.

## 6. Perfiles de instancia

```bash
# Crear
aws iam create-instance-profile --instance-profile-name <perfil>

# Meter el rol dentro del perfil
aws iam add-role-to-instance-profile --instance-profile-name <perfil> --role-name <rol>

# Listar todos
aws iam list-instance-profiles --query "InstanceProfiles[].InstanceProfileName" --output table

# Ver uno y el rol que contiene
aws iam get-instance-profile --instance-profile-name <perfil>

# Ver en qué perfiles está un rol
aws iam list-instance-profiles-for-role --role-name <rol>

# Sacar el rol del perfil
aws iam remove-role-from-instance-profile --instance-profile-name <perfil> --role-name <rol>

# Borrar el perfil
aws iam delete-instance-profile --instance-profile-name <perfil>
```

## 7. Receta completa: rol para una EC2

```bash
aws iam create-role --role-name <rol> --assume-role-policy-document file://confianza.json
aws iam put-role-policy --role-name <rol> --policy-name <nombre> --policy-document file://permisos.json
aws iam create-instance-profile --instance-profile-name <perfil>
aws iam add-role-to-instance-profile --instance-profile-name <perfil> --role-name <rol>
aws iam get-instance-profile --instance-profile-name <perfil>
```

El segundo y el cuarto comando no responden nada cuando salen bien.

## 8. Orden para borrar

IAM no deja borrar un rol que todavía está en uso.

```bash
# 1. Sacar el rol del perfil y borrar el perfil
aws iam remove-role-from-instance-profile --instance-profile-name <perfil> --role-name <rol>
aws iam delete-instance-profile --instance-profile-name <perfil>

# 2. Borrar las políticas en línea y quitar las administradas
aws iam delete-role-policy --role-name <rol> --policy-name <nombre>
aws iam detach-role-policy --role-name <rol> --policy-arn arn:aws:iam::aws:policy/<NombrePolítica>

# 3. Borrar el rol
aws iam delete-role --role-name <rol>
```

## 9. Errores frecuentes

| Error | Qué significa | Qué hacer |
|---|---|---|
| `the following arguments are required: --assume-role-policy-document` | Faltó la política de confianza | Escribir el comando completo en una línea |
| `Unable to load paramfile file://...` | No encuentra el archivo JSON | Revisar la carpeta actual y el nombre del archivo |
| `MalformedPolicyDocument` | El JSON tiene un error o un campo inválido | Revisarlo con `python3 -m json.tool` |
| `EntityAlreadyExists` | Ya existe un rol o perfil con ese nombre | Usar otro nombre o borrar el anterior |
| `NoSuchEntity` | El rol, el perfil o la política no existe | Listar antes de consultar o borrar |
| `DeleteConflict` | El rol todavía tiene políticas o está en un perfil | Seguir el orden para borrar |
| `LimitExceeded` al agregar un rol a un perfil | El perfil ya tiene un rol | Sacar el rol anterior primero |
| `AccessDenied ... no identity-based policy allows` | A tu usuario le falta el permiso | Adjuntar la política que falta |
| `AccessDenied ... explicit deny in a service control policy` | La cuenta lo bloquea | No se arregla con permisos del usuario |
