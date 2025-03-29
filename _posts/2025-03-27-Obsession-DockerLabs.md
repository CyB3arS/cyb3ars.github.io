---
title: Obsession
description:
author: cyb3ar
date: 2025-03-27
pin: true
math: true
mermaid: true
image:
  path: /assets/images/dklabs/ve/Obsession/Banner.png
  alt: 

machine: Obsession
platform: dklabs
difficult: ve #ve VeryEase, e Easy, m Medium, h hard, vh veryhard, i insane
categories: [DockerLabs, VeryEasy]
temas: [enumeration,SSH Brute Force,CVE,Fuzzing SSH Users]
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

Encontramos 3 puertos abiertos 80, 21 y 22

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Nmap1.png)

### Identificación de versiones y posibles vulnerabilidades

```bash
nmap -p 22,80 -sVC 172.17.0.2
```

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Nmap2.png)

Encontramos que hay un ftp que tiene acceso anonymous y verificamos con whatweb que otra información podemos encontrar

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Whatweb.png)

Visitamos la web a ver que mas tenemos

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Web.png)

## Fuzzing Web

Al revisar no encontramos mucho así que hacemos una búsqueda de directorios y archivos por aquello usando Feroxbuster

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Feroxbuster.png)

Encontramos un directorio llamado backups con un archivo backup.txt

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Backup.png)

Intentamos abrir el archivo y vemos que tenemos un usuario russoski

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/File.png)

## Explotación 

Ahora que hemos conseguido un usuario podemos usar hydra para tratar de averiguar la contraseña

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Hydra.png)

Y efectivamente la conseguimos "iloveme", seguidamente nos intentamos conectar por SSH

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/SSH.png)

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/User.png)
## Elevación de Privilegios (Escalada)

Comenzamos buscando todos aquellos binarios con permisos SUID de los cuales podemos aprovecharnos para elevar nuestros privilegios

```bash
find / -perm -4000 -user root 2>/dev/null
```

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Perms.png)

Resulta que el binario env tiene permisos SUID y con este binario tenemos nuestra escalada

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Escalada.png)

## Otra opción de escalada

A nivel de sudoers el usuario russoski tiene permisos sobre el binario vim para ejecutarlo como root sin contraseña

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Sudoers.png)

Y el binario tiene la posibilidad de escape a una shell así que si ejecutamos vim como sudo (root)

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/vim.png)

Y usando la instrucción

```shell
:!/bin/bash
```

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Escape.png)

Nos retorna una Shell con el usuario root

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Root.png)

ahora podemos ir al directorio de root y ver que tenia entre sus archivos.. 

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Files.png)

Y con esto hemos terminado la maquinita...
































