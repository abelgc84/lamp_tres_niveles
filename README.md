# lamp_tres_niveles
### Balanceador de carga con infraestructura LAMP en tres niveles.

# Índice.
1. [Introducción.](#introducción)
2. [Infraestructura.](#infraestructura)
   * [Infraestructura de red.](#infraestructura-de-red)
   * [Creación de Instancias.](#creación-de-instancias)
4. [Configuración.](#configuración)
   * [Servidores Apache.](#servidores-apache)
   * [MySQL.](#mysql)
   * [Balanceador.](#balanceador)
5. [Certificado](#certificado)
6. [Ajustes de seguridad.](#ajustes-de-seguridad)
   * [Balanceador de carga.](#balanceador-de-carga)
   * [Servidores Apache.](#servidores-apache)
   * [Base de datos.](#base-de-datos)
8. [Screencash.](#screencash)

# Introducción.

En este proyecto se realiza el montaje de una pila LAMP en tres niveles sobre máquinas virtuales con AWS.

En un primer nivel tendremos el balanceador de carga.
En el segundo nivel tendremos dos servidores web en Backend.
En el tercer nivel tendremos el servidor de bases de datos.

# Infraestructura.

Como ya se ha mencionado usamos Amazon Web Services para realizar todo el despliegue de la infraestructura. A continuación se explicarán detalladamente los pasos a seguir para la creación de la infraestructura necesaria, tanto la red como los servidores.

## Infraestructura de red.

#### VPC

En primer lugar creamos una red virtual VPC (Virtual Private Cloud). Vamos al menú de **servicios de AWS**, elegimos **VPC** y pulsamos sobre el botón **Crear VPC**.

En este menú tenemos dos opciones, la opción **Solo la VPC** y la opción **VPC y más**. La diferencia entre ellas es que con la primera solo creamos la VPC y con la segunda podemos crear las subredes, las tablas de enrutamiento y las puertas de enlace desde el mismo menú. Como necesitamos crear todos esos recursos elegimos la opción **VPC y más**, de esta manera las asociaciones entre los recursos se harán de manera automática durante el proceso de creación y nos agilizará el trabajo.

Las opciones que nos encontramos durante la creación de la VPC son las siguientes:
* **Generar automáticamente las etiquetas de nombres**: Nos permite nombrar ahora los recursos creados con sus etiquetas de nombre. La marcamos activa para poder editar y generar los nombres desde esta ventana de creación y no tener que editar las etiquetas después. La llamaré AGC.
* **Bloque CIDR IPv4**: Elegimos la dirección IP inicial y el tamaño de la VPC mediante la notación CIDR. Le doy la IP 10.0.10.0/24.
* **Bloque CIDR IPv6**: Permite el uso de direcciones IPv6. Lo dejamos desactivado.
* **Tenencia**: Esta opción sirve para definir si las instancias que se lancen en la VPC se ejecutarán con hardware compartido o dedicado. La dejamos en predeterminado.
* **Número de zonas de disponibilidad**: Las zonas de disponibilidad son los centros de datos de AWS. Recomiendan usar al menos dos para entornos de producción para disponer de alta disponibilidad. En nuestro caso seleccionamos una.
* **Cantidad de subredes públicas**: La subred pública será la que tenga acceso a Internet, pondremos una subred pública donde alojaremos nuestro balanceador de carga.
* **Cantidad de subredes privadas**: La subred privada será la que no tenga acceso público, pondremos una sbred privada donde alojaremos nuestros servidores apache y base de datos.
* **Personalizar bloques CIDR de subredes**: Se puede elegir los bloques CIDR o dejar que se autoconfigure. Les asigno los bloques 10.0.10.0/25 y 10.0.10.128/25.
* **Gateway NAT**: Para acceder a la red privada es necesario crearle una Gateway NAT. Le ponemos una.
* **Puntos de enlace de la VPC**: Nos permite crear una VPC aislada y conectar los servicios de AWS como S3. Le ponemos ninguna.
* **Opciones de DNS**: Lo dejamos como viene por defecto, activado.

Pulsamos sobre el botón **Crear VPC** y habremos creado el siguiente esquema:

Una red virtual dividida en dos subredes. Cada subred tiene su propia tabla de enrutamiento, con su propia salida a internet. La Gateway NAT, que usan nuestras máquinas en Backend para salir a Internet, la usaremos solo para realizar las instalaciones necesarias en los servidores, cuando hayamos acabado eliminaremos la NAT puesto que no será necesario que esa subred tenga salida a internet.

![01](https://github.com/abelgc84/lamp_tres_niveles/assets/146434908/33d3deb9-57ae-4888-b1be-9ed2fb182621)



#### IP elástica

Necesitaremos una IP elástica para asignársela al balanceador de carga. En servicios VPC vamos al menú **Direcciones IP elásticas** y pulsamos sobre el botón **Asignar la dirección IP elástica**. No hay nada que configurar para las direcciones IP elásticas, simplemente la creamos y le damos un nombre.

![05](https://github.com/abelgc84/lamp_tres_niveles/assets/146434908/19c72b1e-6d65-406a-8d89-31cc475d84c1)



## Creación de Instancias

Para montar nuestra infraestructura necesitaremos cuatro máquinas. Una que actúe como balanceador de carga, dos de ellas actuarán como servidores web y la última será nuestra base de datos. El balanceador lo crearemos en la subred pública y tanto los servidores como la base datos las crearemos en la subred privada. 

Vamos al menú **servicios AWS** y elegimos **EC2**, pulsamos sobre el botón **Lanzar instancias** y accederemos al menú de creación de instancias. La creación de las cuatro máquinas es igual para todas: 
* **Nombre y etiquetas**: Nombramos la instancia y le ponemos etiquetas adicionales si fuera necesario.
* **Amazon Machine Image (AMI)**: Seleccionamos el sistema operativo. Usaremos Debian 12.
* **Tipo de instancia**: Hay varios tipos de instancias diferentes que especifican los recursos que tendrá. La dejamos en t2.micro. 
* **Par de claves**: Para poder conectarse de manera segura se necesita un par de claves. Elegimos par de claves vockey.
* **Configuración de red**: Editaremos la configuración para incluir las máquinas en la VPC, y cada una en su subred correspondiente. Además, deshabilitamos la asignación de IP pública automática a las máquinas. Aprovechando que estamos editando la configuración de red, creamos y nombramos los grupos de seguridad de cada máquina. La configuración de red quedaría de la siguiente manera (ejemplo con la máquina balanceador).

![03](https://github.com/abelgc84/lamp_tres_niveles/assets/146434908/9fbaf19b-9b82-4bde-9355-8a4847f5825a)

Las opciones de almacenamiento y los detalles avanzado durante la creación de la AMI los dejamos de manera predeterminada.


Como las instancias las hemos creado todas sin dirección IP pública no tenemos ninguna manera de acceder a ellas. Para poder acceder a ellas creamos una máquina temporal con IP pública automática que usaremos de puente para entrar en las otras.

Al final tenemos:

![04](https://github.com/abelgc84/lamp_tres_niveles/assets/146434908/fd513650-23a3-4fe2-b0bb-4cabd68e29fb)



# Configuración.

Antes de nada copiamos el archivo de nuestra clave en la máquina puente para poder usarlo desde allí y acceder a las otras.
```
scp -i labsuser.pem labsuser.pem admin@3.237.224.119:/$HOME
```
Para que esta copia sea válida y la clave pueda ser utilizada debemos cambiar los permisos para que no pueda ser usada ni por el grupo ni por otros.
```
chmod go-r labsuser.pem
```
![05](https://github.com/abelgc84/lamp_tres_niveles/assets/146434908/404337ad-c361-4eff-a5e0-53a271af69cc)

Por comodidad a la hora de trabajar, le cambiaremos el nombre a cada máquina para que en el prompt salga su nombre en lugar de una dirección IP. El cambio se hará visible con un reinicio o al salir y entrar en la máquina.
```
sudo hostnamectl set-hostname nombre_de_la_máquina
```
![06](https://github.com/abelgc84/lamp_tres_niveles/assets/146434908/0edd1698-2a1c-4619-a0cc-ae4e18cc263b)

La configuración que buscamos es la siguiente: 
* El balanceador de carga estará en un servidor apache que trabajará con https, con un certificado válido acreditado por Let´s Encrypt.
* Los servidores apache que tendrán nuestra aplicación web trabajarán con http, puesto que no hay necesidad de que la comunicación con el balanceador este cifrada.
* El servidor de base de datos, se comunicará con los servidores apache a través del puerto predeterminado de mariadb.



## Servidores Apache.

Instalamos apache y los módulos necesarios de php.
```
sudo apt update
sudo apt install -y apache2
sudo apt install php libapache2-mod-php php-mysql
```
![07](https://github.com/abelgc84/lamp_tres_niveles/assets/146434908/2ec92ec8-fccc-4c95-9739-40eef4b9e0f3)

Hacemos una copia del archivo 000-default.conf, para mantener la plantilla intacta, y la editamos para modificar el DocumentRoot.
```
cp 000-default.conf usuarios80.conf
sudo nano usuarios80.conf
```
![10](https://github.com/abelgc84/lamp_tres_niveles/assets/146434908/329878e5-a542-4810-a243-28002749bee4)


```
sudo a2ensite usuarios80.conf
sudo a2dissite 000-default.conf
```
![08](https://github.com/abelgc84/lamp_tres_niveles/assets/146434908/1ccb1892-7974-464c-a46e-58f252b673e1)


Instalamos git, creamos la carpeta que usaremos para alojar nuestra aplicación y clonamos el repositorio en ella.
```
sudo apt install -y git
sudo mkdir /var/www/html/usuario
sudo git clone https://github.com/josejuansanchez/iaw-practica-lamp.git /var/www/html/usuarios
```
![09](https://github.com/abelgc84/lamp_tres_niveles/assets/146434908/d8af3a15-ce19-4052-8be4-9f23e83709d0)

Editamos el archivo config.php de la aplicación para darle los parámetros de la base de datos.
```
sudo nano /var/www/html/usuarios/src/config.php
```
![11](https://github.com/abelgc84/lamp_tres_niveles/assets/146434908/aa4fe662-6db6-47d1-a400-9a3e7aa2e890)

Le damos la propiedad de los archivos al usuario predeterminado de apache.
```
sudo chown -R www-data:www-data /var/www/html/usuarios
```
![28](https://github.com/abelgc84/lamp_tres_niveles/assets/146434908/2f6fa0dc-6721-470d-9c64-5443c25438a9)

Reiniciamos el servicio.
```
sudo systemctl restart apache2
```

Instalamos el cliente de mariadb.
```
sudo apt install -y mariadb-client
```
![12](https://github.com/abelgc84/lamp_tres_niveles/assets/146434908/12f914c6-3e60-48c0-8d8c-ea4a75a492a6)

Copiamos la base de datos para llevarla al servidor de base de datos.
```
scp -i labsuser.pem admin@10.0.10.151:/var/www/html/usuarios/db/database.sql .
scp -i labsuser.pem database.sql admin@10.0.10.185:/$HOME
```
![14](https://github.com/abelgc84/lamp_tres_niveles/assets/146434908/8723d75a-3f66-4283-b496-d83fed180963)


## MySQL.

Instalamos el servidor mariadb.
```
sudo apt update
sudo apt install -y mariadb-server
```
![15](https://github.com/abelgc84/lamp_tres_niveles/assets/146434908/274f1195-adfe-49b5-962b-f2a136865338)

Editamos el archivo 50-server.cnf para modificar el bind-address.
```
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
```
![16](https://github.com/abelgc84/lamp_tres_niveles/assets/146434908/ce7d0012-134f-4d76-b4ac-bc6876e8cca2)

Cargamos el script de la base de datos, creamos el usuario y le damos los permisos para poder usar la base de datos de la aplicación.
```
sudo mysql -u root < $HOME/database.sql
sudo mysql -u root
CREATE USER 'usuarios_user@10.0.10.%' IDENTIFIED BY '1234';
GRANT ALL PRIVILEGES ON lamp_db.* TO 'usuarios_user@10.0.10.%';
FLUSH PRIVILEGES;
```
![17](https://github.com/abelgc84/lamp_tres_niveles/assets/146434908/ef9c2c92-eaa9-49ad-9890-0798faf27101)

El puerto por defecto que escucha mariadb es el 3306, por lo que tenemos que crear una regla nueva, en los grupos de seguridad de las máquinas, que permita la entrada de las conexiones por dicho puerto y cuyo origen venga de las otras máquinas. Muestro las dos reglas creadas para la máquina mysql. Las máquinas apache tendrán una sola regla cuyo origen sea la base de datos.
![18](https://github.com/abelgc84/lamp_tres_niveles/assets/146434908/cd57568c-40a0-4b88-b9fb-da9dcbf98044)

Hacemos una prueba de conexión para verificar que funciona correctamente. Intentamos acceder desde una de las máquinas apache.
```
mysql -u usuarios_user@10.0.10.% -p -h 10.0.10.185
```
![19](https://github.com/abelgc84/lamp_tres_niveles/assets/146434908/b37e11cf-dd3d-4abe-8fb9-50a9c123263f)



## Balanceador.

El balanceador será la entrada a nuestra aplicación por lo que le vamos a asociar nuestra IP elástica.
> En realidad haciendo esto al principio nos hubieramos podido ahorrar la máquina puente.

En el menú **Red y seguridad** vamos a **Direcciones IP elásticas**. Seleccionamos la IP elástica que tenemos creada y elegimos **Acciones**, **Asociar la dirección IP elástica**. Seleccionamos la opción para asociarla a una instancia y elegimos la instancia del balanceador.
![20](https://github.com/abelgc84/lamp_tres_niveles/assets/146434908/39c3dd16-b1ff-4cbd-b2a6-567a3b9f6ae6)

Instalamos apache.
```
sudo apt update
sudo apt install -y apache2
```
![21](https://github.com/abelgc84/lamp_tres_niveles/assets/146434908/37a6c0f5-8242-41fd-b618-7173e03aa88e)

Activamos los módulos de apache necesarios para la función de balanceador.
```
sudo a2enmod proxy
sudo a2enmod proxy_http
sudo a2enmod proxy_ajp
sudo a2enmod rewrite
sudo a2enmod deflate
sudo a2enmod headers
sudo a2enmod proxy_balancer
sudo a2enmod proxy_connect
sudo a2enmod proxy_html
sudo a2enmod lbmethod_byrequests
```
![22](https://github.com/abelgc84/lamp_tres_niveles/assets/146434908/44d006aa-e35f-49cf-9819-d2bed5d3ae99)


Hacemos una copia del archivo default-ssl.conf para editarlo y no tocar la plantilla. Editamos el nuevo archivo, activamos su configuración y desactivamos el que viene por defecto.
```
sudo cp default-ssl.conf balanceador.conf
sudo a2ensite balanceador.conf
sudo a2dissite 000-default.conf
```
![23](https://github.com/abelgc84/lamp_tres_niveles/assets/146434908/cd50b683-2af9-43e4-a7d4-afe2fc170306)

Añadimos al archivo de configuración las directivas Proxy y ProxyPass:
```
sudo nano balanceador.conf
```
![24](https://github.com/abelgc84/lamp_tres_niveles/assets/146434908/eafe976b-9582-441d-9a72-d6aec99c6ab9)

Activamos el módulo ssl de apache y reiniciamos el servicio.
```
sudo a2enmod ssl
sudo systemctl restart apache2
```
![25](https://github.com/abelgc84/lamp_tres_niveles/assets/146434908/c45c0486-7b6c-46e1-9ab2-6725003ebffb)



# Certificado

Usamos [My No-IP](https://www.noip.com/es-MX) para crear un nombre de dominio gratuito y asociarlo a la IP elástica de nuestro balanceador. El proceso no tiene ninguna complicación, crearse una cuenta e introducir los datos.
![26](https://github.com/abelgc84/lamp_tres_niveles/assets/146434908/18df30e0-9625-4db6-aabb-e0b298468d8d)

Para conseguir nuestro certificado autorizado usaremos [Let´s Encrypt](https://letsencrypt.org/es/how-it-works/). Es una autoridad de certificación que proporciona certificados gratuitos para el cifrado de seguridad de nivel de transporte.

Necesitamos demostrar que tenemos el control del dominio para poder obtener el certificado de Let´s Encrypt. Para ello usaremos [Certbot](https://certbot.eff.org/).

Instalamos snapd.
```
sudo apt update
sudo apt install snapd
```
![29](https://github.com/abelgc84/lamp_tres_niveles/assets/146434908/e78a52d1-60f0-41ff-b754-a1d3854540f5)

Instalamos snap core y lo actualizamos a su versión más reciente.
```
sudo snap install core
sudo snap refresh core
```
![30](https://github.com/abelgc84/lamp_tres_niveles/assets/146434908/7191d07d-e53a-4321-82e7-157b6a9e5928)

Eliminamos alguna versión previa, si existiera, e instalamos Certbot.
```
sudo apt remove certbot
sudo snap install --classic certbot
```
![31](https://github.com/abelgc84/lamp_tres_niveles/assets/146434908/16e1c487-ec0b-425d-a0cf-dbeae8c5e41f)

Creamos un alias para el comando certbot e instalamos certbot.
```
sudo ln -s /snap/bin/certbot /usr/bin/certbot
sudo certbot --apache
```
![32](https://github.com/abelgc84/lamp_tres_niveles/assets/146434908/24240b83-3c01-4fc7-9273-ad398917aba9)

Durante la instalación del certificado se nos realizará algunas preguntas:
* Dirección de correo electrónico.
* Aceptar terminos de uso.
* Se nos preguntará si queremos compartir nuestra dirección de correo.
* Y por último, nos preguntará nuestro nombre de dominio.
* En el caso de tener varios sitios activos, nos preguntará a cuál queremos vincular el certificado.

![27](https://github.com/abelgc84/lamp_tres_niveles/assets/146434908/7fd64b05-31a5-41e8-b3c4-76a00420bfeb)



# Ajustes de seguridad.

Para editar las ACL de las subredes nos vamos al menú **VPC**, **Seguridad**, **ACL de red**. Elegimos la VPC correspondiente, nos vamos a **Reglas de entrada** y pulsamos sobre **Editar reglas de entrada**. La ACL de red es la lista de control de acceso de la VPC, por lo que afectará a ambas subredes.

El único tráfico que debería entrar desde Internet a nuestra red es a través de las consultas por https al balanceador de carga, por lo que la única regla de entrada que deberiamos tener es para permitir la entrada por el puerto 443. Dejaremos activada una regla que permita la entrada por el puerto 22 para poder conectarnos por ssh, aunque en un entorno como AWS esta regla se podría omitir, y activarla solo cuando fuera necesario conectarse a las máquinas.

A continuación configuramos los grupos de seguridad de las máquinas.

## Balanceador de carga.

En nuestra red pública tenemos el servidor apache que actúa como balanceador de carga. Este servidor recibe peticiones desde internet a través del puerto 443 y se comunica con los servidores web de la red privada mediante el puerto 80.


## Servidores apache.

La única comunicación desde el exterior de esta red es a través del balanceador de carga por el puerto 80.

## Base de datos.
# Screencash.

A continuación se muestra en un breve [video](https://youtu.be/u7tZoemympM) con la aplicación web, usuarios, funcionando sobre esta infraestructura. En el video se introducen y eliminan usuarios, tanto desde un dispositivo ajeno como desde la propia máquina que realiza la grabación.









El documento técnico.
Además, deberás incluir un screencash donde se aprecie el funcionamiento de la aplicación desplegada.
En la entrega de la tarea deberás incluir la URL de tu repositorio y la URL de acceso a la aplicación desplegada.
