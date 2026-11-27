# EC2

## ¿Qué es?

EC2 (Elastic Compute Cloud) es el servicio de servidores virtuales de AWS. Permite crear una máquina con sistema operativo, CPU, memoria y disco en pocos minutos, usarla el tiempo que se necesite y apagarla o eliminarla después. Cada máquina creada se llama **instancia**.

## ¿Para qué sirve?

- **Correr aplicaciones, APIs o sitios web** que necesitan un servidor.
- **Probar sin comprar hardware:** se crea, se usa y se elimina.
- **Ajustar la capacidad:** cambiar el tamaño o la cantidad de servidores según la demanda.

## Conceptos de cómputo

- **Instancia:** una máquina virtual concreta. Es "tu servidor".
- **AMI (Amazon Machine Image):** la plantilla con el sistema operativo y el software inicial. Su ID cambia según la región.
- **Tipo de instancia:** el tamaño del servidor, por ejemplo `t3.micro`. Define CPU, memoria y red.
- **Volumen EBS:** el disco virtual de la instancia.
- **Par de llaves:** una llave pública que guarda AWS y una privada (`.pem`) que guardas tú. Sirve para entrar por SSH.
- **User data:** un script que la instancia ejecuta sola la primera vez que arranca.
- **Perfil de instancia:** el contenedor que le entrega un rol de IAM a la instancia.

## Conceptos de red

- **VPC (Virtual Private Cloud):** tu red privada dentro de AWS. Toda instancia vive en una.
- **Subred:** una parte de la VPC, ubicada en una zona de disponibilidad.
- **IP pública:** la dirección con la que se llega a la instancia desde internet. Cambia si la instancia se detiene y se vuelve a iniciar.
- **IP privada:** la dirección de la instancia dentro de la VPC.
- **Security group:** el firewall de la instancia. Define qué tráfico puede entrar.
- **Puerto:** la "puerta" por la que entra cada tipo de tráfico. El 22 es SSH y el 80 es HTTP.
- **SSH (Secure Shell):** el protocolo para abrir una terminal remota cifrada en la instancia.

## Cómo leer un tipo de instancia

En `t3.micro`:

| Parte | Valor | Significa |
|---|---|---|
| Familia | `t` | Uso general |
| Generación | `3` | La versión de esa familia |
| Tamaño | `micro` | Cuánta CPU y memoria tiene |

## Estados de una instancia

| Estado | Qué significa | ¿Cobra? |
|---|---|---|
| `pending` | Está arrancando | No |
| `running` | Está encendida | Sí, instancia y disco |
| `stopped` | Está apagada, conserva el disco | Solo el disco |
| `terminated` | Fue eliminada, no se puede recuperar | No |

## Reglas clave

- **Una instancia cobra mientras está encendida.** Apagarla detiene el cobro del cómputo, pero el disco se sigue cobrando.
- **Terminar una instancia es definitivo.** Se pierde con su disco.
- **Un security group niega toda entrada por defecto.** Solo pasa lo que se permite de forma explícita.
- **El puerto 22 nunca se abre a todo internet** (`0.0.0.0/0`). Solo a tu IP.
- **El archivo `.pem` se descarga una sola vez.** Si se pierde, no hay SSH con esa llave.
- **El `.pem` nunca se sube a GitHub.**
- **EC2 es un servicio regional.** Una instancia creada en una región no se ve desde otra.
- **En EC2 el sistema operativo es tu responsabilidad:** actualizaciones, parches y lo que instales (modelo de responsabilidad compartida).

## EC2 y un servidor tradicional

| | Servidor propio (on-premise) | EC2 |
|---|---|---|
| Tiempo para tenerlo | Semanas: comprar, instalar, conectar | Minutos |
| Pago | Compra inicial del equipo | Por el tiempo de uso |
| Crecer | Comprar más hardware | Cambiar el tipo o lanzar más instancias |
| Si falla el equipo | Lo repara tu equipo | AWS reemplaza el hardware |
| Energía, red y espacio físico | Los gestionas tú | Los gestiona AWS |
| Sistema operativo y aplicación | Los gestionas tú | Los gestionas tú |

## Analogía

La AMI es el molde y la instancia es la galleta: de un mismo molde salen muchas galletas iguales. El tipo de instancia es el tamaño de la galleta. La VPC es el conjunto cerrado donde vive el servidor, el security group es el portero que decide quién entra y por cuál puerta, y el par de llaves es la llave de tu apartamento.

## Comandos básicos

| Comando | Qué hace |
|---|---|
| `aws ec2 run-instances ...` | Lanza una instancia |
| `aws ec2 describe-instances` | Lista las instancias |
| `aws ec2 stop-instances --instance-ids <id>` | Apaga una instancia |
| `aws ec2 start-instances --instance-ids <id>` | Enciende una instancia apagada |
| `aws ec2 terminate-instances --instance-ids <id>` | Elimina una instancia |
| `ssh -i <clave>.pem ec2-user@<ip>` | Entra a la instancia por SSH |

La lista completa está en [`02-command.md`](./02-command.md).

## En las cuentas nuevas (plan gratuito con proyectos)

- EC2 solo funciona en la región de tu proyecto.
- Cada hora de instancia encendida consume créditos. Una `t3.micro` gasta muy poco, pero una instancia olvidada varios días sí se nota.
- Si los créditos se acaban, el acceso se cierra.

## Para qué lo usamos en la semana 1

Lanzamos una instancia con un servidor web, protegida por un security group, y le asignamos el rol `ec2-lector-s3`. Luego comprobamos que la instancia puede leer el bucket sin tener ninguna llave guardada. Es el laboratorio que une todo lo anterior: CLI, IAM, buckets y roles.