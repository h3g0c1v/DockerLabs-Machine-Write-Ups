---
Nombre de la máquina: Vacaciones
Sistema Operativo: Linux
Dificultad: Very Easy 🔵
Skills: Web Enumeration, Hydra Brute Force, Abusing Sudoers
Enlace de descarga: https://mega.nz/file/YCEGAISD#y6iWUG_auH4vUApClb9ix7H6JmOCKm4vAYS2TjQn59g
---
---

## Montando el laboratorio

Lo primero que deberemos hacer para practicar con esta máquina es, dirigirnos a la página de [dockerlabs](https://dockerlabs.es/) y descargarnos la máquina desde el [siguiente enlace](https://mega.nz/file/YCEGAISD#y6iWUG_auH4vUApClb9ix7H6JmOCKm4vAYS2TjQn59g).

```
sudo bash auto_deploy.sh vacaciones.tar
```

Al ejecutar el comando, nos pedirá la contraseña. Al introducirla correctamente, tendremos que esperar hasta que nos indique que la máquina está desplegada.

![desplegando la maquina vacaciones](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/ca543115-963a-4ad3-9d65-2fad540356e8)

Como podemos ver en la imagen anterior, la máquina está desplegada correctamente y su dirección IP es la **172.17.0.2**.

## Como eliminar la máquina vulnerable

Esto nos habrá montado la máquina vulnerable *Vacaciones* a través de *docker*. En caso de querer eliminarla, simplemente presionaremos en el teclado **CTRL** + **C** y la máquina se eliminará automáticamente.

![eliminando la maquina vacaciones](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/4f5f9f9e-789b-4b3c-bb88-5b6fa26a9c1f)

## Comprobando la conectividad

Una vez desplegada, deberemos cerciorarnos de que tenemos conectividad con esta. Para ello, haremos uso del comando `ping` para realizar el chequeo.

![ping máquina Vacaciones](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/a4380fca-46ef-43ee-90e3-85298f385375)

Comprobamos que tenemos conectividad y, podemos intuir mediante el **TTL** que, el sistema operativo de la máquina víctima es *Linux*.

## Enumeración

Pasaremos con la enumeración de la máquina. Ejecutaremos el siguiente comando con el fin de descubrir posibles puertos abiertos de la víctima.

```
sudo nmap -p- --open --min-rate 5000 -vvv -n -Pn 172.17.0.2
```

Al ejecutar el comando anterior, el resultado es el siguiente.

![primer nmap maquina vacaciones](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/d12f5f61-6ced-4384-9ca9-834e3ba5f1ee)

Observamos que, los puertos que están abiertos son el puerto *22* y *80*. Ahora que sabemos esto, detectaremos la versión y servicio que corren bajo dichos puertos y lanzaremos una serie de scripts de enumeración propios de la herramienta `nmap` para que nos muestre información adicional.

```
sudo nmap -sCV -p80,22 172.17.0.2
```

Ejecutamos el comando.

![segundo nmap maquina vacaciones](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/8189bb7c-f61c-4fa5-9701-8a72968834da)

En el resultado anterior detectamos que bajo el puerto 22 está corriendo el protocolo **SSH** y en el puerto 80 **HTTP** siendo *Apache* el servicio utilizado para este último puerto. Vamos a pasar a ver el contenido de la página web.

![puerto 80 pagina web maquina vacaciones](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/38305f47-1de7-4e9a-aac1-a5cdba07b036)

¡Vaya! No vemos nada en la web, vamos a visualizar el código fuente para ver si hay algún tipo de comentario.

![codigo fuente pagina web puerto 80 maquina vacaciones](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/9057be7c-ba11-47b4-b887-235afe452181)

Como vemos en la imagen anterior, parece que los usuarios *juan* y *camilo* son usuarios válidos del sistema, por lo que nos lo apuntamos y vamos a probar a efectuar un ataque de fuerza bruta por el servicio de **SSH** para estos usuarios.

## Fuerza bruta con Hydra a SSH

Lo primero que haremos es guardar estos usuarios en un fichero. En mi caso se llamará `users`.

![fichero de usuarios descubiertos maquina vacaciones](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/a8540855-8501-4aeb-bd51-b571347fdcae)

Ahora, efectuaremos el ataque por fuerza bruta con el siguiente comando.

```
hydra -L users -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2 -t 4
```

Al cabo de un rato, nos descubrirá la contraseña de *camilo*.

![contraseña usuario camilo hydra por ssh](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/57c8f394-285f-4878-9a99-f637f6f5276d)

## Acceso al sistema (SSH)

Como sabemos las credenciales por **SSH** para el usuario *camilo*, nos conectaremos a la máquina.

![conectandonos por ssh con el usuario camilo](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/5e604df1-7602-45e7-835b-375243d2a1d3)

## Convirtiéndonos en el usuario juan

Si recordamos, anteriormente vimos el siguiente comentario en la página web.

![codigo fuente pagina web puerto 80 maquina vacaciones](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/ab3e1ba9-09ca-4d9d-b755-18c7b5dcbbc6)

Nos comenta *juan*, que nos ha dejado un correo importante en nuestro buzón. Normalmente, hay dos lugares en los que pueden estar los correos que nos llegan en *Linux*. Uno de ellos será en nuestro directorio **HOME** y el otro es en el directorio `/var/mail`. En este caso, el correo se encuentra en la ruta `/var/mail/camilo/correo.txt`.

![correo importante de juan](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/ee930eb1-4e4e-4be7-9d92-0de1f8b606c7)

Parece que *juan* se ha ido de vacaciones y nos ha dejado su contraseña así que vamos a probar si realmente es válida.
![convirtiendonos en juan](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/18027f30-b122-44dc-ba52-c1f6e6bdf61e)

## Escalada de privilegios (acceso como root)

La contraseña que nos ha dado es correcta y hemos conseguido acceder como el usuario *juan*. Ahora, con este usuario, visualizaremos lo que podemos hacer a nivel de *sudoers*.

![permisos sudoers usuario juan](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/e0f26831-5437-42e5-9471-45e112b3eb22)

Si nos vamos a [GTFOBins](https://gtfobins.github.io/) y buscamos por este binario, observaremos que presenta un riesgo bastante grave que tengamos permisos de ejecutarlo como *root* sin proporcionar contraseña, ya que nos permite convertirnos en el usuario *root* de una manera muy sencilla.

![ruby gtfobins como sudo](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/107ceb3c-a706-4a95-8410-1569ec739639)

Si ejecutamos el comando que nos indica, conseguiremos acceso como el usuario *root* y habremos rooteado la máquina *Vacaciones*.

![acceso como root con el binario ruby como sudo](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/c0a11a29-670c-4ac8-8901-6cabe7eb5989)
