---
id: tutorial-ubuntu-on-windows-es
summary: Acceda al poder inigualable del terminal de Ubuntu, incluidas herramientas como SSH, apt y vim, directamente en su equipo con Windows 10.
categories: server
language: es
tags: tutorial,installation,windows,ubuntu,terminal
difficulty: 2
status: published
translator: Adolfo Jayme Barrientos <fitojb@ubuntu.com>
published: 2018-01-05
author: Graham Morrison <graham.morrison@canonical.com>
feedback_url: https://github.com/canonical-websites/tutorials.ubuntu.com/issues

---

# Instalación de Ubuntu en Windows 10
Duration: 1:00

## Visión de conjunto

Positive
: Switch language: ES | [EN](tutorial-ubuntu-on-windows)

El excepcional terminal de Ubuntu está [disponible gratuitamente][msubuntu] para Windows 10.

Cualquier usuario de Linux lo sabe: es en la consola donde sucede la magia. Es perfecta para gestionar archivos, programar, administrar equipos remotamente y centenares de tareas más.

El terminal de Ubuntu para Windows incluye muchas de las prestaciones que encontrará al utilizar el terminal en Ubuntu:

- Abanico insuperable de paquetes, actualizaciones y funciones de seguridad
- Bash, Z-Shell, Korn y otros intérpretes de órdenes, sin máquinas virtuales ni arranque dual
- Ejecute herramientas nativas como SSH, git, apt y dpkg directamente desde su equipo con Windows
- Una enorme comunidad de usuarios amables y accesibles

![screenshot](https://assets.ubuntu.com/v1/00e5322f-win10-ubuntu-trusted-app.png)

## Requisitos
Duration: 1:00

Necesitará un equipo x86-64 que ejecute Windows 10.

Debe actualizar Windows 10 como mínimo a la versión [Fall Creators][win10fall], publicada en octubre de 2017. Esta actualización incluye el **subsistema Windows para Linux**, necesario para ejecutar el terminal de Ubuntu.

![screenshot](https://assets.ubuntu.com/v1/dbc96044-win10-ubuntu-fall-update.png)

## Instalar Ubuntu para Windows 10
Duration: 2:00

Es posible instalar Ubuntu desde la Tienda de Microsoft:

1. Sírvase del menú Inicio para abrir la aplicación Microsoft Store.
1. Busque *Ubuntu* y seleccione el primer resultado, «Ubuntu», de fabricante Canonical Group Limited.
1. Pulse en el botón *Instalar*.

Ubuntu se descargará e instalará automáticamente. La aplicación Microsoft Store le informará del progreso de la operación. 

![screenshot](https://assets.ubuntu.com/v1/13ab8b2c-win10-ubuntu-store.png)

## Iniciar Ubuntu en Windows 10
Duration: 2:00

Ubuntu puede iniciarse del mismo modo que cualquier otro programa de Windows 10: por ejemplo, puede buscarlo y seleccionarlo a través del menú Inicio.

### Primer inicio

La primera vez que inicie Ubuntu, el sistema le informará que se encuentra «instalándose» y deberá esperar unos minutos.

Cuando finalice el proceso, se le solicitará un nombre de usuario y una contraseña específicos para la instalación de Ubuntu. No es necesario que estos datos de acceso sean los mismos que los que emplea para entrar en Windows 10. Una vez que culmine este paso, se encontrará en la consola Bash de Ubuntu.

![screenshot](https://assets.ubuntu.com/v1/2d30f071-win10-ubuntu-first-run.png)

¡Enhorabuena! Ha instalado y activado el terminal de Ubuntu en Windows 10. Ahora tiene a su alcance toda la potencia de la consola.

## Obtener ayuda
Duration: 1:00

Si necesita orientación para comenzar a utilizar el terminal de Ubuntu, eche un vistazo a la [documentación comunitaria][commdocs]. Si se ha estancado, tiene ayuda siempre a su disposición (en inglés):

* [Ask Ubuntu][askubuntu]
* [Foros de Ubuntu][forums]
* [Asistencia a través de IRC][ubuntuirc]

<!-- LINKS -->
[msubuntu]: https://www.microsoft.com/es-es/store/p/ubuntu/9nblggh4msv6
[getstartedcli]: https://help.ubuntu.com/community/UsingTheTerminal
[storelink]: ms-windows-store://pdp/?productid=9NBLGGH4MSV6&referrer=unistoreweb&scenario=click&webig=11a9a85f-44f0-4cf5-ac1f-d9e148f2c23b&muid=01A3F9D8DEC2605B1426F331DF03617B
[win10fall]:https://support.microsoft.com/es-es/help/4028685/windows-10-get-the-fall-creators-update
[commdocs]: https://help.ubuntu.com/community/UsingTheTerminal
[askubuntu]: https://askubuntu.com/
[forums]: https://ubuntuforums.org/
[ubuntuirc]: https://wiki.ubuntu.com/IRC/ChannelList
