# Logging CloudWatch Ec2 AWS Fluent-bit

### Objetivo:

Reflejar logs en Watchlogs para Ec2

* Servidor Web Nginx 
* Base de datos MySQL
* Drupal con PHP-FPM
* Fluent-bit

### Requisitos 

- Una cuenta con Ec2 en AWS
- Docker, Docker-compose y git instalados en Ec2
  
### Clonar el repositorio 

* Accedemos a nuestra máquina Ec2, comprobamos que tenemos los paquetes necesarios que hemos indicado anteriormente en los requisitos.
* Creamos un directorio de trabajo donde vamos a clonar el repositorio
* Clonamos el repositorio
```shell 
git clone https://github.com/CeliaGMqrz/ec2_fluent-bit_entorno_drupal.git
```

### Crear el directorio de datos 

Nos situamos dentro del repositorio.
Necesitamos crear estos directorios para contener los datos de la base de datos.

```shell 
mkdir -p storage/files storage/mysql
```

### Cargar las variables de entorno 

Cargamos las variables de entorno necesarias para el escenario, ya que están incluidas en el fichero `docker-compose.yaml`

!! No te olvides de modificar las variables para tu proyecto.

> Estas credenciales solo son muestras de prueba. Se recomienda usar contraseñas seguras.

Las encuentras en el fichero `.env` en el repositorio

### Construir las imágenes 

Necesitamos construir las imágenes para nuestro entorno, para ello:

Ejecuta el pequeño script `bash build_images.sh`

### CloudWatch

Desde el panel central de AWS vamos a indicar en el buscador `Cloudwatch`

![aws1.png](/images/aws1.png)

En el panel de la izquierda podemos ver `Grupos de registros` donde entraremos para crear nuestro grupo de logs.

![aws2.png](/images/aws2.png)

Creamos el grupo nuevo con un nombre identificativo, le podemos pasar el tiempo de vencimiento. Además aquí podemos añadir etiquetas o tags pero ahora mismo no vamos a agregar ninguna etiqueta, ya que de esto se encarga fluent-bit.

![aws3.png](/images/aws3.png)

Comprobamos en el panel que está creado

![aws4.png](/images/aws4.png)

### Configuración del entorno 

#### Fluent-bit 

[[Documentación oficial]](https://fluentbit.io/)

Fluent Bit procesa y reenvía los registros o logs, en este caso a CloudWatch de AWS. También es capaz de recopilar datos (métricas) y filtrar esos datos según necesitemos.

Es eficiente ya que cuenta con un alto rendimiento y un bajo uso de CPU y memoria.

Hemos usado una imagen dockerizada de fluent-bit especialmente para CloudWatch

`amazon/aws-for-fluent-bit`

En el fichero yml está expuesto de la siguiente forma:

```shell
  fluentd:
    container_name: fluentd
    image: amazon/aws-for-fluent-bit
    ports:
      - 24224:24224
      - 24224:24224/udp
    volumes:
      - ./fluent-bit/fluent-bit.conf:/fluent-bit/etc/fluent-bit.conf
    networks:
      - milocal

```

Como vemos utiliza un fichero de configuración en el que vamos a definir nuestro grupo de registro creado anteriormente.

```shell
. . . 
. . . 

[INPUT]
    Name              forward
    Listen            0.0.0.0
    Port              24224
    Buffer_Chunk_Size 1M
    Buffer_Max_Size   6M

[OUTPUT]
    Name cloudwatch_logs
    Match   *
    region eu-west-1
    log_group_name fluent-bit-cloudwatch
    log_stream_prefix fluent-bit-cloudwatch-celia-
    auto_create_group On
    workers 1

```

Donde indicamos el nombre de la salida, la región donde esta nuestra ec2. El nombre de grupo de registros y el prefijo que le pondremos a cada salida.

Si dejamos la opción `auto_create_group On` se crearan los grupos si no los tenemos creados previamente.

Indicamos el numero de workers también.

#### Entorno drupal

No vamos a entrar en detalles con los contenedores de drupal, nginx y mysql ya que el objetivo de este posts se enfoca en la obtención y salida de logs.


### Desplegar el entorno 

Una vez vista la configuración:

Lanzamos docker-compose up, en el directorio principal donde se encuentra el fichero yml

```shell
docker-compose up -d
```

### Contenedores en funcionamiento:

Comprobamos que los contenedores están en pleno funcionamiento

```shell
CONTAINER ID   IMAGE                       COMMAND                  CREATED          STATUS          PORTS                                                                                                    NAMES
5a8b82b87586   cgmarquez95/nx              "/bin/bash /assets/b…"   8 seconds ago    Up 7 seconds    0.0.0.0:80->80/tcp, :::80->80/tcp, 0.0.0.0:443->443/tcp, :::443->443/tcp                                 nginx_c
103e46a32ba4   cgmarquez95/dp              "docker-php-entrypoi…"   11 seconds ago   Up 8 seconds    9000/tcp                                                                                                 drupal_c
0c592a379fe4   cgmarquez95/mybd            "docker-entrypoint.s…"   13 seconds ago   Up 11 seconds   3306/tcp, 33060/tcp                                                                                      mysql_c
0122a379bca1   amazon/aws-for-fluent-bit   "/bin/sh -c /entrypo…"   13 seconds ago   Up 11 seconds   2020/tcp, 0.0.0.0:24224->24224/tcp, 0.0.0.0:24224->24224/udp, :::24224->24224/tcp, :::24224->24224/udp   fluentd

```


### Drupal desde navegador 
Si accedemos a la ip de nuestra ec2 comprobamos que está funcionando correctamente y se nos muestra el instalador de drupal.

![drupal.png](/images/drupal.png)


### Muestra de registros en CloudWatch

Una vez realizado todo el proceso vamos a nuestro CloudWatch y podemos ver que se nos ha creado los logs correspondientes a cada contenedor en este caso

* nginx
* mysql
* drupal

![awsfinal.png](/images/awsfinal.png)


Si entramos en los eventos de registro podemos verlo en formato JSON.

![logsdrupal.png](/images/logsdrupal.png)

![logsnginx.png](/images/logsnginx.png)




