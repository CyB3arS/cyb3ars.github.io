---
title: Injection
description:
author: cyb3ar
date: 2025-03-19
pin: true
math: true
mermaid: true
image:
  path: /assets/images/dklabs/ve/Injection/Banner.png
  alt: 

machine: Injection
platform: dklabs
difficult: ve #ve VeryEase, e Easy, m Medium, h hard, vh veryhard, i insane
categories: [DockerLabs, VeryEasy]
tags: [sql-injection,enumeration,permissions]
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

Con este comando podemos ver que la maquina esta activa y tenemos alcance hacia ella

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

Al ejecutar el escaneo de puertos podemos ver que la maquina tiene 2 puertos abiertos el 22 y el 80

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Nmap1.png)

### Identificación de versiones y posibles vulnerabilidades:

```bash
nmap -p 22,80 -sVC 172.17.0.2
```

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Nmap2.png)

Con este comando podemos ver que el servicio SSH corre en el puerto 22 con la versión OpenSSH 8.9p1 además confirmamos que el equipo es un Ubuntu Linux y que corre en el puerto 80 un servicio HTTP Apache 2.4.52. Por lo cual podemos usar la herramienta whatweb para extraer un poco de información sobre la misma:

```bash
whatweb 172.17.0.2
```

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Whatweb.png)

Podemos hacer una enumeración en búsqueda de posibles directorios o archivos usando la herramienta feroxbuster:

```bash
feroxbuster --url http://172.17.0.2 -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -t 200 -d 0 -x php,html,txt
```

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Feroxbuster.png)

Vemos que nos encuentra el archivo index.php por ello vamos en el navegador a ver que es

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/index.png)

## Explotación 

Al ingresar al sitio nos encontramos con esta página la cual nos hace pensar que debe existir alguna base de datos corriendo por abajo contra la cual se validan los datos de inicio de sesión por lo cual deberíamos verificar la posibilidad de que no esté bien programada permitiendo que se acontezca un SQL Injection.

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/SQLInjection.png)

Al probar con la SQL Injection efectivamente no está bien sanitizada y logramos obtener acceso:

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Compromiso.png)

Al verificar la información que nos ha devuelto esta página vemos un posible usuario Dylan y una contraseña: KJSDFG789FGSDF78

Como teníamos presente desde el escaneo existe un puerto 22 abierto el cual es un servicio SSH por lo cual vamos a probar las credenciales que acabamos de encontrar con suerte y nos podemos conectar

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/ssh.png)

Y efectivamente tenemos acceso al equipo y estamos conectados como Dylan.

## Elevación de Privilegios (Escalada)

Comenzamos buscando todos aquellos binarios con permisos SUID de los cuales podemos aprovecharnos para elevar nuestros privilegios

```bash
find / -perm -4000 -user root 2>/dev/null
```

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/SUID.png)

Al verificar nos encontramos que el binario /usr/bin/env tiene permisos SUID y al verificarlo con [GTOBins](https://gtfobins.github.io/gtfobins/env/) vemos que es explotable:

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/atributos.png)

Por lo cual procedemos a realizar la explotación y elevación de privilegios:

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/exploit.png)

Y con esto hemos terminado la maquinita...