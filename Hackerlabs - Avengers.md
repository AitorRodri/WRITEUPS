
# ==RECONOCIMIENTO - EXPLOTACION==


Realizamos un escaneo de nmap en la maquina victima:

`sudo nmap -sS -sCV -p- -v -n -Pn 10.10.10.142 -oN scan.txt`

![[Pasted image 20240924114948.png]]

La maquina victima tiene 4 puertos abiertos: ftp, ssh, http y mysql. El puerto ftp permite el login como el usuario anonimous, vamos a ver lo que hay en su interior:

![[Pasted image 20240924120049.png]]

Vamos a descargarnos los archivos y vemos su contenido

`mget FLAG.txt credential_mysql.txt.zip`
`cat FLAG.txt`

![[Pasted image 20240924120221.png]]
Hemos encontrado una flag, vamos a ver el contenido de "credential_mysql.txt.zip"

`unzip credential_mysql.txt.zip`

![[Pasted image 20240924120405.png]]
Nos pide una contraseña por lo que utilizaremos la herramienta zip2john para extraer el hash del archivo zip a un archivo "hash.txt" y con john haremos un ataque de fuerza bruta para descifrar el hash:

![[Pasted image 20240924120923.png]]

Como no encontramos la contraseña vamos a seguir investigando el puerto 80:

![[Pasted image 20240924121006.png]]

Si miramos el codigo fuente vemos lo siguiente:

![[Pasted image 20240924121148.png]]

Vamos al directorio /code:
![[Pasted image 20240924121226.png]]

Vemos que hay un archivo "code.html" vamos a ver lo que hay en su interior:

![[Pasted image 20240924121325.png]]

Parece estar vacio pero si miramos en el codigo fuente:

![[Pasted image 20240924121408.png]]
Vemos un archivo que pone secreto y la palabra "pikachu"

Vamos a hacer un fuzzing de directorios para ver que mas podemos encontrar:

`gobuster dir -u http://10.10.10.142 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x html,php,xml,txt,jpg,png,pdf,md`

![[Pasted image 20240924121811.png]]

En el directorio /php encontramos lo siguiente

![[Pasted image 20240924121929.png]]
Un usuario h4lc3 y una contraseña con asteriscos

En el directorio /flags encontramos lo siguiente:
![[Pasted image 20240924122202.png]]

En /mysql encontramos dos archivos:

![[Pasted image 20240924122259.png]]

Otra flag:
![[Pasted image 20240924122317.png]]

En database.html podemos ver lo siguiente:

![[Pasted image 20240924122435.png]]

Y en su codigo fuente...

![[Pasted image 20240924122459.png]]
Una contraseña encriptada


En /webs podemos encontrar lo siguiente:

![[Pasted image 20240924122707.png]]

El archivo developers.html contiene un panel de login:

![[Pasted image 20240924122753.png]]

y secret.html contiene un formulario donde se busca una palabra:

![[Pasted image 20240924122857.png]]

Vamos a desencriptar la contraseña que hemos conseguido en "database.html":

![[Pasted image 20240924123304.png]]

Nos sale la palabra fuerza bruta. Esta palabra la podemos utilizar para desencriptar el zip, login con ssh utilizando el usuario, login en el panel en la web o para buscar esta palabra en el formulario:

![[Pasted image 20240924123447.png]]

He encontrado a otro usuario: hulk

Si desciframos el contenido del archivo "code.html" que contenia el cifrado "pikachu" podemos ver el siguiente mensaje:

![[Pasted image 20240924124219.png]]

Vamos a probar si podemos acceder por ssh con el usuario "hulk" y contraseña "fuerzabruta"

![[Pasted image 20240924125546.png]]


# ==ESCALADA==

Lo primero que suelo hacer es comprobar los permisos del usuario para ejecutar comandos como sudo:

![[Pasted image 20240924125751.png]]
No tenemos ningun permiso

Vamos a ver los usuarios que hay en /home

![[Pasted image 20240924125841.png]]
Tenemos 4 usuarios

En el directorio home de root tenemos los siguientes archivos:

![[Pasted image 20240924125937.png]]

En el directorio "db" tenemos lo siguiente:

![[Pasted image 20240924130032.png]]

Ejecutamos el comando "tree" para verlo mejor

`tree -a`

![[Pasted image 20240924130157.png]]

Leemos la flag:

![[Pasted image 20240924130217.png]]

Hacemos lo mismo en el directorio /mysql/hint

![[Pasted image 20240924130352.png]]

Leemos el archivo:

![[Pasted image 20240924130433.png]]
Dice que si miramos bien el archivo podemos saber la contraseña para descifrarlo


Vamos a ver el contenod de .passwd:

![[Pasted image 20240924130552.png]]
Dice que con algun usuario puedo ejecutar ese script para escalar privilegios


Vamos a ver el contenido del directorio wait:

![[Pasted image 20240924130820.png]]
Nos da una contraseña que permitira desencriptar un archivo: decryptavengers

Podemos probar a desencriptar el zip con la palabra "shit_how_they_did_know_this_password" ya que decia que este archivo contenia la password:

![[Pasted image 20240924131837.png]]

Las credenciales para mysql son:
- Username=hulk
- password=fuerzabrutaXXXX

Tenemos que crear una wordlist con todos los numeros del 0001 al 3000:

![[Pasted image 20240924132224.png]]
En wordlist.txt es donde se me genera el diccionario

Ahora vamos a realizar un ataque de fuerza bruta a mysql con la wordlist generada:

`hydra -l hulk -P wordlist.txt mysql://10.10.10.142`

![[Pasted image 20240924133334.png]] 

Intentamos conectarnos desde nuestra maquina pero nos da error:
![[Pasted image 20240924133538.png]]

Por lo que intentaremos contectarnos desde la maquina victima:

![[Pasted image 20240924133608.png]]

En la base de datos db_flag encontramos otra flag:

![[Pasted image 20240924133721.png]]

En la base de datos db_true encontramos unas credenciales:

![[Pasted image 20240924133827.png]]

En la base de datos no_db encontramos credenciales:

![[Pasted image 20240924134248.png]]

Nos cambiamos al usuario stif con las credenciales que hemos conseguido:

![[Pasted image 20240924134436.png]]

Dentro del directorio flag podemos encontrar un archivo "flag.txt.zip" pero no sabemos la contraseña:

![[Pasted image 20240924134633.png]]

Dentro de pista encontramos un archivo que solo antman tiene permisos:
![[Pasted image 20240924134726.png]]

Dentro de .power podemos encontrar la siguiente pista:
![[Pasted image 20240924134849.png]]

Como no nos deja hacer unzip en el usuario actual, vamos a ver los comandos que podemos ejecutar como sudo

![[Pasted image 20240924140305.png]]

Si podemos ejecutar bash como sudo podemos hacer lo siguiente para conseguir una shell como root:

`sudo bash -p`

![[Pasted image 20240924140415.png]]
