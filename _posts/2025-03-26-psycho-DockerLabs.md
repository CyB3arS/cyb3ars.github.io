---
title: Psycho
description:
author: cyb3ar
date: 2025-03-26
pin: true
math: true
mermaid: true
image:
  path: /assets/images/dklabs/ve/psycho/Banner.png
  alt: 

machine: psycho
platform: dklabs
difficult: e #ve VeryEase, e Easy, m Medium, h hard, vh veryhard, i insane
categories: [DockerLabs, Easy]
tags: [Fuzzing Web Parameters,SSH PrivateKey,enumeration,Perl,Python3]
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
-n Desactivamos resolución de DNS
-Pn Desactivamos el HostDiscovery
```

---------------------------------------------------------------------------------

### Analizando Resultados del Escaneo

Al ejecutar el escaneo podemos ver que la maquina tiene 2 puertos abiertos el 22 y el 80 TCP

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Nmap1.png)

### Identificación de versiones y posibles vulnerabilidades

Ejecutamos un escaneo con los Scripts básicos de reconocimiento para identificación de versiones y vulnerabilidades

```bash
nmap -p 22,80 -sVC 172.17.0.2
```

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Nmap2.png)

Utilizando whatweb verificamos la página web del puerto 80 pero no obtenemos mucho

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Whatweb.png)

Al revisar el sitio web nos encontramos con lo siguiente

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Web.png)

Este error en la parte inferior es curioso y puede indicar la existencia de algún parámetro

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Web2.png)

Es por ello que vamos a realizar Fuzzing de parámetros

### Fuzzing

Utilizando ffuf y una wordlist podemos buscar posibles parámetros

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/ffuf.png)

Sin embargo, vemos demasiado ruido y la información que es relevante no la vemos por lo cual debemos filtrar 

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/ffuf_filter.png)

Después de un tiempo descubrimos el parámetro, el cual es "secret" por lo que trataremos de ver como lo podemos utilizar

## Explotación

Después de algunas pruebas logramos dar con algo

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Parameter1.png)

Logramos listar todos los usuarios y vemos que tenemos un usuario luisillo y otro llamado vaxei aparte de root que tienen una shell por lo que tratamos de ver si es posible obtener alguna id_rsa de alguno de estos usuarios

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Parameter2.png)

Después de probar con los usuarios descubiertos encontramos que el usuario vaxei si contaba con una id_rsa por lo que la traemos a nuestra maquina atacante

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/id_rsa.png)

Ahora que tenemos el id_rsa en nuestro equipo atacante debemos darle permisos

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Permisos.png)

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/VerificaPermisos.png)

Después de haberle dado los permisos correctos a la id_rsa nos intentamos conectar utilizando el usuario vaxei por SSH

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/SSH.png)

## Elevación de Privilegios 

Como logramos tener acceso con el usuario vaxei ahora debemos elevar nuestros privilegios para lo cual buscamos binarios SUID pero no encontramos nada utilizable

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/SUID.png)

Ojo al verificar permisos en SUDOERS
vemos que el usuario vaxei tiene permisos para ejecutar un binario perl como luisillo sin necesidad de contraseña y tal como veíamos anteriormente solo 3 usuarios tenían una bash (luisillo, vaxei y root)

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/SUDOERS.png)

Por lo que usando perl tratamos de obtener una bash del usuario luisillo

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/SUDO2.png)

Y logramos obtener una bash con el usuario luisillo

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/luisillo.png)

Posteriormente volvemos a verificar los permisos de sudo

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/SUDO_Luisillo.png)

Analizando la salida de permisos del usuario luisillo vemos que tiene permisos de ejecutar como cualquier usuario el binario /usr/bin/python3 y específicamente el script /opt/paw.py

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/PermisosOPT.png)

Además, al revisar los permisos el directorio opt vemos que tiene configurado permisos de lectura, escritura y ejecución a otros usuarios. Lo cual nos permitiría eliminar el archivo y recrearlo con el contenido que nos plazca

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/File_Privs.png)

Sin embargo, a nivel de usuario no podríamos modificar directamente el archivo porque solo el propietario tiene permisos de lectura, escritura y ejecución, Por ello eliminamos el archivo y lo recreamos.

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Remove_File.png)

Lo editamos para que ejecute lo que nos plazca

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Edit_py.png)

En este caso lo vamos editar para que nos retorne una bash.. y podemos hacerlo de 2 formas

#### Usando la libreria OS

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/os_shell.png)

#### Usando la Liberia Subprocess

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/subprocess_shell.png)

Después de editar el archivo, guardar los cambios y ejecutar nuestro archivo tenemos nuestra bash del usuario root

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Exploit.png)

![](/assets/images/{{page.platform}}/{{page.difficult}}/{{page.machine}}/Root.png)

Y con esto hemos terminado la maquinita...