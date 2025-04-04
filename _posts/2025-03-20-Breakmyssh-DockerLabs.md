---
title: BreakmySSH
description:
author: cyb3ar
date: 2025-03-20
pin: true
math: true
mermaid: true
image:
  path: /assets/images/dklabs/ve/breakmyssh/Banner.png
  alt: 

machine: breakmyssh
platform: dklabs
difficult: ve #ve VeryEase, e Easy, m Medium, h hard, vh veryhard, i insane
categories: [DockerLabs, VeryEasy]
tags: [enumeration,SSH Brute Force,CVE,Fuzzing SSH Users]
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

Podemos ver que la maquina tiene 1 puerto abierto el 22 TCP

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Nmap1.png)

### Identificación de versiones y posibles vulnerabilidades

```bash
nmap -p 22,80 -sVC 172.17.0.2
```

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Nmap2.png)

Tenemos el servicio SSH corriendo en el puerto 22 con la versión OpenSSH 7.7, validamos si existen exploit para esa versión de SSH con searchsploit.

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Searchsploit.png)

Sin embargo, en lo personal utilice otro recurso para lograr enumerar usuarios.

[GitHub - CVE-2018-15473](https://github.com/Sait-Nuri/CVE-2018-15473)

## Fuzzing

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Fuzzing.png)

Encontramos el usuario lovely.

## Explotación 

Ahora como conocemos un usuario (lovely y por general existe un usuario root) vamos a probar con fuerza bruta conseguir la contraseña de los usuarios para conectarnos al servicio SSH. Para ello usaremos hydra con el siguiente comando:

```bash
hydra -l lovely -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2
hydra -l root -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2
```

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/hydra1.png)

Hemos encontrado ambas contraseñas lovely:rockyou y root:estrella con la cual nos podemos conectar por ssh al usuario de root.

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Lovely.png)

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Root.png)

Y con esto hemos terminado la maquinita.

## Otra Posible Escalada

Si solo hubiéramos encontrado lovely y tuviéramos que realizar la Escalada en el directorio opt nos damos cuenta que existe un archivo .hash

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/hash.png)

en el cual nos encontramos el siguiente hash

```text
aa87ddc5b4c24406d26ddad771ef44b0
```

Si lo traemos a nuestra maquina atacante y podemos usar Hashcat para identificar el tipo y hash y crackearlo
le pasamos el hash a hashid y nos indica que pude ser cualquier de estos tipos, pero intuyendo probamos con md5

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Hashid.png)

```bash 
hashcat -m 0 hash /usr/share/wordlists/rockyou.txt
```

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Hashcat.png)

Y tenemos la contraseña.

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Cracked.png)

Y con esto hemos terminado la maquinita...