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
En un primer nivel tendremos el balanceador de carga, expuesto a Internet.
En el segundo nivel tendremos dos servidores web en Backend, en una red privada sin salida a Internet.
En el terce nivel tendremos el servidor de bases de datos, en otra red privada sin salida a Internet.

# Explicación.

A continuación se explicará detalladamente los pasos a seguir para la realización de las diversas configuraciones.

## Infraestructura de red

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
