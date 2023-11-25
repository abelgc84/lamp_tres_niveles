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

En primer lugar creamos una VPC (Virtual Private Cloud). Vamos al menú de *servicios de AWS*, elegimos *VPC* y pulsamos sobre el botón *Crear VPC*.
La nombramos y elegimos una dirección de red privada. En este caso se llamara abel_gijon_lamp_tres_niveles y tendrá la dirección IP 192.168.1.0/24.

![01](https://github.com/abelgc84/lamp_tres_niveles/assets/146434908/cd39fbf6-cd23-434e-9a18-2bbd708ae3e5)

#### Subredes

A continuación creamos dos subredes dentro de la VPC que acabamos de crear. Dentro del menú de *Nube Privada Virtual* seleccionamos el submenú *Subredes* y pulsamos sobre el botón *Crear Subred*.

En el menú de creación de subredes configuramos las dos subredes de la siguiente manera:

Elegimos la VPC a la que pertenecerán las subredes.

![02](https://github.com/abelgc84/lamp_tres_niveles/assets/146434908/ebd177b2-6ccf-498e-906b-d7485d46b4e3)

Nombramos las subredes y elegimos el tamaño que tendrá cada una.

![03](https://github.com/abelgc84/lamp_tres_niveles/assets/146434908/27418050-5463-472f-ad13-a8ab958c73da)

![04](https://github.com/abelgc84/lamp_tres_niveles/assets/146434908/8573bfed-56c2-4522-b789-10970624dd4a)

#### IP elástica

Necesitaremos una IP elástica para asignársela al balanceador de carga. En servicios VPC vamos al menú *Direcciones IP elásticas* y pulsamos sobre el botón *Asignar la dirección IP elástica*. No hay nada que configurar para las direcciones IP elásticas, simplemente la creamos y le damos un nombre.

![05](https://github.com/abelgc84/lamp_tres_niveles/assets/146434908/19c72b1e-6d65-406a-8d89-31cc475d84c1)

#### Gateway

Para que nuestras máquinas tengan salida a Internet y podamos hacer las instalaciones necesarias crearemos, de manera temporal, una *Gateway de Internet* y la asociaremos a una de las susbredes. Para crearla no hay más que ir al menú *Puertas de enlace de Internet* y pulsar sobre el botón *Crear Gateway de Internet*. La creación es tán fácil como asignarle un nombre.

![09](https://github.com/abelgc84/lamp_tres_niveles/assets/146434908/3ef7e685-2430-4588-85e5-f104e0f85ab7)

Una vez creada la asociamos con nuestra VPC, abrimos las *Acciones* y elegimos *Conectar a la VPC*.

![10](https://github.com/abelgc84/lamp_tres_niveles/assets/146434908/00dd2581-9208-463f-a1f4-0013b470c3ab)

Con nuestra puerta de enlace creada, vamos al menú *Tablas de enrutamiento* y añadimos una ruta a nuestra VPC para que el tráfico de red pueda salir a Internet.

![11](https://github.com/abelgc84/lamp_tres_niveles/assets/146434908/05ac8e6e-bdf7-4b03-b716-3be3b6c810d4)

![12](https://github.com/abelgc84/lamp_tres_niveles/assets/146434908/09f96bb2-4f03-402e-b5e7-4e5dd859ad95)

Ahora vamos al menú *Asociaciones de subredes* para asociar las dos subredes a esa misma tabla de enrutamiento.

![15](https://github.com/abelgc84/lamp_tres_niveles/assets/146434908/88c1cf43-ef47-4b28-a6d4-3e3e992e72e3)


## Creación de Instancias

Para montar nuestra infraestructura necesitaremos cuatro máquinas. Una que actúe como balanceador de carga, dos de ellas actuarán como servidores web y la última será nuestra base de datos. Tanto el balanceador como los servidores red las crearemos en la subred_apache y la base datos la crearemos en la subred_mysql. 

La creación de las cuatro máquinas es igual para todas: 
* La nombramos, cada una con su nombre.
* Elegimos la AMI, tendrán todas una debian 12.
* Seleccionamos el par de claves, vockey en este caso.
* Editamos la configuración red para incluir las máquinas en la VPC, cada máquina en su subred correspondiente y les deshabilitamos la asignación de una IP pública de manera automática.
* Nombramos también los grupos de seguridad para que se reconozcan con mayor facilidad.

![07](https://github.com/abelgc84/lamp_tres_niveles/assets/146434908/842a092a-6ec6-4105-adfe-12faef758d21)

Como las instancias las hemos creado todas sin dirección IP pública no tenemos ninguna manera de acceder a ellas. Para poder acceder a ellas iremos cambiando la asociación de la IP elástica conforme nos haga falta.

Al final tenemos:

FOTO DE LAS INSTANCIAS

# Configuración.

A realizar en cada máquina:

Asociamos la IP elástica a nuestra máquina y conectamos por ssh.

Si fuera necesario acceder a otra máquina podemos copiair nuestra clave por scp y acceder desde esta máquina. En caso de que la necesitaramos copiar, por seguridad, borramos el archivo antes de irnos.
```
scp -i labsuser.pem labsuser.pem admin@18.212.89.13:/$HOME
```
Para que esta copia sea válida y pueda ser utilizada debemos cambiar los permisos para que no pueda ser usada ni por el grupo no por otros.
```
chmod go-r labsuser.pem
```

![17](https://github.com/abelgc84/lamp_tres_niveles/assets/146434908/761a575d-1768-4f88-9ff2-2ec946991d08)


Le cambiaremos el nombre a cada máquina para que en el prompt salga su nombre en lugar de una dirección IP.
```
sudo hostnamectl set-hostname nombre_de_la_máquina
```

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
![18](https://github.com/abelgc84/lamp_tres_niveles/assets/146434908/5be82aaa-f5dc-4385-bfd0-c7e9e0383c59)

Hacemos una copia del archivo 000-default.conf para editarlo y activarlo. 
```
cp 000-default.conf balanceador.conf
sudo a2ensite balanceador.conf
sudo a2dissite 000-default.conf
sudo nano balanceador.conf
```
![19](https://github.com/abelgc84/lamp_tres_niveles/assets/146434908/400382c4-f1e9-48e1-a562-ea2588a1130a)

Añadimos al archivo de configuración las directivas Proxy y ProxyPass:
![20](https://github.com/abelgc84/lamp_tres_niveles/assets/146434908/82703a74-dd8a-46c1-bb9a-6728befe2913)

Reiniciamos el servicio apache.
```
sudo systemctl restart apache2
```

## Apache1.

Instalamos apache y los módulos necesarios de php.
```
sudo apt update
sudo apt install -y apache2
sudo apt install php libapache2-mod-php php-mysql
```



## Apache2.

Repetimos el proceso hecho en apache1.
![23](https://github.com/abelgc84/lamp_tres_niveles/assets/146434908/4e588b03-c760-4500-bdc7-8b9cb0acd540)



## MySQL.


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
