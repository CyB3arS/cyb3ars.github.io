---
title: borazuwarahctf
description:
author: cyb3ar
date: 2025-03-26
pin: true
math: true
mermaid: true
image:
  path: /assets/images/dklabs/ve/borazuwarahctf/Banner.png
  alt: 

machine: borazuwarahctf
platform: dklabs
difficult: ve #ve VeryEase, e Easy, m Medium, h hard, vh veryhard, i insane
categories: [DockerLabs, VeryEasy]
temas: [enumeration,SSH Brute Force,Metadata]
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

Al ejecutar el escaneo podemos ver que la maquina tiene 2 puertos abiertos el 22 y 80 TCP

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Nmap1.png)

Al visitar la web disponible en el puerto 80 lo único que encontramos "tails" podría ser un usuario y una pista 

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Web.png)

Procedemos a realizar fuzzing en búsqueda de directorios ocultos o archivos

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Feroxbuster.png)

Y nos encontramos una imagen

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Imagen.png)

Esta sospechoso por lo que revisamos los metadatos por aquello y efectivamente había algo interesante un usuario

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Exiftool.png)

Ahora teniendo este usuario probamos hacer un ataque por fuerza bruta usando hydra y con suerte encontramos la contraseña

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/hydra.png)

Ahora nos conectamos con ssh usando borazuwarah:123456

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/SSH.png)

## Elevación de Privilegios (Escalada)

Al revisar el /etc/passwd vemos el equipo tiene 2 usuarios tienen accesos a una bash

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Usuarios.png)

Revisando la cuenta del usuario vemos que pertenece al grupo sudo

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Sudo.png)

Ademas que a nivel de permisos sudoers tiene permiso de ejecutar todo como cualquier usuario

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Sudoers.png)

Por lo tanto elevamos privilegios simplemente con un sudo su y la contraseña del usuario

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Root.png)

Y con esto hemos terminado la maquinita...