# lamp_tres_niveles
### Balanceador de carga con infraestructura LAMP en tres niveles.

# Índice
1. [Introducción](#introducción)
2. [Explicación](#explicación)
   1. [Infraestructura de red](#infraestructura-de-red)
   2. [Creación de Instancias](#creación-de-instancias)
4. [Screencash](#screencash)

# Introducción.

En este proyecto se realiza el montaje de una pila LAMP en tres niveles sobre máquinas virtuales con AWS.

En un primer nivel tendremos el balanceador de carga.
En el segundo nivel tendremos dos servidores web en Backend.
En el tercer nivel tendremos el servidor de bases de datos.

# Explicación.

A continuación se explicará detalladamente los pasos a seguir para la realización de las diversas configuraciones.

## Infraestructura de red

Como ya se ha mencionado usamos Amazon Web Services para realizar todo el despliegue de la infraestructura.

En primer lugar creamos una VPC (Virtual Private Cloud). Vamos al menú de servicios de AWS y elegimos VPC, pulsamos sobre el botón *Crear VPC*.
La nombramos y elegimos una dirección de red privada. En este caso se llamara abel_gijon_lamp_tres_niveles y tendrá la dirección IP 192.168.1.0/24.


![01](https://github.com/abelgc84/lamp_tres_niveles/assets/146434908/cd39fbf6-cd23-434e-9a18-2bbd708ae3e5)

A continuación creamos dos subredes dentro de la VPC. Dentro del menú de *Nube Privada Virtual* seleccionamos el submenú *Subredes* y pulsamos sobre el botón *Crear Subred*.

En el menú de creación de subredes configuramos las dos subredes de la siguiente manera:

![02](https://github.com/abelgc84/lamp_tres_niveles/assets/146434908/ebd177b2-6ccf-498e-906b-d7485d46b4e3)

![03](https://github.com/abelgc84/lamp_tres_niveles/assets/146434908/27418050-5463-472f-ad13-a8ab958c73da)

![04](https://github.com/abelgc84/lamp_tres_niveles/assets/146434908/8573bfed-56c2-4522-b789-10970624dd4a)


## Creación de Instancias

## Screencash



Crear un repositorio público en github para el proyecto.
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
