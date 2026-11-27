# Comandos de EC2 con AWS CLI

Los valores entre `< >` se reemplazan por los tuyos. Ejemplo: `<nombre>` → `laura`.

Escribe cada comando en una sola línea. Los comandos con `file://` buscan el archivo en la carpeta donde estás parado.

## 1. Antes de empezar

```bash
# ¿Quién soy?
aws sts get-caller-identity

# ¿En qué región estoy? EC2 es regional
aws configure get region
```

## 2. Buscar la AMI

```bash
# ID de la AMI más reciente de Amazon Linux 2023 en tu región
aws ssm get-parameters --names /aws/service/ami-amazon-linux-latest/al2023-ami-kernel-default-x86_64 --query "Parameters[0].Value" --output text
```

El ID cambia según la región. No copies el de un compañero que trabaje en otra.

## 3. Red

```bash
# Ver la VPC por defecto
aws ec2 describe-vpcs --filters Name=isDefault,Values=true --query "Vpcs[].VpcId" --output text

# Ver las subredes
aws ec2 describe-subnets --query "Subnets[].[SubnetId,AvailabilityZone,MapPublicIpOnLaunch]" --output table

# Ver tu IP pública
curl -s https://checkip.amazonaws.com
```

## 4. Par de llaves

```bash
# Crear y guardar la llave privada
aws ec2 create-key-pair --key-name <clave> --query "KeyMaterial" --output text > <clave>.pem

# Proteger el archivo (obligatorio para SSH)
chmod 400 <clave>.pem

# Listar
aws ec2 describe-key-pairs --query "KeyPairs[].KeyName" --output table

# Borrar de AWS
aws ec2 delete-key-pair --key-name <clave>
```

## 5. Security group

```bash
# Crear (la descripción va sin tildes ni eñes)
aws ec2 create-security-group --group-name <grupo> --description "HTTP abierto y SSH solo desde mi IP" --query "GroupId" --output text

# Permitir HTTP (puerto 80) desde cualquier lugar
aws ec2 authorize-security-group-ingress --group-id <sg-id> --protocol tcp --port 80 --cidr 0.0.0.0/0

# Permitir SSH (puerto 22) solo desde tu IP
aws ec2 authorize-security-group-ingress --group-id <sg-id> --protocol tcp --port 22 --cidr <tu-ip>/32

# Ver las reglas
aws ec2 describe-security-groups --group-ids <sg-id> --query "SecurityGroups[].IpPermissions[].[FromPort,IpRanges[0].CidrIp]" --output table

# Quitar una regla
aws ec2 revoke-security-group-ingress --group-id <sg-id> --protocol tcp --port 22 --cidr <tu-ip>/32

# Borrar (ninguna instancia debe estar usándolo)
aws ec2 delete-security-group --group-id <sg-id>
```

## 6. Script de arranque (user data)

```bash
cat > inicio.sh << 'EOF'
#!/bin/bash
dnf install -y httpd
systemctl enable --now httpd
echo "<h1>Hola desde mi instancia EC2</h1>" > /var/www/html/index.html
EOF
```

Se ejecuta una sola vez, cuando la instancia arranca por primera vez.

## 7. Lanzar una instancia

```bash
aws ec2 run-instances --image-id <ami-id> --instance-type t3.micro --key-name <clave> --security-group-ids <sg-id> --iam-instance-profile Name=<perfil> --user-data file://inicio.sh --tag-specifications "ResourceType=instance,Tags=[{Key=Name,Value=<nombre-instancia>}]" --query "Instances[0].InstanceId" --output text
```

| Opción | Qué define |
|---|---|
| `--image-id` | La AMI: sistema operativo |
| `--instance-type` | El tamaño del servidor |
| `--key-name` | El par de llaves para SSH |
| `--security-group-ids` | El firewall |
| `--iam-instance-profile` | El rol que tendrá la instancia |
| `--user-data` | El script de arranque |
| `--tag-specifications` | El nombre de la instancia |

## 8. Consultar instancias

