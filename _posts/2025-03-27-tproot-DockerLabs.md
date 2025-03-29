---
title: tproot
description:
author: cyb3ar
date: 2025-03-27
pin: true
math: true
mermaid: true
image:
  path: /assets/images/dklabs/ve/tproot/Banner.png
  alt: 

machine: tproot
platform: dklabs
difficult: ve #ve VeryEase, e Easy, m Medium, h hard, vh veryhard, i insane
categories: [DockerLabs, VeryEasy]
tags: [enumeration,CVE]
---

## Pasos Iniciales

- Descargar la maquina {{page.title}} de [DockerLabs](https://dockerlabs.es/)
- Teniendo la maquina en nuestro equipo debemos correr el siguiente comando para descomprimirla:

```bash
unzip {{page.title}}.zip
```

- Procedemos a iniciar maquina {{page.title}} con el siguiente comando:

```bash
#Con esto damos permisos de ejecución al archivo encargado de desplegarnos la máquina.
sudo chmod +x auto_deploy.sh
#Levantamos la maquina objetivo
sudo bash auto_deploy.sh {{page.title}}.tar
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

Con este comando podemos ver que la maquina esta activa y tenemos alcance hacia ella. 

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

---------------------------------------------------------------------------------
### Analizando Resultados del Escaneo

Encontramos 2 puertos abiertos 80 y 21

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Nmap1.png)
### Identificación de versiones y posibles vulnerabilidades

Ejecutamos un escaneo en búsqueda de posibles vulnerabilidades

```bash
nmap -p 22,80 -sVC 172.17.0.2
```

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Nmap2.png)

Vemos que tiene una versión de vsftpd 2.3.4 la cual buscamos con searchsploit

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/SearchSploit.png)

Vemos que hay una posible vulnerabilidad nos bajamos un exploit de estos para probar si tenemos suerte

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/GetSploit.png)
## Explotación 

Procedemos a ejecutar el exploit

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/RunSploit.png)

Al ejecutar el exploit se produce un error como que el puerto esta cerrado pero al ejecutarlo por segunda vez ya funciona.

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Shell.png)

Nos cambiamos a una bash

```bash

script /dev/null -c bash

```

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Root.png)

Y con esto hemos terminado la maquinita...















