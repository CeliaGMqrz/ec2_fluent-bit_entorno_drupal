# Logging Watchlogs Ec2 AWS Fluent-bit

### Objetivo:

Reflejar logs en Watchlogs para Ec2

* Servidor Web Nginx 
* Base de datos MySQL
* Drupal con PHP-FPM
* Fluent-bit

### Requisitos 

- Docker y Docker Compose instalados
- Una cuenta con Ec2 en AWS

### Clonar el repositorio 

```shell 
git clone https://github.com/CeliaGMqrz/ec2_fluent-bit_entorno_drupal.git
```

### Crear el directorio de datos 

```shell 
mkdir -p storage/files storage/mysql
```

### Cargar las variables de entorno 

No te olvides de modificar las variables para tu proyecto.

> Estas credenciales solo son muestras de prueba. Se recomienda usar contraseñas seguras.


Las encuentras en el fichero `.env` en el repositorio


### Construir las imágenes 

Ejecuta el pequeño script `bash build_images.sh`

### Desplegar el entorno 

Lanzamos docker-compose up, en el directorio principal donde se encuentra el fichero yml

```shell
docker-compose up -d
```

### Contenedores en funcionamiento:

```shell

```

### Drupal desde navegador 

![captura.png](/images/captura.png)

