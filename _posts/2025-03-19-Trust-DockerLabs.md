---
title: Trust
description:
author: cyb3ar
date: 2025-03-19
pin: true
math: true
mermaid: true
image:
  path: /assets/images/dklabs/ve/Injection/Banner.png
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

Para iniciar con esta maquina debemos verificar conexión con la maquina objetivo. Para ello le lanzamos el comando ping:

```bash
ping -c 1 172.17.0.2
```

Con este comando podemos ver que la maquina esta activa y tenemos alcance hacia ella, además por el TTL que tiene el cual esta cercano a los 64 podemos pensar que es un equipo Linux, ya que Windows por lo general tiene un TTL cercano a los 128. 

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Pasted image 20250319203547.png)

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

Al ejecutarlo podemos ver que la maquina tiene 2 puertos abiertos el 22 y el 80

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Pasted image 20250320152205.png)

Analizamos los servicios para identificar versiones y posibles vulnerabilidades:

```bash

nmap -p 22,80 -sVC 172.18.0.2

```

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Pasted image 20250320152305.png)

Con este comando podemos ver que el servicio SSH corre en el puerto 22 con la versión OpenSSH 9.2p1 además confirmamos que el equipo es un  Debian y que corre en el puerto 80 un servicio HTTP Apache 2.4.57. Por lo cual podemos usar la herramienta whatweb para extraer un poco de información sobre la misma:

```bash

whatweb 172.18.0.2

```

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Pasted image 20250320153259.png)

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Pasted image 20250320153349.png)

Podemos hacer una enumeración en búsqueda de posibles directorios o archivos usando la herramienta feroxbuster:

```bash

feroxbuster --url http://172.17.0.2 -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -t 200 -d 0 -x php,html,txt

```

Vemos que nos encuentra el archivo index.html y secret.php por ello vamos en el navegador a ver que es:

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Pasted image 20250320154117.png)

La web a la que nos lleva es la siguiente:

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Pasted image 20250320154216.png))

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Pasted image 20250320155005.png)

Lo único que vemos acá que nos puede interesar es Mario un posible usuario, a nivel del código fuente no hay nada.
# Explotación 

Como conocemos un posible usuario vamos a probar con fuerza bruta conseguir la contraseña del usuario para conectarnos al servicio SSH. Para ello usaremos hydra con el siguiente comando:

```bash

hydra -l mario -P /usr/share/wordlists/rockyou.txt ssh://172.18.0.2

```

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Pasted image 20250320155355.png)

Hemos encontrado la contraseña "chocolate" con la cual nos podemos conectar por ssh al usuario de mario.

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Pasted image 20250320155704.png)

Y efectivamente tenemos acceso al equipo y estamos conectados como Mario.
# Elevación de Privilegios (Escalada)

```bash

find / -perm -4000 -user root 2>/dev/null

```

Al ejecutar este comando buscamos todos aquellos binarios con permisos SUID con los cuales podemos aprovecharnos de ellos para elevar nuestros privilegios 

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Pasted image 20250320155754.png)

Sin embargo no tenemos nada que podamos aprovechar, seguimos buscando

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Pasted image 20250320155900.png)

Y nos encontramos lo siguiente el usuario Mario tiene permisos de ejecución del binario  /usr/bin/vim para ejecutarlo como cualquier usuario del sistema y desde cualquier host. Lo peligroso de este binario es que nos permite escapar a una Shell del usuario que lo ejecute, con lo cual tenemos nuestra escalada.

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Pasted image 20250320160342.png)

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Pasted image 20250320160410.png)

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Pasted image 20250320160432.png)

Y con esto hemos terminado la maquinita..