---
title: Vacaciones
description: 
author: cyb3ar
date: 2025-03-27
pin: true
math: true
mermaid: true
image:
  path: /assets/images/dklabs/ve/Vacaciones/Banner.png
  alt: 
machine: Vacaciones
platform: dklabs
difficult: ve
categories:
  - DockerLabs
  - VeryEasy
tags:
  - enumeration
  - SSH
  - Brute
  - Force
  - CVE
  - Fuzzing
  - SSH
  - Users
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

![[Nmap1.png]]
### Identificación de versiones y posibles vulnerabilidades

```bash
nmap -p 22,80 -sVC 172.17.0.2
```

![[Nmap2.png]]

Como tenemos un servicio Web nos dirigimos a verificar que tenemos, pero no vemos nada, por ello revisamos el código fuente en búsqueda de comentarios o algo que nos sea de utilidad

![[Web.png]]

Al revisar el código fuente vemos que hay 2 posibles usuarios, Juan y Camilo

![[Source_code.png]]

## Fuzzing de Directorios y Archivos

Antes de continuar buscamos directorios ocultos u archivos pero no hay nada

![[Feroxbuster.png]]
## Explotación 

Por lo que le pasamos la información de los usuarios a hydra y procedemos a hacer un ataque de fuerza bruta al ssh utilizando los usuarios camilo y juan encontrando las credenciales de camilo y nos conectamos con SSH

![[Pasted image 20250326115903.png]]

Y nos conectamos

![[Camilo.png]]

Ahora que nos logramos conectar verificamos que otros usuarios hay con acceso a una sh o una bash y encontramos 4 usuarios

![[Users.png]]
## Elevación de Privilegios (Escalada)

Comenzamos buscando todos aquellos binarios con permisos SUID de los cuales podemos aprovecharnos para elevar nuestros privilegios

```bash
find / -perm -4000 -user root 2>/dev/null
```

Tambien verificamos si tenemos permisos para ejecutar algún binario como sudo pero no

![[Pasted image 20250326120447.png]]

Como el usuario menciono en la web que le había dejado un correo importante a Camilo no dirigimos al directorio /var/mail/camilo y nos encontramos con el correo que nos dejo Juan

![[Correo.png]]

Y Juan nos dejo una contraseña con lo cual nos podemos migrar al usuario de Juan

![[Juan.png]]

Revisamos los permisos que tenia Juan para ejecutar binarios privilegiados y Oh Sorpresa..

![[Permisos.png]]

Juan estas despedido... tiene permisos para ejecutar ruby como cualquier usuario y en este caso como root que es el que nos interesa entonces procedemos a elevar nuestros privilegios

![[Root.png]]

Y con esto hemos terminado la maquinita...






