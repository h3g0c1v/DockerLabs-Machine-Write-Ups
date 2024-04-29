---
Nombre de la máquina: Trust
Sistema Operativo: Linux
Dificultad: Very Easy 🔵
Skills: Web Enumeration, Hydra Brute Force, Abusing sudoers
Enlace de descarga: https://mega.nz/file/UacxFKDR#G5KBHBt8ASB0lHPuttnaxKROAa40FMGrvBoIBf6ak0E
---
---

## Montando el laboratorio

Lo primero que deberemos hacer para practicar con esta máquina es, dirigirnos a la página de [dockerlabs](https://dockerlabs.es/) y descargarnos la máquina desde el [siguiente enlace](https://mega.nz/file/UacxFKDR#G5KBHBt8ASB0lHPuttnaxKROAa40FMGrvBoIBf6ak0E). Una vez descargada y descomprimida ejecutaremos el siguiente comando.

```bash
sudo bash auto_deploy.sh trust.tar
```

Al ejecutar el comando, nos pedirá la contraseña. Al introducirla correctamente, tendremos que esperar hasta que nos indique que la máquina está desplegada.

![Desplegando máquina Trust](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/5642f3fd-086b-4791-b5a0-f55cebc58b81)

Como podemos ver en la imagen anterior, la máquina está desplegada correctamente y su dirección IP es la **172.17.0.2**. Esto nos habrá montado la máquina vulnerable *Trust* a través de *docker*. 

## Como eliminar la máquina vulnerable

En caso de querer eliminarla, simplemente presionaremos en el teclado **CTRL** + **C** y la máquina se eliminará automáticamente.

![Eliminando máquina Trust](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/e80c6f86-622a-40c8-8d32-c98dd6845945)

## Comprobando la conectividad

Una vez desplegada, deberemos cerciorarnos de que tenemos conectividad con esta. Para ello, haremos uso del comando `ping` para realizar el chequeo.

![ping máquina Trust](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/a4935e5a-497d-43d7-9cf5-15a7ac6e7269)

Comprobamos que tenemos conectividad y, podemos intuir mediante el **TTL** que, el sistema operativo de la máquina víctima es *Linux*.

## Enumeración

Pasaremos con la enumeración de la máquina. Ejecutaremos el siguiente comando con el fin de descubrir posibles puertos abiertos de la víctima.

```bash
sudo nmap -p- --open --min-rate 5000 -vvv -n -Pn 172.17.0.2
```

Al ejecutar el comando anterior, el resultado es el siguiente.

![Primer nmap de la máquina Trust](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/eb6b0941-4673-457b-b977-1248159641d0)

Observamos que, los puertos que están abiertos son el puerto *22* y *80*. Ahora que sabemos esto, detectaremos la versión y servicio que corren bajo dichos puertos y lanzaremos una serie de scripts de enumeración propios de la herramienta `nmap` para que nos muestre información adicional.

```bash
sudo nmap -sCV -p80,22 172.17.0.2
```

Ejecutamos el comando.

![Segundo nmap de la máquina Trust](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/cefb37c9-8bf9-4783-bfd7-1e1d9c900118)

En el resultado anterior detectamos que, bajo el puerto 22 está corriendo el protocolo **SSH** y en el puerto 80 **HTTP** siendo *Apache* el servicio utilizado. Antes de realizar ningún *fuzzing* de subdirectorios, con herramientas como `fuzz` o `gobuster` ejecutaremos el script *http-enum.nse* de `nmap` para ver si nos devuelve algo interesante.

```bash
sudo nmap --script=http-enum 172.17.0.2
```

Ejecutamos el comando.

![http-enum de la máquina Trust](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/f8c2a98e-7586-4f9f-bb44-c826fc971914)

Al no devolvernos nada interesante, nos dirigiremos a la página web para visualizar su contenido.

![puerto 80 de la máquina Trust](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/196f06d6-97e1-4f5d-8e2c-582e1752ac5e)

Por lo que vemos, es la página por defecto de *Apache2* así que aquí no hay nada interesante que ver. Podemos probar a ver que hay en los comentarios del código HTML de la página, sin embargo, no encontraremos nada inusual.

Ahora si, realizaremos *fuzzing* de subdirectorios con la herramienta `gobuster`. A demás de busca, únicamente, por subdirectorios probaremos a buscar por alguna extensión interesante.

```bash
gobuster dir -u "http://172.17.0.2/" -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -x php,txt,py,bak,php.bak
```

Al realizar la enumeración, el resultado es el de a continuación.

![Gobuster de la máquina Trust](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/9c92a257-f143-4a62-9b42-2b4d7094cd72)

Nos llama bastante la atención ese tal fichero `secret.php`, así que vamos a averiguar que contiene.

![secret-php de la máquina Trust](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/c6767e01-1793-4df3-adf3-acf9eb5fb57f)

Encontramos que, el usuario *Mario* es un usuario válido por lo que se abre la vía de realizar un **ataque de fuerza bruta a SSH**, con la herramienta `hydra` para ver si averiguamos la contraseña de ese tal *Mario*. 

## Fuerza bruta con Hydra a SSH

Para realizar un ataque de fuerza bruta a *SSH* con la herramienta `hydra`, tendremos que ejecutar el siguiente comando.

```bash
hydra -l mario -P /usr/share/wordlist/rockyou.txt ssh -t 4
```

Ejecutaremos el comando y, al cabo de un rato, encontraremos una contraseña válida para este usuario.

![fuerza bruta hydra puerto 22 SSH de la máquina Trust](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/c6e429c7-ae95-4296-afd8-e90cfd9e73d0)

## Acceso al sistema

Ahora que tenemos credenciales válidas a nivel del sistema nos conectaremos a la máquina.

```bash
ssh mario@172.17.0.2
```

Nos conectamos aceptando el *fingerprint* y estaremos dentro de la máquina víctima.

![acceso a la máquina Trust por SSH](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/968b4430-1b90-4cf8-89d2-e8fb133d02c7)

Teniendo la contraseña de Mario, consultaremos lo que podemos hacer a nivel de permisos *sudoers*.

```bash
sudo -l
```

Vemos que podemos ejecutar el comando `/usr/bin/vim` como root sin proporcionar su contraseña.

![Permisos de sudoers de la máquina Trust](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/1332b077-2c5e-416d-8241-f10bec92fd57)

Esto es bastante peligroso, ya que es una vía potencial de escalada de privilegios.

## Escalada de privilegios (obteniendo acceso como root)

A través de este binario, podemos escalar privilegios como el usuario que lo ejecutemos. Podemos dirigirnos a la página de [GTFOBins](https://gtfobins.github.io/) y buscar por este binario.

![abusing vim SUID to LPE máquina Trust](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/cf3a3859-1aac-42ad-b5f8-7e84bd71fd98)

Nos dice que, ejecutando el comando anterior, podremos abusar de este binario para escalar privilegios.

![acceso como root en la máquina Trust](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/5880b9bb-7064-4383-80ec-86004959f88c)

En este caso, es como el usuario root que podemos ejecutar este comando, por lo que nos hemos podido convertir en este.
