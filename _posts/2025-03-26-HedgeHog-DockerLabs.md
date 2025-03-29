---
title: HedgeHog
description:
author: cyb3ar
date: 2025-03-26
pin: true
math: true
mermaid: true
image:
  path: /assets/images/dklabs/ve/HedgeHog/Banner.png
  alt: 

machine: HedgeHog
platform: dklabs
difficult: ve #ve VeryEase, e Easy, m Medium, h hard, vh veryhard, i insane
categories: [DockerLabs, VeryEasy]
temas: [enumeration,SSH Brute Force]
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

Al ejecutarlo podemos ver que la maquina tiene 2 puertos abiertos el 22 y 80 TCP

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Nmap1.png)

Al visitar la web disponible en el puerto 80 lo único que encontramos es tails el cual podría ser un usuario y una pista por donde continuar. 

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Web.png)

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Source.png)

Sin embargo, no hay más por lo cual tratamos de buscar directorios ocultos usando feroxbuster pero tampoco hay mucho

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Feroxbuster.png)

Probamos con Gobuster por aquello

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Gobuster.png)

## Explotación 

Como nos dan una pista "tails" el cual puede ser un usuario y tenemos otro puerto y es SSH podemos forzar para intentar encontrar la contraseña

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/hydra1.png)

Sin embargo, después de esperar un buen tiempo no vemos nada por lo que si tomamos a tails como una pista podemos pensar que la contraseña pudiera estar entre las últimas del diccionario. Por lo cual vamos a hacer lo siguiente:

Nos copiamos el rockyou a una ubicación distinta para no alterar el original, pero usando el comando tac para invertir el contenido.

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Tac.png)

y limpiamos todos los posibles espacios en blanco

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Sed.png)

Y después de un ratito. tenemos una contraseña.

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/hydra2.png)

Y estamos dentro

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Compromiso.png)

## Elevación de Privilegios (Escalada)

Comenzamos buscando todos aquellos binarios con permisos SUID de los cuales podemos aprovecharnos para elevar nuestros privilegios

```bash
find / -perm -4000 -user root 2>/dev/null
```
Archivos con permisos SUID tampoco tenemos archivos de importancia

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/SUID.png)

Revisamos un poco el usuario y no tenemos nada

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Id.png)

Ojito aquí, esto sí que es de importancia, tenemos permisos de ejecución de comandos como el usuario sonic

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Sudo.png)

y si verificamos este equipo tiene 3 usuarios con un acceso a una Shell, tails, sonic y root

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Passwd.png)

como tenemos permisos de ejecutar binarios como sonic usamos para que nos brinde una bash de ese usuario

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Move.png)

al lograr entablar una bash con el usuario sonic vemos que en directorio documentos hay una contraseña

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Sonic.png)

hemos probado la contraseña con root y con sonic. Y la contraseña es del usuario sonic, sin embargo, al revisar vemos que el usuario sonic puede ejecutar comandos como cualquier usuario y sin contraseña

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Sudo2.png)

y simplemente escalamos privilegios.

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Root.png)

Y listo

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Root2.png)

Y con esto hemos terminado la maquinita...