# Bucket

## ¿Qué es?

Un bucket es un contenedor de almacenamiento en Amazon S3 (Simple Storage Service), el servicio de archivos de AWS. Funciona como una carpeta principal en la nube donde se guardan archivos de cualquier tipo: imágenes, documentos, respaldos o reportes.

## ¿Para qué sirve?

- **Guardar archivos** sin administrar discos ni servidores.
- **Compartir archivos** entre aplicaciones y servicios de AWS.
- **Respaldar información** de forma duradera.
- **Alojar archivos estáticos**, como las imágenes de una página web.

## Conceptos

- **Bucket:** el contenedor principal. No se puede crear un bucket dentro de otro.
- **Objeto:** cada archivo guardado, junto con sus datos (tamaño, fecha, tipo).
- **Clave (key):** el nombre completo del objeto dentro del bucket, por ejemplo `reportes/2026/ventas.txt`.
- **Prefijo:** la parte de la clave que parece una carpeta, como `reportes/2026/`. S3 no tiene carpetas reales: las simula con el nombre.
- **ARN del bucket:** `arn:aws:s3:::nombre-bucket`. Con `/*` al final se refiere a todos sus objetos.

## Características

- **Nombre único global:** no pueden existir dos buckets con el mismo nombre en todo AWS, ni siquiera en cuentas distintas.
- **Privado por defecto:** nadie puede acceder a su contenido sin un permiso explícito.
- **Capacidad ilimitada:** el bucket no tiene tope de almacenamiento. Se paga solo por lo que se guarda.
- **Pertenece a una región:** se crea en una región específica, aunque su nombre sea global.
- **Sin versiones por defecto:** subir un archivo con el mismo nombre reemplaza el anterior.

## Reglas del nombre

- Entre 3 y 63 caracteres.
- Solo minúsculas, números y guiones.
- Debe empezar y terminar con letra o número.
- No se puede cambiar después de creado.

## Reglas clave

- **Un bucket se borra solo si está vacío.**
- **Hacer público un bucket es responsabilidad tuya**, no de AWS (modelo de responsabilidad compartida).
- **Leer un objeto y listar el bucket son permisos distintos:** `s3:GetObject` y `s3:ListBucket`.
- **Para compartir un archivo privado se usa un enlace temporal**, no se abre el bucket.

## Analogía

El bucket es un casillero, los objetos son lo que se guarda adentro y la política de IAM define quién tiene llave y qué puede hacer con ella.

## Comandos básicos

| Comando | Qué hace |
|---|---|
| `aws s3 mb s3://nombre-bucket` | Crea un bucket |
| `aws s3 ls` | Lista todos los buckets de la cuenta |
| `aws s3 cp archivo.txt s3://nombre-bucket/` | Sube un archivo al bucket |
| `aws s3 ls s3://nombre-bucket/` | Lista el contenido del bucket |
| `aws s3 rm s3://nombre-bucket/archivo.txt` | Borra un archivo del bucket |
| `aws s3 rb s3://nombre-bucket` | Borra el bucket (debe estar vacío) |

La lista completa está en [`02-command.md`](./02-command.md).

## Para qué lo usamos en la semana 1

Es el recurso que protegemos para demostrar el menor privilegio: el rol `ec2-lector-s3` solo puede leer objetos de este bucket y nada más.