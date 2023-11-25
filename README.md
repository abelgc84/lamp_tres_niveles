# lamp_tres_niveles
### Balanceador de carga con infraestructura LAMP en tres niveles.

# Índice.
1. [Introducción.](#introducción)
2. [Infraestructura.](#infraestructura)
   1. [Infraestructura de red.](#infraestructura-de-red)
   2. [Creación de Instancias.](#creación-de-instancias)
4. [Configuración.](#configuración)
5. [Screencash.](#screencash)

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

#### Gateway NAT

Para que nuestras máquinas tengan salida a Internet y podamos hacer las instalaciones necesarias crearemos una Gateway NAT y la asociaremos a una de las susbredes. La creación de una Gateway NAT requiere que se le asocie una IP elástica, por lo que primero creamos nuestra IP elástica.

En servicios VPC vamos al menú *Direcciones IP elásticas* y pulsamos sobre el botón *Asignar la dirección IP elástica*. No hay nada que configurar para las direcciones IP elásticas, simplemente la creamos y le damos un nombre.

![05](https://github.com/abelgc84/lamp_tres_niveles/assets/146434908/19c72b1e-6d65-406a-8d89-31cc475d84c1)

Con nuestra IP elástica ya disponible vamos a servicios VPC, al menú *Gateways NAT* y pulsamos sobre el botón *Crear gateway NAT*. Para crearla simplemente la nombramos, le asociamos una subred y nuestra IP elástica.

![06](https://github.com/abelgc84/lamp_tres_niveles/assets/146434908/291ad96d-516e-4671-b5b1-eec636359c96)

## Creación de Instancias

Para montar nuestra infraestructura necesitaremos cuatro máquinas. Una que actúe como balanceador de carga, dos de ellas actuarán como servidores web y la última será nuestra base de datos. Tanto el balanceador como los servidores red las crearemos en la subred_apache y la base datos la crearemos en la subred_mysql. 

La creación de las cuatro máquinas es igual para todas: 
* La nombramos, cada una con su nombre.
* Elegimos la AMI, tendrán todas una debian 12.
* Seleccionamos el par de claves, vockey en este caso.
* Editamos la configuración red para incluir las máquinas en la VPC, cada máquina en su subred correspondiente y les deshabilitamos la asignación de una IP pública de manera automática.
* Nombramos también los grupos de seguridad para que se reconozcan con mayor facilidad.

![07](https://github.com/abelgc84/lamp_tres_niveles/assets/146434908/842a092a-6ec6-4105-adfe-12faef758d21)

Como las instancias las hemos creado todas sin dirección IP pública no tenemos ninguna manera de acceder a ellas. Para poder acceder ellas creamos una quinta máquina que, de manera temporal, usaremos como puente para entrar en las otras a través de ssh, a esta máquina le habilitamos la asignación automática de IP pública durante la creación.

Al final tenemos:

![08](https://github.com/abelgc84/lamp_tres_niveles/assets/146434908/0c4a2cca-7c17-4dd4-9c76-e8d676340e9b)

# Configuración.


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
