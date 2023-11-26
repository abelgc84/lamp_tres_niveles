# lamp_tres_niveles
### Balanceador de carga con infraestructura LAMP en tres niveles.

# Índice.
1. [Introducción.](#introducción)
2. [Infraestructura.](#infraestructura)
   * [Infraestructura de red.](#infraestructura-de-red)
   * [Creación de Instancias.](#creación-de-instancias)
4. [Configuración.](#configuración)
   * [Balanceador.](#balanceador)
   * [Apache1.](#apache1)
   * [Apache2.](#apache2)
   * [MySQL.](#mysql)
6. [Screencash.](#screencash)

# Introducción.

En este proyecto se realiza el montaje de una pila LAMP en tres niveles sobre máquinas virtuales con AWS.

En un primer nivel tendremos el balanceador de carga.
En el segundo nivel tendremos dos servidores web en Backend.
En el tercer nivel tendremos el servidor de bases de datos.

# Infraestructura.

A continuación se explicarán detalladamente los pasos a seguir para la creación de la infraestructura necesaria.

## Infraestructura de red.

Como ya se ha mencionado usamos Amazon Web Services para realizar todo el despliegue de la infraestructura.

#### VPC

En primer lugar creamos una red virtual VPC (Virtual Private Cloud). Vamos al menú de **servicios de AWS**, elegimos **VPC** y pulsamos sobre el botón **Crear VPC**.

En este menú tenemos dos opciones, la opción **Solo la VPC** y la opción **VPC y más**. La diferencia entre ellas es que con la primera solo creamos la VPC y con la segunda podemos crear las subredes, las tablas de enrutamiento y las puertas de enlace desde el mismo menú. Como tenemos que crear todos esos recursos elegimos la opción **VPC y más**, de esta manera las asociaciones entre los recursos se harán de manera automática y nos agilizará el trabajo.

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
* **Configuración de red**: Editaremos la configuración para incluir las máquinas en la VPC, y cada una en su subred correspondiente. Además, deshabilitamos la asignación de IP pública automática a las máquinas. Aprovechando que estamos editando la configuración de red, reamos y nombramos los grupos de seguridad de cada máquina. La configuración de red quedaría de la siguiente manera (ejemplo con la máquina balanceador).

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

Le cambiaremos el nombre a cada máquina para que en el prompt salga su nombre en lugar de una dirección IP. El cambio se hará visible con un reinicio o al salir y entrar en la máquina.
```
sudo hostnamectl set-hostname nombre_de_la_máquina
```
![06](https://github.com/abelgc84/lamp_tres_niveles/assets/146434908/0edd1698-2a1c-4619-a0cc-ae4e18cc263b)





## Servidores Apache.

Instalamos apache y los módulos necesarios de php.
```
sudo apt update
sudo apt install -y apache2
sudo apt install php libapache2-mod-php php-mysql
```
![07](https://github.com/abelgc84/lamp_tres_niveles/assets/146434908/2ec92ec8-fccc-4c95-9739-40eef4b9e0f3)

Hacemos el archivo default-ssl.conf, para mantener la plantilla intacta, y lo editamos.
```
cp default-ssl.conf usuarios.conf
sudo nano usuarios.conf
```
![10](https://github.com/abelgc84/lamp_tres_niveles/assets/146434908/bb2c77de-bf53-41c7-8fb8-8b37cfccda25)

```
sudo a2ensite usuarios.conf
sudo a2dissite 000-default.conf
```
![08](https://github.com/abelgc84/lamp_tres_niveles/assets/146434908/3022226f-4f70-49a4-8f7b-ca121acde597)

Instalamos git y clonamos el repositorio.
```
sudo apt install -y git
sudo mkdir /var/www/html/usuario
sudo git clone https://github.com/josejuansanchez/iaw-practica-lamp.git /var/www/html/usuarios
```
![09](https://github.com/abelgc84/lamp_tres_niveles/assets/146434908/d8af3a15-ce19-4052-8be4-9f23e83709d0)

Editamos el archivo config.php de la aplicación.
```
sudo nano /var/www/html/usuarios/src/config.php
```
![11](https://github.com/abelgc84/lamp_tres_niveles/assets/146434908/aa4fe662-6db6-47d1-a400-9a3e7aa2e890)

Instalamos el cliente de mariadb.
```
sudo apt install -y mariadb-client
```
![12](https://github.com/abelgc84/lamp_tres_niveles/assets/146434908/12f914c6-3e60-48c0-8d8c-ea4a75a492a6)

Activamos los módulos ssl de apache2.
```
sudo a2enmod ssl
```
![13](https://github.com/abelgc84/lamp_tres_niveles/assets/146434908/378a3516-507d-4fd8-98d5-3b0c11eff6ed)

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



## Balanceador.

Instalamos apache.
```
sudo apt update
sudo apt install -y apache2
```
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


Hacemos una copia del archivo 000-default.conf para editarlo y activarlo. 
```
cp 000-default.conf balanceador.conf
sudo a2ensite balanceador.conf
sudo a2dissite 000-default.conf
sudo nano balanceador.conf
```


Añadimos al archivo de configuración las directivas Proxy y ProxyPass:


Reiniciamos el servicio apache.
```
sudo systemctl restart apache2
```


# Screencash.




Crear la infraestructura en AWS, nombrando las máquinas con NombreAlumnoMáquina, por ejemplo, CarlosGonzalezBalanceador. Aprovisionar las máquinas para al menos instalar los servidores web y de Base de Datos.
Hacer pública la máquina de capa1 con una ip elástica.
Instalar un certificado válido con CertBot y Let´s Script, asociado a un nombre de dominio.
Crear una lista de control de acceso que solo permita el tráfico necesario en cada subred.
Ajustar los grupos de seguridad para proteger las instancias.
Desplegar la aplicación de gestión de usuarios en cada uno de los servidores de Backend.
OPCIONAL: Desplegar cualquier CMS en lugar de la aplicación de usuarios.
Crear un documento técnico en Markdown, denominado Readme.md, con la explicación de la práctica paso a paso. En dicho documento se deben incluir imágenes (capturas de pantalla) donde se pueda ver apreciar que cada servidor está corriendo en cada máquina. Este documento deberá tener:

Índice.
Introducción, explicando cual es el objetivo y en qué infraestructura vamos a trabajar.
Explicación paso a paso de las configuraciones realizadas, así como cualquier aclaración que se considere oportuna.
Es fundamental que el documento no contenga faltas de ortografía y tenga un formato adecuado.
El repositorio de GitHub deberá contener:

El documento técnico.
Además, deberás incluir un screencash donde se aprecie el funcionamiento de la aplicación desplegada.
En la entrega de la tarea deberás incluir la URL de tu repositorio y la URL de acceso a la aplicación desplegada.
