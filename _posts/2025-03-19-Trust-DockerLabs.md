---
title: Trust
description:
author: cyb3ar
date: 2025-03-19
pin: true
math: true
mermaid: true
image:
  path: /assets/images/dklabs/ve/Trust/Banner.png
  alt: 

machine: Trust
platform: dklabs
difficult: ve #ve VeryEase, e Easy, m Medium, h hard, vh veryhard, i insane
categories: [DockerLabs, VeryEasy]
tags: [Fuzzing,Web,enumeration,SSH Brute Force]
---

## Pasos Iniciales

- Descargar la maquina {{page.machine}} de [DockerLabs](https://dockerlabs.es/)
- Teniendo la maquina en nuestro equipo debemos correr el siguiente comando para descomprimirla:

```bash
unzip {{page.machine}}.zip
```

- Procedemos a iniciar maquina {{page.machine}} con el siguiente comando:

```bash
#Con esto damos permisos de ejecucion al archivo encargado de desplegarnos la maquina.
sudo chmod +x auto_deploy.sh
#Levantamos la maquina objetivo
sudo bash auto_deploy.sh {{page.machine}}.tar
```

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->

> A partir de este momento ya tenemos nuestra maquina objetivo lista para comenzar a jugar con ella.
{: .prompt-info }

<!-- markdownlint-restore -->

----------------------------------------------------------------------------

## Reconocimiento y Enumeración

Para iniciar con esta máquina debemos verificar conexión con la maquina objetivo. Para ello le lanzamos el comando ping:

```bash
ping -c 1 172.17.0.2
```

Con este comando podemos ver que la maquina esta activa y tenemos alcance hacia ella, además por el TTL que tiene el cual está cercano a los 64 podemos pensar que es un equipo Linux, ya que Windows por lo general tiene un TTL cercano a los 128. 

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Ping.png)

Ahora que sabemos que el equipo este activo procedemos a realizar un escaneo de puertos, servicios y versiones utilizando la herramienta Nmap.

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

---------------------------------------------------------------------------------------------------------

Al ejecutar el escaneo de puertos podemos ver que la maquina tiene 2 puertos abiertos el 22 y el 80

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Primer_Escaneo.png)

Procedemos a analizar los servicios para identificar posibles versiones vulnerables:

```bash
nmap -p 22,80 -sVC 172.18.0.2
```

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Escaneo_Puertos_Servicios.png)

Con este comando podemos ver que el servicio SSH corre en el puerto 22 con la versión OpenSSH 9.2p1 además confirmamos que el equipo es un Debian y que corre en el puerto 80 un servicio HTTP Apache 2.4.57. Por lo cual podemos usar la herramienta whatweb para extraer un poco de información sobre la misma:

```bash
whatweb 172.18.0.2
```

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Whatweb.png)

Podemos hacer una enumeración en búsqueda de posibles directorios o archivos usando la herramienta feroxbuster:

```bash
feroxbuster --url http://172.17.0.2 -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -t 200 -d 0 -x php,html,txt
```
![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Feroxbuster.png)

Vemos que nos encuentra el archivo index.html:

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Index_html.png)

Asi como el archivo secret.php:

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Secret.png))

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/SecretCodeSource.png)

Lo único que vemos acá que nos puede interesar es Mario un posible usuario, a nivel del código fuente no hay nada.

# Explotación 

Como conocemos un posible usuario vamos a probar con fuerza bruta conseguir la contraseña del usuario para conectarnos al servicio SSH. Para ello usaremos hydra con el siguiente comando:

```bash
hydra -l mario -P /usr/share/wordlists/rockyou.txt ssh://172.18.0.2
```

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Hydra1.png)

Hemos encontrado la contraseña "chocolate" con la cual nos podemos conectar por ssh al usuario de mario.

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/ssh.png)

Y efectivamente tenemos acceso al equipo y estamos conectados como Mario.

# Elevación de Privilegios (Escalada)

Comenzamos buscando todos aquellos binarios con permisos SUID de los cuales podemos aprovecharnos para elevar nuestros privilegios:

```bash
find / -perm -4000 -user root 2>/dev/null
```

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Perms.png)

Sin embargo, no tenemos nada que podamos aprovechar, por lo tanto seguimos buscando

Revisamos Permisos SUDO

```bash
sudo -l
```

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/PermisosSUDO.png)

Y nos encontramos lo siguiente el usuario Mario tiene permisos de ejecución del binario /usr/bin/vim para ejecutarlo como cualquier usuario del sistema y desde cualquier host. Lo peligroso de este binario es que nos permite escapar a una Shell del usuario que lo ejecute, con lo cual tenemos nuestra escalada.

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/SudoRoot.png)

Escape de vim a una bash

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Pasted image 20250320160410.png)

Con esto hemos elevado nuestros privilegios

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Root.png)

Y hemos terminado la maquinita...