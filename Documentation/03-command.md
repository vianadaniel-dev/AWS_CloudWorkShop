## Comandos básicos

### Buckets

| Comando | Qué hace |
|---|---|
| `aws s3 mb s3://nombre-bucket` | Crea un bucket |
| `aws s3 ls` | Lista todos los buckets de la cuenta |
| `aws s3 rb s3://nombre-bucket` | Borra el bucket (debe estar vacío) |
| `aws s3 rb s3://nombre-bucket --force` | Borra el bucket con todo su contenido |

### Subir y descargar

| Comando | Qué hace |
|---|---|
| `aws s3 cp archivo.txt s3://nombre-bucket/` | Sube un archivo al bucket |
| `aws s3 cp s3://nombre-bucket/archivo.txt .` | Descarga un archivo a la carpeta actual |
| `aws s3 cp s3://nombre-bucket/archivo.txt -` | Muestra el contenido del archivo en la terminal |
| `aws s3 cp carpeta/ s3://nombre-bucket/carpeta/ --recursive` | Sube una carpeta completa |
| `aws s3 cp s3://bucket-a/archivo.txt s3://bucket-b/` | Copia un archivo de un bucket a otro |

### Listar, mover y borrar objetos

| Comando | Qué hace |
|---|---|
| `aws s3 ls s3://nombre-bucket/` | Lista el contenido del bucket |
| `aws s3 ls s3://nombre-bucket/ --recursive --human-readable --summarize` | Lista todo, con tamaños legibles y el total |
| `aws s3 mv archivo.txt s3://nombre-bucket/` | Mueve un archivo al bucket (lo borra del origen) |
| `aws s3 mv s3://nombre-bucket/viejo.txt s3://nombre-bucket/nuevo.txt` | Renombra un objeto |
| `aws s3 rm s3://nombre-bucket/archivo.txt` | Borra un archivo del bucket |
| `aws s3 rm s3://nombre-bucket/ --recursive` | Borra todo el contenido del bucket |

### Sincronizar

| Comando | Qué hace |
|---|---|
| `aws s3 sync carpeta/ s3://nombre-bucket/` | Sube solo los archivos nuevos o modificados |
| `aws s3 sync s3://nombre-bucket/ carpeta/` | Descarga solo los archivos nuevos o modificados |
| `aws s3 sync carpeta/ s3://nombre-bucket/ --delete` | Además borra del destino lo que ya no existe en el origen |

### Compartir

| Comando | Qué hace |
|---|---|
| `aws s3 presign s3://nombre-bucket/archivo.txt --expires-in 300` | Genera un enlace temporal (300 segundos) para descargar un archivo privado |

### Información del bucket

| Comando | Qué hace |
|---|---|
| `aws s3api get-bucket-location --bucket nombre-bucket` | Muestra en qué región está el bucket |
| `aws s3api head-object --bucket nombre-bucket --key archivo.txt` | Muestra tamaño, fecha y tipo de un objeto sin descargarlo |
| `aws s3api get-public-access-block --bucket nombre-bucket` | Muestra si el acceso público está bloqueado |

### Patrón de los nombres

Los comandos de `aws s3` imitan los de Linux: `ls` lista, `cp` copia, `mv` mueve, `rm` borra, `mb` crea bucket y `rb` lo borra. El orden siempre es origen primero y destino después.

### `s3` y `s3api`

- `aws s3`: comandos sencillos para el trabajo diario con archivos.
- `aws s3api`: acceso a todas las opciones del servicio, con comandos más largos.