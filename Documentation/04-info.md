# Roles y perfiles de instancia

## ¿Qué es?

Un rol es una identidad de IAM que no tiene contraseña ni llaves permanentes. No pertenece a una persona: alguien o algo se lo "pone" por un rato, y mientras lo tiene puesto recibe credenciales temporales con los permisos del rol.

Un perfil de instancia es el contenedor que conecta un rol con una instancia EC2. Es la forma en que la instancia recibe el rol.

## ¿Para qué sirve?

- **Que un servicio use otro servicio sin guardar llaves:** una EC2 que lee un bucket, por ejemplo.
- **Evitar llaves permanentes dentro de servidores o código:** si no hay llave guardada, no hay llave que robar.
- **Dar permisos por tarea y no por persona:** el rol describe qué puede hacer quien lo use.

## Usuario y rol: la diferencia

| | Usuario | Rol |
|---|---|---|
| Para quién es | Una persona o aplicación fija | Quien lo asuma en ese momento |
| Credenciales | Permanentes (llave de acceso) | Temporales, se renuevan solas |
| Dónde se guardan | En `~/.aws/credentials` | En ningún archivo |
| Si se filtran | Sirven hasta que alguien las borre | Dejan de servir en pocas horas |
| Uso típico | Tu trabajo desde la terminal | Una EC2 o una Lambda que usa otros servicios |

## Conceptos

- **Asumir un rol:** ponerse el rol y recibir credenciales temporales.
- **Política de confianza:** dice **quién** puede asumir el rol. Es obligatoria: un rol no existe sin ella.
- **Política de permisos:** dice **qué** puede hacer quien tenga el rol puesto.
- **Principal:** el "quién" de la política de confianza. Para EC2 es `ec2.amazonaws.com`.
- **Credenciales temporales:** llaves que AWS genera al asumir el rol y que vencen solas.
- **STS (Security Token Service):** el servicio de AWS que entrega esas credenciales.
- **Política en línea:** escrita por ti y pegada a un solo rol. Se crea con `put-role-policy`.
- **Política administrada:** ya hecha por AWS y reutilizable. Se adjunta con `attach-role-policy`.
- **Perfil de instancia:** el contenedor que lleva el rol hasta la instancia EC2.

## Un rol necesita dos políticas

**1. Confianza: quién puede asumirlo** (`confianza.json`)

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": { "Service": "ec2.amazonaws.com" },
    "Action": "sts:AssumeRole"
  }]
}
```

Solo el servicio EC2 puede ponerse este rol.

**2. Permisos: qué puede hacer** (`permisos.json`)

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": ["s3:GetObject"],
    "Resource": "arn:aws:s3:::nombre-bucket/*"
  }]
}
```

Solo puede leer objetos de ese bucket. Nada más.

`"Version": "2012-10-17"` es la versión del lenguaje de políticas, no una fecha que se deba actualizar.

## Cómo funciona con EC2

1. Creas el rol con su política de confianza.
2. Le das permisos.
3. Creas un perfil de instancia y metes el rol adentro.
4. Lanzas la instancia EC2 indicando ese perfil.
5. EC2 asume el rol y deja las credenciales temporales disponibles dentro de la instancia.
6. La CLI y el SDK que corren en la instancia las usan solas, sin `aws configure`.
7. AWS renueva las credenciales antes de que venzan.

## Reglas clave

- **Un rol sin política de confianza no se puede crear.**
- **Un rol recién creado no tiene permisos:** todo está negado por defecto.
- **Un perfil de instancia contiene un solo rol.**
- **La consola web crea el perfil de instancia automáticamente; la CLI no.** Con la CLI hay que crearlo a mano.
- **Los cambios en los permisos del rol aplican de inmediato**, sin reiniciar la instancia.
- **Para borrar un rol** primero se saca del perfil de instancia y se le quitan las políticas.
- **Un perfil recién creado puede tardar unos segundos** en estar disponible para EC2.

## Analogía

Un usuario es el carné personal de un empleado: tiene su nombre y lo lleva siempre. Un rol es el chaleco de brigadista que está colgado en la pared: no es de nadie, se lo pone quien está autorizado y lo devuelve al terminar. La política de confianza es la lista de quiénes pueden usar el chaleco, la política de permisos es lo que el chaleco permite hacer, y el perfil de instancia es el gancho del que la EC2 lo toma.

## Comandos básicos

| Comando | Qué hace |
|---|---|
| `aws iam create-role --role-name <rol> --assume-role-policy-document file://confianza.json` | Crea el rol con su política de confianza |
| `aws iam put-role-policy --role-name <rol> --policy-name <nombre> --policy-document file://permisos.json` | Le da permisos al rol |
| `aws iam get-role --role-name <rol>` | Muestra los datos del rol |
| `aws iam create-instance-profile --instance-profile-name <perfil>` | Crea un perfil de instancia |
| `aws iam add-role-to-instance-profile --instance-profile-name <perfil> --role-name <rol>` | Mete el rol dentro del perfil |
| `aws iam get-instance-profile --instance-profile-name <perfil>` | Muestra el perfil y el rol que contiene |

La lista completa está en [`02-command.md`](./02-command.md).

## En las cuentas nuevas (plan gratuito con proyectos)

- Crear roles, políticas y perfiles de instancia está permitido.
- En la lista de roles aparecen algunos creados por AWS, como los que empiezan por `AWSServiceRoleFor` o están en la ruta `managed/`. No se modifican ni se borran.

## Para qué lo usamos en la semana 1

Creamos el rol `ec2-lector-s3`, que solo puede leer objetos de un bucket, y lo envolvemos en un perfil de instancia. En la práctica de EC2 lanzamos la instancia con ese perfil y comprobamos que puede leer el archivo sin tener ninguna llave guardada.
