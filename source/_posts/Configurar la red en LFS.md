---
title: Configurar la red en LFS
layour: post
uuid: fasfasww2d-arr-11ed-fasfw-wqr2fg
tags: [LFS, spanish]
categories: LFS
index_img: https://img.164314.xyz/2024/10/3003a3d768ecde1d3a9922ab8980f736.png
date: 2024-010-8 18:07:08
---


# Configuración de RED

![](https://img.164314.xyz/2024/10/3003a3d768ecde1d3a9922ab8980f736.png)

Según *capítulo 9.5* *General Network configuration* de manual de instalación LFS nos indica como podríamos configurarlo la red. Siguiendo los pasos es fácil de configurar. 

¡Empezamos a configurar! 

En la terminal, usamos **ip link** para ver qué tarjetas de red tenemos y conocer sus nombres. Vemos que tenemos 4, de las cuales dos son las que necesitamos configurar: una para el acceso a Internet y la otra para una conexión dual (ping). Pero el problema es que no están activadas. Antes de activarlas, las configuramos.

![](https://img.164314.xyz/2024/10/c3b09575b8cab272fec7ffd7a4bac03c.png)


## Configuración de HostOnly(dual ping)

Para configurar tenemos que añadir un fichero de llamado *ifconfig.x* en **/etc/sysconfig** donde x es el nombre de la tarjeta. Creamos ese fichero utilizando **vim**, en mi caso enp0s8 es la tarjeta para dual ping. Creamos ese fichero y editamos de la siguiente forma:

![](https://img.164314.xyz/2024/10/a087d978bd24e9edbe06aba24a71a7ce.png)

No olvidar después de editar hay que guardarlo(shift + zz). Ya tenemos el fichero creado, pero aún no tenemos la conexión, tenemos que activarla, utilizando **ifup enp0s8**

![](https://img.164314.xyz/2024/10/ba5abc93c5ae14bfcc28d1887f2ee7e0.png)



Probamos ping a ver cómo nos queda:

- Host – Guest 

![](https://img.164314.xyz/2024/10/28f86739d5f3e6fd8e4f7ea2ae3644df.png)

- Guest – Host 

![](https://img.164314.xyz/2024/10/f10bd13cd2b83f91cf3e10201b6ba213.png)

**¡No hay error, la conexión entre Host y Guest ha establecido de forma correcta!**

**Esta captura es un resumen de configuración hasta ahora**

![](https://img.164314.xyz/2024/10/5099cab2545eeb01b9e1e09b5bbe0a1c.png)

## Configuración de NAT(Internet, manual)

Configurar NAT es similar a configurar HostOnly. Solo tendríamos que cambiar el nombre de la tarjeta de red y la IP a NAT. En nuestro caso, la IP que he asignado a NAT es 10.2.2.15 y el Gateway es 10.0.2.2. Aunque se puede configurar de forma automática utilizando DHCP, pero no lo tenemos instalado. En los siguientes pasos mostraré cómo podemos instalar DHCP y configurarlo.

![](https://img.164314.xyz/2024/10/0d2d5cf5015185a6b674edaaf072efa3.png)

**Este es un resumen de red**

![](https://img.164314.xyz/2024/10/f4b58f8f5f1a62590ca950bcf7f2ef9b.png)

![](https://img.164314.xyz/2024/10/31af410de9d24644b430eb396054c163.png)



Prueba de ping a **Google**

![](https://img.164314.xyz/2024/10/6841d89e80d731f40f041a83cc2db6d0.png)

## Resolver problema de UNKNOWN HOST

![](https://img.164314.xyz/2024/10/fb68d0429fce3d41e8d6a4bf9dd01865.png)

Este tipo de problema ocurre porque no tenemos configurado el DNS. Para solucionarlo, solo tendríamos que añadir dos líneas más en **/etc/resolv.conf**. Usamos **vim** para añadirlas:

*nameserver 8.8.8.8*

*nameserver 8.8.4.4*

![](https://img.164314.xyz/2024/10/24cf0024a5c6af9dcdd74a78efd1e566.png)

Probamos de nuevo a ping Google.com. Ya no hay problema. 

![](https://img.164314.xyz/2024/10/e315f179f533967daf0f84efc6ae44ac.png)


## Configuración de NAT (DHCP)

Para configurar la red de forma automática, se utiliza el paquete **DHCPCD**. Como no lo tenemos instalado, debemos instalarlo antes de proceder con la configuración.

**DHCPCD:** <https://www.linuxfromscratch.org/blfs/view/stable/basicnet/dhcpcd.html>

Seguir las instrucciones de instalación es sencillo. Ahora comenzaremos a configurar la red utilizando DHCP.

Los dos archivos anteriores (**ifconfig.x**) ya no son necesarios después de instalar **DHCPCD**, usamos **mv** para guardar una copia. Antes de editar la configuración, debemos desactivar las dos tarjetas de red usando **ifdown <nombre\_tarjeta>**.

En **/etc/dhcpcd.conf** están todas las configuraciones; podemos dejarlas como están o editarlas de forma manual.

- **Automático**: Ejecutar **dhcpcd** y reiniciar.
- **Manual**: Añadir dos líneas más para la configuración de dos tarjetas, ejecutar **dhcpcd** y reiniciar el equipo.

```
*# configuration enp0s17 and enp0s8*

*interface enp0s17*

*interface enp0s8*

*static ip\_address=192.168.56.15/24*
```

### *Resultado* 

![](https://img.164314.xyz/2024/10/c821af1031249b746671841d1f7ceb28.png)

![](https://img.164314.xyz/2024/10/d6a1e538a8db6a5bd5c7be619bde81b8.png)

![](https://img.164314.xyz/2024/10/9d35612d64093ac313ec04963078bbf0.png)

