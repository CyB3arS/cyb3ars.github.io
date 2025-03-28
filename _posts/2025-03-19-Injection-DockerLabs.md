---
title: Injection
description: Maquina de DockerLabs de dificultad muy facil
author: cyb3ar
date: 2025-03-19
categories: [DockerLabs, VeryEasy]
tags: [sql-injection,enumeration,permissions]
pin: true
math: true
mermaid: true
image:
  path: /assets/images/Dockerlabs/VeryEasy/Injection/Banner/Injection.png
  alt: 
---

## Pasos Iniciales

- Descargar la maquina vulnerable de [DockerLabs](https://dockerlabs.es/)
- Teniendo la maquina en nuestro equipo debemos correr el siguiente comando para descomprimirla:

```bash
unzip injection.zip
```

- Procedemos a levantar nuestra maquina objetivo con el siguiente comando:

```bash
#Con esto damos permisos de ejecucion al archivo encargado de desplegarnos la maquina.
sudo chmod +x auto_deploy.sh
#Levantamos la maquina objetivo
sudo bash auto_deploy.sh maquina.tar
```

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->

> A partir de este momento ya tenemos nuestra maquina objetivo lista para comenzar a jugar con ella.
{: .prompt-info }

<!-- markdownlint-restore -->

----------------------------------------------------------------------------

## Reconocimiento y Enumeración:

Para iniciar con esta maquina debemos verificar conexión con la maquina objetivo. Para ello le lanzamos el comando ping:

```bash
ping -c 1 172.17.0.2
```

Con este comando podemos ver que la maquina esta activa y tenemos alcance hacia ella, además por el TTL que tiene el cual esta cercano a los 64 podemos pensar que es un equipo Linux, ya que Windows por lo general tiene un TTL cercano a los 128. 

![](/assets/images/Dockerlabs/VeryEasy/Injection/Pasted image 20250319203547.png)

Ahora que sabemos que el equipo esta activo procedemos a realizar un escaneo de puertos, servicios y versiones utilizando la herramienta Nmap.

```bash
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 172.17.0.2
```

Este comando al ejecutarlo con estos parámetros le estamos indicando lo siguiente:

```bash
-p- Escanea los 65535 puertos existentes
--open Reporta solo los puertos abiertos
-sS Realiza un stealth Scan el cual no completa el three-way handshake (SYN / SYN-ACK / RST)
--min-rate 5000 Procesa no menos de 5000 paquetes por segundo
-vvv Permitimos que nos retorne la salida conforme va procesando
-n Desactivamos resolucion de DNS
-Pn Desactivamos el HostDiscovery
```

---------------------------------------------------------------------------------

### Analizando Resultados del Escaneo

Al ejecutarlo podemos ver que la maquina tiene 2 puertos abiertos el 22 y el 80

![](/assets/images/Dockerlabs/VeryEasy/Injection/Pasted image 20250319203958.png)

### Identificación de versiones y posibles vulnerabilidades:

```bash
nmap -p 22,80 -sVC 172.17.0.2
```

![](/assets/images/Dockerlabs/VeryEasy/Injection/Pasted image 20250319204843.png)

Con este comando podemos ver que el servicio SSH corre en el puerto 22 con la versión OpenSSH 8.9p1 además confirmamos que el equipo es un Ubuntu Linux y que corre en el puerto 80 un servicio HTTP Apache 2.4.52. Por lo cual podemos usar la herramienta whatweb para extraer un poco de información sobre la misma:

```bash
whatweb 172.17.0.2
```

![](/assets/images/Dockerlabs/VeryEasy/Injection/Pasted image 20250319205203.png)

Podemos hacer una enumeración en búsqueda de posibles directorios o archivos usando la herramienta feroxbuster:

```bash

feroxbuster --url http://172.17.0.2 -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -t 200 -d 0 -x php,html,txt

```

Vemos que nos encuentra el archivo index.php y config.php por ello vamos en el navegador a ver que es:

![](/assets/images/Dockerlabs/VeryEasy/Injection/Captura de pantalla 2025-03-19 210903.png)

La web a la que nos lleva es la siguiente:

![](/assets/images/Dockerlabs/VeryEasy/Injection/Pasted image 20250319205453.png)

## Explotación 

Al ingresar al sitio nos encontramos con esta pagina la cual nos hace pensar que debe existir alguna base de datos corriendo por abajo contra la cual se validan los datos de inicio de sesión por lo cual deberíamos verificar la posibilidad de que no este bien programada permitiendo que se acontezca un SQL Injection.

![](/assets/images/Dockerlabs/VeryEasy/Injection/Pasted image 20250319205924.png)

Al probar con la SQL Injection efectivamente no esta bien sanitizada y logramos obtener acceso:

![](/assets/images/Dockerlabs/VeryEasy/Injection/Pasted image 20250319210022.png)

Al verificar la información que nos ha devuelto esta pagina vemos un posible usuario Dylan y una contraseña: KJSDFG789FGSDF78

Como teníamos presente desde el escaneo existe un puerto 22 abierto el cual es un servicio SSH por lo cual vamos a probar las credenciales que acabamos de encontrar con suerte y nos podemos conectar:

![](/assets/images/Dockerlabs/VeryEasy/Injection/Pasted image 20250319211755.png)

Y efectivamente tenemos acceso al equipo y estamos conectados como Dylan.

## Elevación de Privilegios (Escalada)

```bash

find / -perm -4000 -user root 2>/dev/null

```

Al ejecutar este comando buscamos todos aquellos binarios con permisos SUID con los cuales podemos aprovecharnos de ellos para elevar nuestros privilegios 

![](/assets/images/Dockerlabs/VeryEasy/Injection/Pasted image 20250319212410.png)

Al verificar el binario /usr/bin/env vemos que tiene permisos SUID y al verificarlo con [GTOBins](https://gtfobins.github.io/gtfobins/env/) vemos que es explotable:

![](/assets/images/Dockerlabs/VeryEasy/Injection/Pasted image 20250319212921.png)

Por lo cual procedemos ha realizar la explotación y elevación de privilegios:

![](/assets/images/Dockerlabs/VeryEasy/Injection/Pasted image 20250319213401.png)

Y con esto hemos terminado la maquinita..