```bash
# Esperar a que esté encendida
aws ec2 wait instance-running --instance-ids <id>

# Listar todas, como tabla
aws ec2 describe-instances --query "Reservations[].Instances[].[InstanceId,State.Name,InstanceType,PublicIpAddress]" --output table

# Ver el estado de una
aws ec2 describe-instances --instance-ids <id> --query "Reservations[0].Instances[0].State.Name" --output text

# Ver su IP pública
aws ec2 describe-instances --instance-ids <id> --query "Reservations[0].Instances[0].PublicIpAddress" --output text

# Ver qué perfil de instancia tiene
aws ec2 describe-instances --instance-ids <id> --query "Reservations[0].Instances[0].IamInstanceProfile.Arn" --output text
```

## 9. Probar el servidor web

```bash
# Desde la terminal (es http, no https)
curl http://<ip>
```

También se puede abrir `http://<ip>` en el navegador. El servidor tarda uno o dos minutos en quedar listo después de que la instancia enciende.

## 10. Entrar por SSH

```bash
# Conectarse (el usuario de Amazon Linux es ec2-user)
ssh -i <clave>.pem ec2-user@<ip>

# Salir de la instancia
exit
```

Comandos útiles **dentro** de la instancia:

```bash
# ¿Con qué identidad trabaja la instancia?
aws sts get-caller-identity

# Leer un objeto del bucket
aws s3 cp s3://<bucket>/<archivo> -

# Ver el registro del script de arranque
sudo cat /var/log/cloud-init-output.log

# Ver si el servidor web está activo
systemctl status httpd
```

## 11. Apagar, encender y eliminar

```bash
# Apagar (conserva el disco; la IP pública cambia al volver a encender)
aws ec2 stop-instances --instance-ids <id>

# Encender
aws ec2 start-instances --instance-ids <id>

# Reiniciar
aws ec2 reboot-instances --instance-ids <id>

# Eliminar (definitivo)
aws ec2 terminate-instances --instance-ids <id>

# Esperar a que termine de eliminarse
aws ec2 wait instance-terminated --instance-ids <id>
```

## 12. Orden para borrar

1. Terminar la instancia y esperar a que quede `terminated`.
2. Borrar el security group.
3. Borrar el par de llaves en AWS y el archivo `.pem` local.

## 13. Errores frecuentes

| Error | Qué significa | Qué hacer |
|---|---|---|
| `InvalidAMIID.NotFound` | Esa AMI no existe en tu región | Buscar la AMI de nuevo en tu región |
| `InvalidKeyPair.NotFound` | El par de llaves no existe en esta región | Revisar el nombre con `describe-key-pairs` |
| `InvalidGroup.Duplicate` | Ya hay un security group con ese nombre | Usar otro nombre o borrar el anterior |
| `InvalidParameterValue ... iamInstanceProfile.name` | El perfil no existe o es muy reciente | Revisar el nombre y esperar unos segundos |
| `UnauthorizedOperation` | No tienes permiso o la cuenta lo bloquea | Leer el mensaje: política faltante o bloqueo de la cuenta |
| `DependencyViolation` al borrar el security group | Una instancia todavía lo usa | Esperar a que la instancia quede `terminated` |
| `Permission denied (publickey)` en SSH | Llave equivocada o usuario equivocado | Usar el `.pem` correcto y el usuario `ec2-user` |
| `UNPROTECTED PRIVATE KEY FILE` en SSH | El `.pem` tiene permisos abiertos | `chmod 400 <clave>.pem` |
| `Connection timed out` en SSH | El puerto 22 no está abierto para tu IP | Revisar la regla; tu IP pudo haber cambiado |
| La página no carga | Falta el puerto 80 o el servidor aún no arranca | Revisar el security group y esperar dos minutos |
| "No veo mi instancia" | La CLI o la consola están en otra región | Revisar `aws configure get region` |

## 14. Reglas de seguridad

- Nunca subas el archivo `.pem` a GitHub.
- Nunca abras el puerto 22 a `0.0.0.0/0`.
- Dentro de la instancia no corras `aws configure`: para eso está el rol.
- Termina las instancias que no estés usando.